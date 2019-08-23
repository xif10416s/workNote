#   shuffle过程
*   https://blog.csdn.net/u012102306/article/details/51637732

#   shuffle 是什么：
*   为什么要shuffle：
    -   针对两类操作： 聚合（groupby） +  排序（sortby)
        +   这两类操作每个partition需要用到所有其他所有partition的数据，也就是宽依赖
*   shuffle做了什么：
    +   分成两个阶段：
        *   map端shuffle witer
            -   读取数据源，每个partition根据key hash数据分成n个组，n为partition数，相当于根据key 的hash重新分区了数据，达到单个partition数据group 的效果
            -   一个partiton的分组数据会写入到 >= n的临时文件，最后合并成一个大的文件
        *   reduce端的shuffle reader
            -   每个分区根据hash key的分组方式读取本地或者远程的相对应的分区的数据，最终每个分区变成了新的分组后的分区的数据，达到了全局分组的效果

##  shuffle处理器，一共三类：
### BypassMergeSortShuffleHandle 对应 BypassMergeSortShuffleWriter
*   使用场景：
    -   不需要在map端做Combine，即mapSideCombine=false
    -   分区数小于 ： bypassMergeThreshold: Int = conf.getInt("spark.shuffle.sort.bypassMergeThreshold", 200)
*   基本操作：
    -   在map端写partition个文件，然后合并成一个文件

### SerializedShuffleHandle 对应 UnsafeShuffleWriter
*   使用场景：
    -   支持序列化数据重排对象，直接操作序列化数据排序
    -   没有定义聚合函数
*   基本操作：
    -   数据写入SerializationStream在直接内存中排序后写入partition个文件，最后合并成一个文件

### BaseShuffleHandle 对应 SortShuffleWriter
*   使用场景： 
    -   不是上面2中的情况
*   基本操作：



##  RDD#groupBy操作为例子【BypassMergeSortShuffleWriter】：`val groupRdd = spark.sparkContext.parallelize(Seq(0, 1, 2, 3, 0, 1, 2, 3, 0, 1, 2, 3, 0, 1, 2, 3), 4).groupBy(f => f)`
*   ![](../images/spark_shuffle_groupby.jpg)
*   映射成key，value的pair形式，this.map(t => (cleanF(t), t))
    -   包装成 MapPartitionsRDD
*   隐式调用，PairRDDFunctions#groupByKey
    -   定义三个函数
        +   val createCombiner = (v: V) => CompactBuffer(v)
            *   如何创建合并对象
            *   CompactBuffer为一个高效存储处理的array
        +    val mergeValue = (buf: CompactBuffer[V], v: V) => buf += v
            *    定义如何合并元素，就是往array中添加元素
        +   mergeCombiners = (c1: CompactBuffer[V], c2: CompactBuffer[V]) => c1 ++= c2
            *   定义如何将两个合并对象合并成一个对象
    -   包装成  ShuffledRDD\\[K, V, C\\](self, partitioner)
        .setSerializer(serializer)
        .setAggregator(aggregator)
        .setMapSideCombine(mapSideCombine)
        -   val aggregator = new Aggregator\\[K, V, C\\](
      self.context.clean(createCombiner),
      self.context.clean(mergeValue),
      self.context.clean(mergeCombiners)) 为之前定义的三个合并相关的函数

##  执行actions,触发任务提交，RDD#count
*   RDD形式
    -   ShuffledRDD  <-- 对于pair形式的rdd,隐式调用PairRDDFunctions#groupByKey生成
        +   prev: MapPartitionsRDD <-- groupby时候先映射成key,value的map操作生成，转成pair形式rdd
            *   prev:ParallelCollectionRDD <-- sc.parallelize 生成，实际处理数据的RDD
                -   data : 1 to n
                -   dependencies_: OneToOneDependecy
                    +   _rdd:ParallelCollectionRDD
        +   dependencies_: ShuffleDependency <-- ShuffledRDD的getDependencies时设置
            *   _rdd:MapPartitionsRDD
            *   shuffleHandler: BypassMergeSortShuffleHandle
    -   ShuffleDependency初始化的时候决定用什么shuffleHandle
        +   SortShuffleManager#registerShuffle 方法确定
            *   spark.shuffle.sort.bypassMergeThreshold（默认200） >= rdd的partition数量，使用BypassMergeSortShuffleHandle
            *   supportsRelocationOfSerializedObjects支持使用SerializedShuffleHandle
            *   否则使用BaseShuffleHandle
