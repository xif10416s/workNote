## 以json文件为例子：对json文件内容进行查询过滤操作

###  图解
![](../../images/spark_sql_base_process.jpg)

###  说明
-   1 .    Client 端 读取json文件转换为Dataframe : </br>
    `spark.read.json("../../spark/main/resources/people.json")`   
    -   1.1  构建[DataSource](0-SparkSql_base.md/#DataSource)对象，数据源 : </br>
    `DataSource.apply(sparkSession,paths = paths,..)`
        -   定义一个数据源，什么格式读写数据
    -   1.2  生成统一的数据处理对象baseRelation: [BaseRelation](#BaseRelation)，spark sql处理的数据源可以是文件系统中的json文件，纯文本文件等，也可以是关系数据库中的数据jdbc，所以需要抽象成统一的数据处理方式，知道数据结构，以及数据如何读取  ：</br>
    `DataSource#resolveRelation(): BaseRelation `
        -   FileFormat的情况解析的同时会做类型推断获取schema => StructType
            +   生成一个DataSet,执行select查询
            +   执行dataSet获取一部分sample数据，根据json内容推断json有哪些字段是什么类型返回
                *   会触发job任务执行
    -   1.3  将baseRelation封装成[LogicalRelation](#LogicalRelation)，主要将schema转换成 [AttributeReference](#AttributeReference) 数组
        +    由baseRelation适配成了一棵LogicalPlan，可以进行查询计划的逻辑处理
        +    数据结构从schema描述转换成AttributeReference描述，以便逻辑计划树操作
    -   1.4  将LogicalRelation封装成Dataframe,类型为Row的[Dataset](#Dataset)，先初始化一下QueryExecution检查一下是否可以执行，然后再封装Dataset: </br>
    `Dataset.ofRows(self, LogicalRelation(baseRelation))`
        -   1.4.1  预先检查一下，初始化[QueryExecution](#QueryExecution)，qe.assertAnalyzed() 执行关系查询的工作流，初始化各个阶段的状态analyzed，withCachedData，optimizedPlan，sparkPlan，executedPlan:</br>
        `val qe = sparkSession.sessionState.executePlan(logicalPlan)  //Dataset` </br>
        `def executePlan(plan: LogicalPlan): QueryExecution = new QueryExecution(sparkSession, plan)     //SessionState`
            -    1.4.1.1  分析阶段 analyzed: LogicalPlan </br>  通过 [Analyzer](#Analyzer)#execute，遍历plan的所有节点应用配置的一系列规则，一系列规则rule，检查查询语句中使用的对象是否准确，如表和列是否存在，是否包含*需要展开，使用的方法是否存在
            -    1.4.1.2  使用缓存 withCachedData: LogicalPlan </br> CacheManager#useCachedData ，遍历所有节点查看是否缓存有相同结果的计划 </br>
            `cachedData.find(cd => plan.sameResult(cd.plan))`
            -    1.4.1.3  优化阶段 optimizedPlan: LogicalPlan </br>
                [SparkOptimizer](#SparkOptimizer)#execute,应用一些列优化规则，优化查询逻辑
            -    1.4.1.4  逻辑计划转物理计划  sparkPlan: SparkPlan </br>
                [SparkPlanner](#SparkPlanner)#plan(plan: LogicalPlan): Iterator[SparkPlan]，对应各种操作的物理计划，包含java代码自动生成
            -    1.4.1.5  准备执行的计划 executedPlan: SparkPlan  </br>
                `preparations.foldLeft(plan)` 在计划执行前应用一些规则
                +   CollapseCodegenStages，如果支持代码生成，在顶层插入WholeStageCodegenExec物理计划，支持全局代码生成
        -   1.4.2  实例化Dataset[Row]对象也就是Dataframe </br>
            `new Dataset[Row](sparkSession, logicalPlan, RowEncoder(qe.analyzed.schema))`
            -   1.4.2.1  根据已经分析好的plan的schema初始化RowEncoder,外部的行  和  spark sql内部的二进制表现形式类型的相互转换
                -   BooleanType -> java.lang.Boolean
                -   StringType -> String
            -   1.4.2.2  初始化QueryExecution ※ 新的对象，没有使用1中初始化的QueryExecution对象
            -   1.4.2.3  实例化Dataset[Row]对象,分析环境准备好
                -   `this(sparkSession, sparkSession.sessionState.executePlan(logicalPlan), encoder)`
                -   logicalPlan: LogicalPlan，已经分析好的逻辑计划
                -   exprEnc: ExpressionEncoder[T] = encoderFor(encoder)，表达式编码器准备
-   2 . Client端dataframe的Transformation操作（相当于装饰器模式，一层层添加功能） :</br>
    `qf = df.filter("age > 15").select($"age" + 1).limit(100)`   
    -   2.1  过滤操作filter("age > 15")，解析表达式生成新的Dataset，将原来的逻辑计划添加了GreaterThan过滤的功能，Dataset#filter(conditionExpr: String): Dataset[T]
        -   2.1.1  解析条件语句成表达式，SparkSqlParser#parseExpression(sqlText: String): Expression , "age > 15"  ==> GreaterThan 表达式
        -   2.1.2.  将表达式GreaterThan封装成Column对象
        -   2.1.3.  将Column对象封装成Filter逻辑计划,` Filter(condition.expr, logicalPlan)`
        -   2.1.4.  生成新的Dataset(sparkSession, logicalPlan)返回
    -   2.2. 投影操作select($"age" + 1)，`select(cols: Column*): DataFrame`
        -   2.2.1.  根据列名称包装成Project逻辑计划
        -   2.2.2.  生成新的Dataset(sparkSession, logicalPlan)返回
    -   2.3.  上限操作limit(100)
        -   2.3.1.  封装成Limit逻辑计划， Limit(Literal(n), logicalPlan)
        -   2.3.2.  生成新的Dataset(sparkSession, logicalPlan)返回
    -   2.4.  总结
        -   2.4.1.    每次Transformation操作都会封装成新的logicPlan 
        -   2.4.2.    生成新的QueryExecution： 分析logiclPlan->优化->转化成对应的物理计划->在顶层元素插入全局代码生成物理计划（WholeStateCodegenExec） 
        -   2.4.3.    封装成新的Dataset
    -   2.5.  最终QueryExecution结构</br>
        -   逻辑计划结构optimizedPlan: LogicalPlan：
            +   GlobalLimit
                +    LocalLimit    
                    +   Project
                        +   Filter
                            +   LogicalRelation
                                +   HadoopFsRelation
        -   可执行的物理计划executedPlan: SparkPlan：
            +   CollectLimitExec
                +   WholeStateCodegenExec
                    +   ProjectExec
                        +   FilterExec
                            +   RowDataSourceScanExec
                                +   rdd = FileScanRdd
                                +   relation = HadoopFsRelation
                                +   schema = StructType
    

-   3 .  Client端dataframe的Action操作,`qf.collect()`,执行queryExecution的executedPlan的方法计算</br>
    `queryExecution.executedPlan.executeCollect().map(boundEnc.fromRow)`
    -   3.1.  执行收集操作，调用顶层对象CollectLimitExec的executeCollect </br>
        `executeCollect(): Array[InternalRow] = child.executeTake(limit)`
    -   3.2.  获取前n行数据，child 对象为WholeStateCodegenExec是一个SparkPlan,SparkPlan#executeTake
        -   3.2.1.  获取byte数组的RDD,为了快速序列化 </br> 
            childRDD = getByteArrayRdd(n: Int = -1): RDD[Array[Byte]]
            -   3.2.1.1.  执行物理计划，获取RDD[InternalRow]，每个partion的Iterator应用了查询逻辑生成的代码：  </br>
                `WholeStageCodegenExec#doExecute: RDD[InternalRow]   // sparkPlan的接口方法`
                -   3.2.1.1.1.  动态生成代码类 ，(ctx, cleanedSource) = WholeStageCodegenExec#doCodeGen,成员结构:</br> 
                    实例化方法：</br>
                    `public Object generate(Object[] references) {`</br>
                    `return new GeneratedIterator(references);}`</br>
                    `}`</br>
                    逻辑处理迭代类GeneratedIterator：</br>
                    `final class GeneratedIteratorextends org.apache.spark.sql.execution.BufferedRowIterator`</br>
                    `public void init()//初始化方法`</br>
                    `${ctx.declareAddedFunctions()}//声明函数`</br>
                    `protected void processNext() //迭代处理函数，最主要的处理方法`
                    -   3.2.1.1.1.1   processNext(): 每一行数据处理逻辑代码生成 ,内容代码，遍历孩子节点的produce方法，直到最终数据源的produce,</br>    `code = child.asInstanceOf[CodegenSupport].produce(ctx, this)`
                        -   3.2.1.1.1.1.1   调用WholeStageCodegenExec的child的CodegenSupport接口的produce方法生成代码,既ProjectExec#produce 
                        -   3.2.1.1.1.1.2   ProjectExec#doProduce调用了child的produce ,既FilterExec#produce 
                        -   3.2.1.1.1.1.3    FilterExec#doProduce 调用了RowDataSourceScanExec的produce方法 
                        -   3.2.1.1.1.1.4   RowDataSourceScanExec#doProduce，数据源物理计划,produce负责生产数据，参数准备
                        </br>
                        //代码模板，生成代码的结构</br>
                        s""" </br>
                               |while ($input.hasNext()) { //加载数据，遍历每一行</br> 
                               |  InternalRow $row = (InternalRow) $input.next(); </br>
                               |  $numOutputRows.add(1); </br>
                               |  ${consume(ctx, columnsRowInput, inputRow).trim} </br>
                               |  if (shouldStop()) return; </br>
                               |} </br>
                             """ </br>              
                        -   3.2.1.1.1.1.5  使用当前sparkplan的produce生成的列或者行，会调用parent的doConsumer(),RowDataSourceScanExec#consume  
                        -   3.2.1.1.1.1.6  FilterExec#doConsume，实现age>15的过滤代码，调用consume方法，会调用parent的doConsumer()
                        -   3.2.1.1.1.1.7  ProjectExec#doConsumer,字段选择代码，调用consume方法，会调用parent的doConsumer()
                        -   3.2.1.1.1.1.8  WholeStageCodegenExec#doConsumer，结果复制代码生成，consume调用链结束，返回RowDataSourceScanExec#doProduce方法
                        -   3.2.1.1.1.1.9  最终生成的代码内容： </br>
                        `// 每一行处理逻辑生成代码 ` </br>
                        `while (scan_input6.hasNext()) {`</br>
                        `    //获取输入行`</br>
                        `    InternalRow scan_row6 = (InternalRow) scan_input6.next();`</br>
                        `    scan_numOutputRows6.add(1);`</br>
                        `   //判断值是否是null`</br>
                        `    boolean scan_isNull13 = scan_row6.isNullAt(0);`</br>
                        `    //获取值`</br>
                        `    long scan_value13 = scan_isNull13 ? -1L : (scan_row6.getLong(0));`</br>
                        `   //null值跳过`</br>
                        `   if (!(!(scan_isNull13))) continue;` </br>
                        `   boolean wholestagecodegen_isNull2 = false;`</br>
                        `   boolean wholestagecodegen_value2 = false;`</br>
                        `   // age > 15 的代码 FilterExec的doConsume生成`</br>
                        `   wholestagecodegen_value2 = scan_value13 > 15L;`</br>
                        `   if (!wholestagecodegen_value2) continue;`</br>
                        `   wholestagecodegen_numOutputRows.add(1);`</br>
                        `   boolean project_isNull32 = false;`</br>
                        `   long project_value32 = -1L;`</br>
                        `   // $"age" + 1 的代码 ProjectExec的doConsume生成`</br>
                        `   project_value32 = scan_value13 + 1L;`</br>
                        `   project_rowWriter8.write(0, project_value32);`</br>
                        `   append(project_result8);`</br>
                        `       if (shouldStop()) return;`</br>
                        `}`
                        -   3.2.1.1.1.1.10  代码生成过程总结：
                            +   起点：从顶层对象WholeStageCodegenExec开始，级联调用child的produce方法
                            +   CodegenSupport代码模型：理解成java中io的装饰器模式，CollectLimitExec，WholeStateCodegenExec，ProjectExec，FilterExec都是增强处理节点（过滤流），RowDataSourceScanExec是最底层数据处理节点（节点流），增强处理节点的produce基本上直接调用child的produce,直到最底层数据处理节点，数据处理节点构造整个代码生成
                            +   produce方法： 构建processNext的处理代码，既每一条记录如何处理的逻辑代码
                                *   准备好输入对象input，以及下一行对象inputRow等
                                *   准备while代码，遍历每一行，级联调用parent的consume方法，生成处理这一行代码，
                            +   consume方法：逻辑功能处理代码的实现，如：FilterExec -> 过滤功能代码：</br>
                            `boolean wholestagecodegen_value2 = false;`</br>
                            `wholestagecodegen_value2 = scan_value13 > 15L;`</br>
                            `if (!wholestagecodegen_value2) continue;`  
                -   3.2.1.1.2.  编译生成的代码,检查编译错误：</br>
                    `CodeGenerator.compile(cleanedSource)`
                -   3.2.1.1.3.  获取数据输入RDD,最终是RowDataSourceScanExec的Rdd，类型FileScanRdd,
                    `val rdds = child.asInstanceOf[CodegenSupport].inputRDDs()`
                    -   解析时DataSourceStrategy，匹配生成：</br>
                        `toCatalystRDD(l, baseRelation.buildScan())//也就是HadoopFsRelation#buildScan方法生成RDD`
                -   3.2.1.1.4.  执行转换操作，将生成的代码逻辑应用到输入Rdd，为每个rdd的partion生成新的Iterator对象，使得Iterator的next方法调用生成代码的processNext方法，也就是应用生成的对每一行数据处理的逻辑 
                    +   3.2.1.1.4.1   获取第一个Rdd（默认只有一个）,遍历每个partion，每个partion就是一组数据
                        *   `rdds.head.mapPartitionsWithIndex { (index, iter) =>`
                    +   3.2.1.1.4.2   编译代码获取类clazz：
                        *   `val clazz = CodeGenerator.compile(cleanedSource)`
                    +   3.2.1.1.4.3   根据编译的类clazz反射实例化对象buffer：
                        *   `val buffer = clazz.generate(references).asInstanceOf[BufferedRowIterator]`
                    +   3.2.1.1.4.4   初始化buffer,把数据集合传递进去
                        *   `buffer.init(index, Array(iter))`
                    +   3.2.1.1.4.5   生成新的Iterator，next方法重写为buffer.next()
                        *   buffer.next()也就是生成代码类GeneratedIterator的processNext方法，包含了生成的处理逻辑代码
            -   3.2.1.2.   执行Rdd转换操作，按格式将RDD[InternalRow]转换为RDD[Array[Byte]],既java object对象转为字节，因为rdd处理时使用的是Byte数组，序列化和传输速度快：</br>
                [size] [bytes of UnsafeRow] [size] [bytes of UnsafeRow] ... [-1]
                +   buffer =4k： </br>
                `val buffer = new Array[Byte](4 << 10)  // 4K`
                +   压缩：</br>
                `val codec = CompressionCodec.createCodec(SparkEnv.get.conf)`
        -   3.2.2.   运行job,遍历每个partion的Iterator,遍历每个元素调用next方法，执行生成代码的逻辑，获取返回数组,主要是RDD执行逻辑FileScanRDD:
            +   `val res = sc.runJob(childRDD,
        (it: Iterator[Array[Byte]]) => if (it.hasNext) it.next() else Array.empty, p)`            
        -   3.2.3.  将返回数组的每个元素从byte数组解码成java对象InternalRow
            +   `res.foreach { r =>
        decodeUnsafeRows(r.asInstanceOf[Array[Byte]]).foreach(buf.+=)
        }`     
        -   3.2.4.  limit功能是在最后获取全部结果集后获取前n条记录,最终返回结果：
            +   `if (buf.size > n) {
      buf.take(n).toArray
    } else {
      buf.toArray
    }`
    -   3.3   InternalRow 转出Java类型，返回数组，collect执行完成
        +   `map(boundEnc.fromRow)`

###  全局代码生成WholeStateCodegenExec生成代码 <== 在原始RDD基础上添加逻辑操作
*    rdd上的操作最终就是以partition[Iterator]为单位，在每个partition上对每个元素执行逻辑操作
*    全局代码生成 = 输入RDD[inputRDD] + Iterator Class[包含元素处理逻辑] 
    -    在inputRDD上map 操作，将原始partition 的Iterator 转换成包含逻辑处理的新Iterator
```
 // rdds为inputRDD
 rdds.head.mapPartitionsWithIndex { (index, iter) => // 遍历每个partition
        val clazz = CodeGenerator.compile(cleanedSource) // 编译动态生成的类
        val buffer = clazz.generate(references).asInstanceOf[BufferedRowIterator] //实例化动态类
        buffer.init(index, Array(iter))// 初始化对象，并将partition传入
        new Iterator[InternalRow] { // 转换成 新的 iterator
          override def hasNext: Boolean = {
            val v = buffer.hasNext // 给partition的每个元素执行动态代码生成的逻辑
            if (!v) durationMs += buffer.durationMs()
            v
          }
          override def next: InternalRow = buffer.next()
        }
```
*   inputRDD
    -   可以是数据源的RDD,如RowDataSourceScanExec#inputRDDs ==> rdd :: Nil
    -   也可以是 InputAdapter的代理RDD,需要调用child#execute()获取输入RDD
*   全局生成代码 继承自BufferedRowIterator 的迭代器
    -   generate实例化类，相关引用传入
    -   init初始化类，传入输入input
    -   processNext迭代处理方法，所有生成代码produceCode执行的地方
```
// 处理代码结构，全局代码最底层计划会生成这个结构如：RowDataSourceScanExec，InputAdapter
// 就是一个while循环，遍历partition的所有元素，添加逻辑代码处理这些元素
 while ($input.hasNext()) {
  InternalRow $row = (InternalRow) $input.next();
  $numOutputRows.add(1);
  ${consume(ctx, columnsRowInput, inputRow).trim}// 各个处理计划对应如何处理逻辑部分
  if (shouldStop()) return;
}
```


###   WholeStateCodegenExec 与  InputAdapter
+   在连续的支持代码生成计划顶层添加WholeStateCodegenExec物理计划
    *   WholeStateCodegenExec表示有若干连续的几个child计划可以生成动态代码
    *   最后一个child计划的 doProduce 构造一个while结构的代码块 并调用parent的doConsume,如：RowDataSourceScanExec，InputAdapter
+   如果不支持代码生成，则在上层添加InputAdapter层 （如果是第一层不支持不添加） 
    *   在代码生成过程中起到承上启下作用
        -   结束上一层WholeStateCodegenExec（也就是doProduce 构造一个while结构的代码块
        -   inputRdds调用child#execute，将调用链传递给child,
    +   通过InputAdapter，可以有多层WholeStateCodegenExec
```
物理计划：
sparkPlan:SupportCodegen
  + child:SupportCodegen
    + child:SupportCodegen
     + child:UnSupportCodegen
      + child:SupportCodegen
         + child:UnSupportCodegen
           + child:SupportCodegen
可执行物理计划 ，全局代码生成计划插入后：
 sparkPlan:WholeStateCodegenExec  //第一层动态代码开始,在连续的3个支持代码生成的最上层
    +child:SupportCodegen         
      + child:SupportCodegen   
        + child:SupportCodegen  
         + child : InputAdapter //第一层结束 ， 在不支持代码生成的上一层
            + child:UnSupportCodegen
             +  child:WholeStateCodegenExec  //第二层动态代码开始
              + child:SupportCodegen
                + child : InputAdapter //第二层结束
                 + child:UnSupportCodegen
                  +  child:WholeStateCodegenExec  //第三层动态代码开始
                     + child:SupportCodegen

```        


## 代码生成详细过程
![](../../images/spark_sql_souce_generate.jpg)

### 计划生成与转换
*   根据对Dataset的操作会生成对应的逻辑计划
    -   filter 操作 ==>  Filter
    -   select($"age" + 1) ==> Project
*   逻辑计划经过优化后转换成相对应的物理计划
    -   Filter ==> FilterExec
    -   Project ==> ProjectExec
*   物理计划执行前进一步优化物理计划
    -   CollapseCodegenStages ==>  添加全局代码生成
        +   WholeStateCodegenExec物理计划添加
        +   InputAdapter物理计划添加
        
### 可执行物理计划执行executedPlan: SparkPlan执行
1.  入口WholeStateCodegenExec#doExecute方法，返回一个包含一组逻辑的RDD
1.  动态代码类生成doCodeGen
    -   准备逻辑处理代码
        +   while循环遍历partition每个元素循环体内嵌入动态生成的逻辑代码
1.  获取输入rdd:inputRDDs
    1.  可能是数据源的数据RDD,如：RowDataSourceScanExec
    2.  也可能是InputAdapter返回的child#execute执行的rdd
1.  遍历inputRDDs的partitions，将partition的iterator包装成动态代码类的iterator
    1.  新的iterator执行了生成代码的逻辑

### RDD 与 Dataset 基本操作对比
####    同样2个操作，一个filter,一个map
|        | RDD          | Dataset  |
| ------------- |:-------------:| -----|
| filter操作     | 新的MapPartitionsRDD| 与map操作相同MapPartitionsRDD |
| map 操作     | 新的MapPartitionsRDD   |  连续支持CodegenSupport的都对应相同的MapPartitionsRDD |
| 转换操作调用| 虚拟调用</br>Iterator.map\[B\](f: A => B)  </br>Iterator.filter(p: A => Boolean)    |   动态代码，直接方法调用</br> while (scan_input6.hasNext()) {</br>// filter 操作代码实现</br>// age + 1 代码操作实现</br>}</br> |
|  数据压缩 | 无  | 有，UnsafeRow格式，容易压缩 |
|  执行逻辑优化| 无 | 有,一些列优化规则 |


