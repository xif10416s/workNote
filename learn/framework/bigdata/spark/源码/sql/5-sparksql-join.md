# join 操作
## join 笛卡尔积
### 代码流程
```
spark.conf.set("spark.sql.autoBroadcastJoinThreshold", -1) // 关闭 broadcast hit
//1 加载数据成DataFrame
val df = spark.read.json("examples/src/main/resources/people.json")
//2 调用dataframe的join操作
val rs = df.join(df)
/**
2.1  首先封装成Join类型的新逻辑计划 ：Join(planWithBarrier, right.planWithBarrier, joinType = Inner, None)，
2.1.1 当前df为left,被join的为right，逻辑计划类型为BinaryNode，包含2个节点
2.1.2 planWithBarrier = AnalysisBarrier(logicalPlan)，给已经解析过的逻辑计划设置解析屏障，避免重复解析
2.2 封装成新的rs:DataFrame =》Dataset.ofRows(sparkSession, logicalPlan)，dataframe通常流程，解析，优化，生成物理计划，生成可执行计划
2.2.1  logical 计划：Join 
2.2.2  sparkPlan 计划 ： CartesianProductExec (根据join 类型匹配生成)，left，right为FileSourceScanExec，执行拉取数据逻辑
2.2.3  executedPlan 计划 ： CartesianProductExec，left,right为 wholeStageCodegenExec，已经插入全局生成代码节点
2.2.3.1 wholeStageCodegenExec的子节点为FileSourceScanExec
*/

// 3 执行count,触发job
rs.count()
/**
3.1 最终会执行：CartesianProductExec# doExecute
3.1.1 分别执行left,right的executre
    val leftResults = left.execute().asInstanceOf[RDD[UnsafeRow]]
        => MapPartitionRDD  =>prev = FileScanRDD
    val rightResults = right.execute().asInstanceOf[RDD[UnsafeRow]]
3.1.2 封装成UnsafeCartesianRDD，
    val pair = new UnsafeCartesianRDD(
    leftResults,
    rightResults,
    right.output.size,
    sqlContext.conf.cartesianProductExecBufferInMemoryThreshold,
    sqlContext.conf.cartesianProductExecBufferSpillThreshold)
3.1.2.1 spark.sql.cartesianProductExec.buffer.in.memory.threshold:保存在数组中内存的阈值，超过后刷新到disk,值太大容易引起OOM
默认4096
3.1.2.2 spark.sql.cartesianProductExec.buffer.spill.threshold：
值太小，引起频繁刷新disk
3.1.2.3 ExternalAppendOnlyUnsafeRowArray 
   专门给UnsafeRows的数组，带阈值刷新到disk

CartesianRDD#getPartitions 分区是如何处理的
    // 总分区数是 = 左表的分区数 * 右表的分区数
    val array = new Array[Partition](rdd1.partitions.length * rdd2.partitions.length)
    // 笛卡尔积分区
    for (s1 <- rdd1.partitions; s2 <- rdd2.partitions) {
          val idx = s1.index * numPartitionsInRdd2 + s2.index
          array(idx) = new CartesianPartition(idx, rdd1, rdd2, s1.index, s2.index)
        }
UnsafeCartesianRDD#compute 数据连接处理
    // 准备一个数组，内存中只保存inMemoryBufferThreshold行，超过的会分隔到文件
    val rowArray = new ExternalAppendOnlyUnsafeRowArray(inMemoryBufferThreshold, spillThreshold)
    // 获取当前笛卡尔积分区
    val partition = split.asInstanceOf[CartesianPartition]
    // 先计算rdd2也就是右表，实际计算过程就是加载数据的过程，添加到数组缓存 《== 加载右表分区的数据
    rdd2.iterator(partition.s2, context).foreach(rowArray.add)
    // 转化成迭代器
    def createIter(): Iterator[UnsafeRow] = rowArray.generateIterator()
    // 将当前分区rdd1的每行数据和rdd2分区的每行数据连接，组成新的行，分装成迭代器返回，注册了清理数组缓存的函数
    val resultIter =
      for (x <- rdd1.iterator(partition.s1, context);
           y <- createIter()) yield (x, y)
    CompletionIterator[(UnsafeRow, UnsafeRow), Iterator[(UnsafeRow, UnsafeRow)]](
      resultIter, rowArray.clear())

CartesianProductExec#doExecute 加载数据执行过滤
    初始化RDD,val pair = new UnsafeCartesianRDD(）

*/

```

## SortMergeJoin 
*   匹配条件 
    -   匹配到有表的连接条件
        +   case ExtractEquiJoinKeys(joinType, leftKeys, rightKeys, condition, left, right)
    -   连接键可排序
        +   RowOrdering.isOrderable(leftKeys)，