*   stage划分
    -   ResultStage
        +   parents ： \\[ShuffleMapStage\\]
            *   shuffleDep :  ShuffleDependency
            *   **rdd : MapPartitionsRDD**  <-- shuffle task执行时计算的rdd
        +   **rdd : ShuffledRDD**  <-- 计算result stage时，先计算的是ShuffleMapStage
*   stage提交计算
    -   首先提交最父层stage, 也就是ShuffleMapStage
    -   stage 转换成一组TaskSet，每个task为ShuffleMapTask

## worker端executor执行ShuffleMapTask
*   ShuffleMapTask#runTask运行shuffle task
*   获取shuffleManager: `val manager = SparkEnv.get.shuffleManager`
*   获取ShuffleWriter：` writer = manager.getWriter[Any, Any](dep.shuffleHandle, partitionId, context)`
    -   shuffleHandle为BypassMergeSortShuffleHandle，所以使用了BypassMergeSortShuffleWriter
*   BypassMergeSortShuffleWriter执行写入操作： `writer.write(rdd.iterator(partition, context).asInstanceOf[Iterator[_ <: Product2[Any, Any]]])`
    -   数据被写入worker机器的文件
    -   临时文件路径
        +   /private/var/folders/vd/txsj8y2d227c538pykv3k0nr0000gn/T/spark-4b5c9efc-08c4-44c8-9da2-14a4208c6cba/executor-f05860ee-1029-4925-b792-001391d74d07/blockmgr-d8334fc6-96a0-4190-abad-04f2817ec14d/0c/shuffle_0_0_0.data
    -   可以通过shuffleBlockResolver根据shuffleId和partitionid找到对应的数据文件
    -   rdd.iterator == MapPartitionsRDD.iterator
        +   递归调用
            -   先计算ParallelCollectionRDD.iterator读取实际数据
            -   在计算MapPartitionsRDD.iterator映射成key,value元素pairRDD
        +   最终被写入文件
*    给driver发送结束状态消息，execBackend.statusUpdate(taskId, TaskState.FINISHED, serializedResult)  ， serializedResult 为MapStatus 

## driver端接受到完成消息并处理
*   DagScheduler#handleTaskCompletion
    -   将结果转换成MapStatus，val status = event.result.asInstanceOf[MapStatus]
        +   结果中记录了shuffle中间结果的地址
    -   添加到stage的output
        +   shuffleStage.addOutputLoc(smt.partitionId, status)
    -   等所有shuffleTask完成，将shuffle输出结果注册到mapOutputTracker
        +   mapOutputTracker.registerMapOutputs(
                shuffleStage.shuffleDep.shuffleId,
                shuffleStage.outputLocInMapOutputTrackerFormat(),
                changeEpoch = true)
    -   ShuffleStage执行完成，提交childStage也就是ResultStage执行

##  worker端executor执行ResultTask【rdd : ShuffledRDD】
###    执行ShuffledRDD#iterator
*   获取shuffleReader,SparkEnv.get.shuffleManager.getReader(dep.shuffleHandle, split.index, split.index + 1, context) =>BlockStoreShuffleReader
    -   从当前partition开始到下一个partition,但是下一个partition不被读取，也就是只读取当前partition数据
*   BlockStoreShuffleReader#read读取shuffle之前保存在文件的内容
    -   shuffleTask的输出结果位置都保存在了mapOutputTracker，可以根据shuffleid获取当前partition获取结果信息
    -   读取数据
    -   通过之前定义的Aggregator，合并相同key的元素
*   TODO
    -   一个partiotion只读取当前partition的shuffle数据？
    -   合并细节，夸partition的数据如何合并


## BypassMergeSortShuffleWriter详解
### 基础
-   基于排序的，hash风格的shuffle
-   记录被写入独立的文件，每个reduce 分区 一个 文件，每个文件的内容都是排序的
-   最后把所有分区文件内容整体排序后合并到一个文件中
-   所有记录不会被缓存在内存
*   可以通过IndexShuffleBlockResolver读取文件内容
-   因为每个partition会有单独的序列化和文件流，所以在partition过多的情况性能会下降

### BypassMergeSortShuffleWriter#write
*   生成n个临时文件 ，n为分区数,每个临时文件对应一个blockid
    -   `Tuple2<TempShuffleBlockId, File> tempShuffleBlockIdPlusFile = blockManager.diskBlockManager().createTempShuffleBlock();`
    -    根据File，和bolckid初始化n个 **DiskBlockObjectWriter**
        +    每个需要处理的partition会有所有partition数量的writers,也就是说n个partition会有n*n个 DiskBlockObjectWriter
        +    将jvm对象写入文件，运行追加内容到指定的block
*   遍历所有rdd的partition的数据，hash[key]决定用哪个DiskBlockObjectWriter写入
    -   partitionWriters[partitioner.getPartition(key)].write(key, record._2());
    -   将rdd的一个partition数据写入到n个文件中去（n为partition数）
*   刷新文件写入提交所有数据并返回写入数据的信息
    -   commitAndGet(): FileSegment
        +    FileSegment(val file: File, val offset: Long, val length: Long) ，文件的数据访问信息

### BypassMergeSortShuffleWriter#writePartitionedFile合并之前的n个文件到一个文件
*   合并文件，Utils.copyStream(in, out, false, transferToEnabled);
    -   合并完成后删除之前的文件
    -   返回每个partition的对应的写入文件的长度的数组
*   创建索引文件，记录每个block的起始位置和长度.
*   返回shuffle结果MapStatus，shuffle 写入完成


## BlockStoreShuffleReader#read读取shuffle内容
### ShuffleBlockFetcherIterator
*   获取block的iterator,从本地或者远程获取指定分区的block数据
*   获取当前分区的所有shuflle block信息， mapOutputTracker.getMapSizesByExecutorId(handle.shuffleId, startPartition, endPartition)
*   initialize ，初始化
    -   划分block,哪些需要远程获取，哪些本地获取，splitLocalRemoteBlocks
        +   本地保存在localBlocks
        +   远程收集在fetchRequests
    -   拉取远程block，fetchUpToMaxBytes
        +   results.put(new SuccessFetchResult(BlockId(blockId), address, sizeMap(blockId), buf,
              remainingBlocks.isEmpty))
    -   拉取本地block,fetchLocalBlocks
        +   results.put(new SuccessFetchResult(blockId, blockManager.blockManagerId, 0, buf, false))

### 合并各个task计算的相同key的结果
*   dep.aggregator.get.combineValuesByKey(keyValuesIterator, context)

###  执行指定的排序
*   通过ExternalSorter排序



##  SortShuffleWriter（不支持SerializedShuffle也不支持BypassMergeSortShuffle的情况）

### 图解
![](../images/spark_shuffle_writer.jpg)

### ExternalSorter
*   排序并合并相同的key的元素
*   通过Partitioner计算key应该在哪个partiton,通过自定义的Comparator在partition内做元素的排序
*   不断将数据放入内存buffer，在buffer中根据partitonid和key排序
    -   需要合并相同key的操作使用PartitionedAppendOnlyMap
        +   通过聚合函数重新计算相同key的值，然后替换
    -   反之使用PartitionedPairBuffer
        +   简单的添加数据
*   当buffer内存满的时候，会刷新到文件中，文件中的内容先根据partitionid排序，相同paritionid内的元素根据key排序
    -   对于每个文件会记录每个partition的保存元素的个数，这样就不需要保存partitionid也能区分是partition