*   默认使用，两个都是大表并且key可排序的情况
```
spark.conf.set("spark.sql.autoBroadcastJoinThreshold", -1) // 关闭 broadcast hit
//1 加载数据成DataFrame
val df = spark.read.json("examples/src/main/resources/people.json")
val df2 = spark.read.json("examples/src/main/resources/people.json")
val rs = df.join(df2,df.col("age") === df2("age")) 
println(rs.count())
/**
生成对应的executePlan为SortMergeJoinExec
    同样有两个内存相关的配置：都保存在内存中
        spark.sql.sortMergeJoinExec.buffer.spill.threshold ：默认是整数最大值，无上限
        sortMergeJoinExecBufferInMemoryThreshold：默认是整数最大值，无上限
SortMergeJoinScanner#findNextInnerJoinRows，匹配left，和right的key相同的数据，基本逻辑：
    1. 以leftIter为基准，获取一行数据，并通过leftProject获取leftKey
        if (streamedIter.advanceNext()) {
          streamedRow = streamedIter.getRow
          streamedRowKey = streamedKeyGenerator(streamedRow)
          true
        }
    2.  在类似的方式获取rightIter的一行数据，和rightKey
    3.  比较leftKey 和 rightKey大小 ,直到left或right没有数据
        -   comp = 0，调用bufferMatchingRows保存
        -   comp > 0 则执行 2 , 获取下一行right
        -   comp <0 则执行 1 ，获取下一行left
总结：
    1. 两个表都只遍历一次
    
SortMergeJoinExec#doExecute ==> 数据读取 ==》 shuffle => sort => 匹配合并
//在executedPlan阶段，left和right为InputAdapter,其中在上层会插入SortExec节点,所以需要先shuffle,
//left和right执行execute()方法后返回排序后的RDD
//zipPartitions将left,right的partition,一一组合成一个新的partition
left.execute().zipPartitions(right.execute()) { (leftIter, rightIter) =>
    //初始化排序器，根据key的类型，使用相对应类型的增序排序器，用来比较left和right的key
    val keyOrdering = newNaturalAscendingOrdering(leftKeys.map(_.dataType))
   // 根据不同的连接类型匹配相对于的处理
   joinType match {
        // 内连接方式 ， inner 和 cross,生成一个能处理内连接的iterator
        case _: InnerLike =>
            new RowIterator {
              // 初始化SortMergeJoinScanner 实现sortmerge的工具类
              val smjScanner =new SortMergeJoinScanner(
              createLeftKeyGenerator(),// 是一个left 表的连接key的Projection,也就是age字段的投影，可以是多个字段，如果连接有多个字段
              createRightKeyGenerator(),// 同理
              keyOrdering,  // 排序器
              RowIterator.fromScala(leftIter), // 左表一个分区的数据，
              RowIterator.fromScala(rightIter), // 右表一个分区的数据
              inMemoryThreshold,
              spillThreshold
            )
            //匹配生成部分，有迭代器处理
            override def advanceNext(): Boolean = {
                // 通过SortMergeJoinScanner扫描匹配的行，并保存
                smjScanner.findNextInnerJoinRows()
            }
            override def getRow: InternalRow = resultProj(joinRow)
            }
        // 左外连接
        case LeftOuter =>
            // 调用findNextOuterJoinRows ，以左表为基准，找到匹配行返回，找不到null行代替GenericInternalRow
            new LeftOuterIterator
        // 右外，全外类似
        // LeftSemi
        case LeftSemi =>
            区别：getRow: InternalRow = currentLeftRow 返回结果只包含匹配的左边的行
        case LeftAnti =>

   }
}

*/


```


## shuffle hash join
*   匹配条件
    -   !conf.preferSortMergeJoin （没有指定使用SortMerge） && 
        canBuildRight(joinType) && 
        canBuildLocalHashMap(right) (判断单个parition是否足够小，可以建立hash table)
           && muchSmaller(right, left)  (判断一个表是否是比另一个表大小的3倍，只有足够小的表建立hash table )
        -   spark.sql.join.preferSortMergeJoin 默认为true (默认不使用shuffle,使用sort)
        -   plan.stats.sizeInBytes < conf.autoBroadcastJoinThreshold * conf.numShufflePartitions
        -   a.stats.sizeInBytes * 3 <= b.stats.sizeInBytes
    -   !RowOrdering.isOrderable(leftKeys) // 或者 连接键不能排序
*   一个大表，一个小表，小表足够小的情况
```
spark.conf.set("spark.sql.autoBroadcastJoinThreshold", 10) // 关闭 broadcast hit
spark.conf.set("spark.sql.join.preferSortMergeJoin","false")
//1 加载数据成DataFrame
val df = spark.read.json("examples/src/main/resources/people.json")
var df2 = spark.read.json("examples/src/main/resources/people.json")
df2 = df2.union(df2).union(df2)
val rs = df.join(df2,df.col("age") === df2("age")) 
println(rs.count())

/**
生成对应的executePlan为ShuffledHashJoinExec
    left,right为ShuffleExchangeExec
        根据key 做 hashPartition
    buildSide=BuildLeft，right为大表
// 大表为streamedPlan 也就是right, 小表为buildPlan 也就是left
// streamedPlan.execute() 与 buildPlan.execute()为 安装 key hash后的rdd
// zipPartitions 将两个rdd 一对一组合
streamedPlan.execute().zipPartitions(buildPlan.execute()) { (streamIter, buildIter) =>
      val hashed = buildHashedRelation(buildIter) // 根据小表的一个partition的数据以及key,创建一个HashedRelation ,相当与一个hash表
      // hashedRelation.get(joinKeys(srow))，直接根据大表的key匹配
      join(streamIter, hashed, numOutputRows, avgHashProbe)
    }


*/

```

## broadcast hash join
*   匹配条件
    -   canBroadcastByHints ， join时指定broadcast(df)
    -   canBroadcastBySizes, 数据量小于autoBroadcastJoinThreshold
```
/**
生成对应的executePlan为BroadcastHashJoinExec
BroadcastHashJoinExec#doExecute
    val broadcastRelation = buildPlan.executeBroadcast[HashedRelation]() // 将小表广播出去
    // 遍历每个大表的partition,直接通过广播地址获取hash表，用来匹配key
    // 而 shuffle hash join的hash表为小表的一个partion
    streamedPlan.execute().mapPartitions { streamedIter =>
      val hashed = broadcastRelation.value.asReadOnlyCopy()
      TaskContext.get().taskMetrics().incPeakExecutionMemory(hashed.estimatedSize)
      join(streamedIter, hashed, numOutputRows, avgHashProbe)
    }


*/
```