*   当调用iterator时，输出文件和buffer中的剩余数据会合并到一个文件中，相同的方式排序和聚合
*   调用stop,删除临时文件

### 入口函数ExternalSorter#insertAll
*   遍历iterator数据插入buffer
*   每次插入都会检查是否到达buffer上线，并相应的刷新到文件，清空内存
    -   每次刷新新产生一个根据partition和key排序后的文件

### buffer类型
####    PartitionedPairBuffer，非聚合，简单追加
*   buffer结构 => data = new Array[AnyRef](2 * initialCapacity)
*   数据保存方式 => insert ,每次保存2个元素
    -   第一个元素是tuple类型，key为partitionid，value为key
        -   data(2 * curSize) = (partitionid, key.asInstanceOf[AnyRef])
        -   partitionid的值是根据给定的partitioner通过可以计算得到，partitionid= partitioner.getPartition(key)
    -   第二个元素为value
        +   data(2 * curSize + 1) = value.asInstanceOf[AnyRef]
    -   data = [(pid1,key1),value1,(pid2,key2),value2,(pid3,key3),value3,....]
    -   当元素个数达到指定容量时候，进行扩容，扩容后的大小为当前长度的2倍
*   内存数据排序,内存上限刷新文件时触发 => partitionedDestructiveSortedIterator
    -   将data的数据按partitionid，相同partitionid的按key排序完成
    -   返回iterator，读取pair值 ，Iterator[(partitinid,key),value]
        +    val pair = (data(2 * pos).asInstanceOf[(Int, K)], data(2 * pos + 1).asInstanceOf[V])

####    PartitionedAppendOnlyMap，合并相同key
*   buffer结构 =>data = new Array[AnyRef](2 * capacity)
*   数据保存方式 => map.changeValue((getPartition(kv._1), kv._1), update)
    -   首先要定义更新函数 updateFunc
        +   ` val update = (hadValue: Boolean, oldValue: C) => {
        if (hadValue) mergeValue(oldValue, kv._2) else createCombiner(kv._2)
      }`
        -   对应key有值的时候合并值，没有的时候新建合并对象
    -   key为（partitionid,key)的tuple , partitionid=partitioner.getPartition(key)
    -   根据key计算存储位置，pos = rehash(k.hashCode) & mask
        +   mask为data的容量大小，随机散列在data数组上
    -   获取pos位置的key,curKey = data(2 * pos)
        +   如果pos位置还没有key，直接保存新数据
            *   data(2 * pos) = k
            *   data(2 * pos + 1) = newValue.asInstanceOf[AnyRef]
        +   如果pos位置有key,而且key一致 ，更新当前key的值
            *   val newValue = updateFunc(true, data(2 * pos + 1).asInstanceOf[V])
            *   data(2 * pos + 1) = newValue.asInstanceOf[AnyRef]
        +   如果pos位置有key,但是不一致，重新计算位置
            *   pos = (pos + delta) & mask  <= 偏移量 delta = 1
*   排序，data上限时触发，destructiveSortedIterator


###    合并文件和buffer中剩余的数据到一个文件，ExternalSorter#writePartitionedFile
*   准备读取所有文件对象， val readers = spills.map(new SpillReader(_))
*   准备buffer中的数据对象，val inMemBuffered = inMemory.buffered
*    (0 until numPartitions).iterator.map 遍历所有partitionid
    -    从内存buffer中取partition为p的iterator，val inMemIterator = new IteratorForPartition(p, inMemBuffered)
    -    从分割文件中取partition为p的文件的一小批数据的iterator，readers.map(_.readNextPartition()) 
    -    合并成所有partition的p的数据的iterator ，iterators
    -    对这些iterator排序，放入一个heap,获取拥有最小值的iterator[通过比较iterator的第一个元素，因为所有iterator的数据已经排序]
        +    将最小的元素写入合并文件
        +    如果iterator还有下一个元素继续放入heap
            *    如果iterator的批数据处理完成就读取下一批数据
*   创建索引文件，记录每个block的起始位置和长度.
*   返回shuffle结果MapStatus，shuffle 写入完成




 


   