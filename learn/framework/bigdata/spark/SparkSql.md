#Spark Sql
##设计目的：
1.  支持spark Rdd 和外部数据的关系处理
2.  高性能的关系数据操作
3.  易于扩展数据源，如半结构化数据，外部数据
4.  可以被其他高级算法使用如graph,machine learning

##基本流程
![](images/4.png)

###以json文件为例子：对json文件内容进行查询过滤操作
-   1 .    Client 端 读取json文件转换为Dataframe : </br>
    `spark.read.json("../../spark/main/resources/people.json")`   
    -   1.1  构建[DataSource](#DataSource)对象，数据源 : </br>
    `DataSource.apply(sparkSession,paths = paths,..)`
    -   1.2  生成统一的数据处理对象baseRelation: [BaseRelation](#BaseRelation)，spark sql处理的数据源可以是文件系统中的json文件，纯文本文件等，也可以是关系数据库中的数据jdbc，所以需要抽象成统一的数据处理方式，知道数据结构，以及数据如何读取  ：</br>
    `DataSource#resolveRelation(): BaseRelation `
    -   1.3  将baseRelation封装成[LogicalRelation](#LogicalRelation)，主要将schema转换成 [AttributeReference](#AttributeReference) 数组
    -   1.4  将LogicalRelation封装成Dataframe,类型为Row的[Dataset](#Dataset)，先初始化一下QueryExecution检查一下是否可以执行，然后再封装Dataset: </br>
    `Dataset.ofRows(self, LogicalRelation(baseRelation))`
        -   1.4.1  预先检查一下，初始化[QueryExecution](#QueryExecution)， 执行关系查询的工作流，初始化各个阶段的状态analyzed，withCachedData，optimizedPlan，sparkPlan，executedPlan:</br>
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
    






##基本组件
##org.apache.spark.sql
#####<span id = 'Dataset'> Datasets </span>
1.  强类型的分布式数据集合
1.  可以并行函数转换处理
1.  DataSet的操作分为transformations 和 actions , Transformations 产生新的 DataSet , actions 执行计算返回结果、Dataset用一个逻辑计划描述数据如何处理计算,action调用时,优化逻辑计划,生成物理计划并行高效执行
1.  通过Encoder
    -   运行时生成的自定义bytecode序列化反序列化,加速,减少网络传输
    -   一些操作无需反序列化就可以操作



#####Encoder
*   将jvm类型为T的jvm对象转换为spark sql 内部的结构表现
*   通常由sparksession隐式转换而来
```
import spark.implicits._
val ds = Seq(1, 2, 3).toDS() // implicitly provided (spark.implicits.newIntEncoder)
```
*   fields
    -   schema : StructType <== 
    -   clsTag: ClassTag[T] <== 保存类型信息，构造array的时候生成该类型的元素


##org.apache.spark.sql.sources
#####<span id = "BaseRelation">BaseRelation</span>
*   代表一组已知schema的元组tuples，spark sql处理数据的统一方式
*   统一的数据处理对象，包含了数据结构，以及如何读取数据


##org.apache.spark.sql.catalyst
#####InternalRow
*   在spark sql中内部使用的抽象类，代表一行
*   只包含列名，和内部对应的类型

##org.apache.spark.sql.catalyst.plans
#####QueryPlan
*   专门指一棵执行计划树
*   QueryPlan的泛型参数仅要求为QueryPlan的子类

##org.apache.spark.sql.catalyst.plans.logical
#####LogicalPlan  extends QueryPlan[LogicalPlan]
*   LogicalPlan比起QueryPlan扩展了resolve相关的操作，还加上了一个statistics变量。 该变量实际上就是一个BigInt，代表计划的执行代价
*   Statistics－－执行计划的代价估计。默认叶子节点的代价为1，非叶子节点的代价为各子结点代价的乘积。不同类型的执行计划通过重载其statistics函数来改变代价计算方式
*   方法
    -   resolveOperators(rule: PartialFunction[LogicalPlan, LogicalPlan]): LogicalPlan <== 遍历所有元素应用rule
    -   resolve(schema: StructType, resolver: Resolver): Seq[Attribute] <== 将StructType 解析成 Attribute

#####LeafNode extends LogicalPlan
*   没有子节点的逻辑计划

#####UnaryNode extends LogicalPlan
*   只有一个子节点

#####BinaryNode extends LogicalPlan
*   2个子节点


##org.apache.spark.sql.catalyst.expressions
#####UnsafeRow extends MutableRow
*   原始内存数据，二进制数据8-byte word per field，方便快速序列化和传输
*   Each tuple has three parts: [null bit set] [values] [variable length portion]

#####Expression extends TreeNode[Expression] 
*   spark sql 语法 或者 api 解析成对应的 experssion，执行时生成相应的java 代码
*   方法
    -   eval(input: InternalRow = null): Any  <== 根据输入的internalRow计算结果
    -   doGenCode(ctx: CodegenContext, ev: ExprCode): ExprCode <== 生成代码

#####LeafExpression extends Expression
*   没有输入表达式

#####UnaryExpression extends Expression
*   一个输入，一个输出

#####BinaryExpression extends Expression
*   两个输入，一个输出

#####UnsafeRow extends MutableRow
*   由原始内存支持的类型取代java对象，spark内部类型，方便序列化反序列化，内存管理
*   每个元组有三部分组成
    -   ［null bit set］用来跟踪字段是否是null，分配了8个字节标记，每隔字段使用一位表示
    -     ［values］每个字段存储一个8字节，保存固定长度字段的类型，对于非固定长度的字段保存offset

##### Substring(str: Expression, pos: Expression, len: Expression) extends TernaryExpression with ImplicitCastInputTypes
*   string截取操作表达式，生成对应的string截取代码

#####Attribute extends LeafExpression with NamedExpression with NullIntolerant


#####<span id = 'AttributeReference'>AttributeReference </span> extends Attribute with Unevaluable
*   在树中由其他操作生成的某个属性的引用
*   属性名称，属性类型（DataType），是否可空，id


##org.apache.spark.sql.catalyst.expressions.codegen
#####ExprCode(var code: String, var isNull: String, var value: String)
*   给定InternalRow的表达式执行生成的java代码


##org.apache.spark.sql.catalyst.trees
#####TreeNode[BaseType <: TreeNode[BaseType]] extends Product
*   语法树的构建都是由一个个TreeNode组成
*   TreeNode指的是用于构建一般的树的结点

##org.apache.spark.sql.catalyst.analysis
#####<span id ='Analyzer'>Analyzer</span>(catalog: Catalog,registry: FunctionRegistry,conf: CatalystConf,maxIterations: Int = 100) extends RuleExecutor[LogicalPlan]
*   sql parser出来的logicalPlan只是针对sql的解析，表是否存在，字段是否存在，字段类型都是未知的
*   通过rules 和　Catalogj解析属性
*   解析步骤
    -   Looking up relations by name from the catalog.
    -   Mapping named attributes, such as col , to the input provided　given operator’s children.
    -   Determining which attributes refer to the same value to give
        them a unique ID
*   batches  ， 一系列规则rule，检查关系和操作是否准确
    -   ResolveRelations
        -   从catalog中检查UnresolvedRelation的表名和别名是否存在
    -   ResolveReferences
        -   检查project投影是否包含*，有的话展开
        -   检查聚合操作是否包含*，如count（*）
        -   检查表连接是否包含有名称冲突的列名
    -   ResolveFunctions
        +   检查使用的函数是否存在
*   execute
    -   执行定义的所有ｒｕｌｅ＝＝＞batches

#####UnresolvedRelation
*   sql 中解析的表名或者别名，实际不一定存在


##org.apache.spark.sql.catalyst.encoders
#####ExpressionEncoder
*   通用jvm对象encoder,java对象和spark sql内部数据结构 internalRow的相互转换
*   字段
    -   schema: StructType <== java类型T对应的结构类型，名称列表+类型列表
    -   serializer: Seq[Expression] <== 一组表达式 ，表示一个类型T的对象 到 internal row 的转换
    -   deserializer: Expression <==  从internalRow转换为对象的表达式
*   方法
    -   toRow(t: T): InternalRow <== 将java对象转换为InternalRow
    -   fromRow(row: InternalRow): T <== 将InternalRow转换为java对象

#####RowEncoder
*   工厂对象，生成 将外部Row转换成spark sql 内部数据对象DataType的子类

##org.apache.spark.sql.types
#####DataType  -- 数据类型
*   spark sql 使用的基本数据类型的基类
*   子类
    -   BooleanType，IntegerType 等


#####StructType -- 结构类型
*   结构类型，由StructField（名称，类型，可否空 三个字段） 数组组成


##org.apache.spark.sql.execution
#####SparkOptimizer extends Optimizer
*   查询规则优化
*   batches
    -   CombineUnions ，Union操作的优化
    -   OptimizeSubqueries，子查询优化
    -   ReplaceIntersectWithSemiJoin，交集操作被替换为左连接
    -   ReplaceDistinctWithAggregate，distinct操作替换为聚合操作
    -   等等

#####SparkPlan extends QueryPlan[SparkPlan]
*   物理操作的基类
*   操作命名规则是名称+Exec结尾
*   子类：
    -   LeafExecNode extends SparkPlan    无字节点
    -   UnaryExecNode extends SparkPlan    一个子节点
    -   BinaryExecNode extends SparkPlan    两个子节点


#####<span id = 'SparkPlanner' >SparkPlanner </span>(val sqlContext: SQLContext) extends SparkStrategies
*   将Optimized Logical Plan变为Physical Plan的规则
*   strategies , 一些列转换策略        
    -   FileSourceStrategy，计划如何扫描一组文件
    -   DataSourceStrategy，计划如何扫描数据源，使用数据源的api,如jdbc
    -   BasicOperators,基本操作
        +   Project ，`execution.ProjectExec(projectList, planLater(child)) :: Nil`
        +   Filter,` execution.FilterExec(condition, planLater(child)) :: Nil`


#####ProjectExec(projectList: Seq[NamedExpression], child: SparkPlan) extends UnaryExecNode with CodegenSupport
*   投影逻辑计划的物理操作
*   投影功能的java代码生成



#####FilterExec(condition: Expression, child: SparkPlan) extends UnaryExecNode with CodegenSupport  
*   过滤计划的物理操作
*   过滤功能的java代码生成

#####SparkStrategies extends QueryPlanner[SparkPlan]
*   self: SparkPlanner => 必须是sparkPlanner的子类
*   成员
    -   EquiJoinSelection extends Strategy 
        +   使用ExtractEquiJoinKeys匹配join操作，根据ｊｏｉｎ的ｋｅｙ和ｓｉｚｅ选择合适的物理计划
        +   通过ExtractEquiJoinKeys匹配选择：
            -   Broadcast　：　广播方式ｊｏｉｎ-－小的一边被广播出去
                -   需要ｊｏｉｎ的中有一个物理大小小于指定的值（SQLConf.AUTO_-BROADCASTJOIN_THRESHOLD =10m)
                -   指定需要broadcast方式
            -   Shuffle hash join
                -   如果一个分区的平均大小足够小，可以生成一个哈希表
            -   Sort merge
        　       -   如果匹配的ｋｅｙ可以排序


#####<span id = 'QueryExecution'>QueryExecution(val sqlContext: SQLContext, val logical: LogicalPlan)</span>
*   执行关系查询的工作流,可以方便的查询执行过程中的中间阶段的状态，按顺序来是：
    1.   analyzed
        +   解析－－调用Analyzer的execute方法，执行各种规则，返回解析后logicPlan
    1.   withCachedData
        +   使用缓存
    1.   optimizedPlan
        +   执行各种逻辑优化
    1.  sparkPlan
        +   根据ｌｏｇｉｃＰｌａｎ的类型生成物理计划
        +   LogicalRelation
            +   DataSourceStrategy
                +    DataSourceScanExec
    1.   executedPlan
        +   在物理计划上扩展
        +   preparations
            +   CollapseCodegenStages
```
if (conf.wholeStageEnabled) {
                WholeStageCodegenExec
                ...
```


#####DataSourceScanExec extends LeafExecNode 


#####RowDataSourceScanExec extends DataSourceScanExec with CodegenSupport
*   从一个relation扫描数据的物理计划节点



#####CodegenSupport extends SparkPlan
*   支持代码生成
*   方法
    -   produce(ctx: CodegenContext, parent: CodegenSupport): String ， 添加注释代码，并调用doProduce生成代码
    -   doProduce(ctx: CodegenContext): String ，生成代码，由子类实现具体方法
    -   consume(ctx: CodegenContext, outputVars: Seq[ExprCode], row: String = null): String ，
    -   doConsume(ctx: CodegenContext, input: Seq[ExprCode], row: ExprCode): String


######InputAdapter(child: SparkPlan) extends UnaryExecNode with CodegenSupport





#####<span id = "WholeStageCodegenExec">WholeStageCodegenExec(child: SparkPlan) extends UnaryExecNode with CodegenSupport </span>
*   把计划的所有子树转操作表达式换成java代码封装到一个java方法里

```
* Here is the call graph of to generate Java source (plan A support codegen, but plan B does not):
 *
 *   WholeStageCodegen       Plan A               FakeInput        Plan B
 * =========================================================================
 *
 * -> execute()
 *     |
 *  doExecute() --------->   inputRDDs() -------> inputRDDs() ------> execute()
 *     |
 *     +----------------->   produce()
 *                             |
 *                          doProduce()  -------> produce()
 *                                                   |
 *                                                doProduce()
 *                                                   |
 *                         doConsume() <--------- consume()
 *                             |
 *  doConsume()  <--------  consume()
 *
 * SparkPlan A should override doProduce() and doConsume().
```
*   方法
    -   doCodeGen(): (CodegenContext, CodeAndComment) ，调用child的produce方法生成代码
    -   doExecute(): RDD[InternalRow] ，调用doCodeGen生成代码，编译并执行代码



##org.apache.spark.sql.execution.joins
#####HashJoin
*   self: SparkPlan =>   子类必须是sparkPlan的子类
*   spark sql表连接相关的join操作

#####class BroadcastHashJoin() extends BinaryNode with HashJoin
*   执行两个sparkplan的inner hash join
*   异步spark job用来计算需要broadcast的关系


##org.apache.spark.sql.execution.datasources
#####<span id = "DataSource">DataSource</span>
*   代表spark sql中插件式数据源的主要类 ， 数据源
*   每种类型的数据源对应各自的数据源提供者对象
    -   json ==>  JsonFileFormat
    -   jdbc ==>  JdbcRelationProvider
*   方法
    -   resolveRelation(checkPathExist: Boolean = true): BaseRelation ，创建BaseRelation对象，用来读取和写入数据到该数据源，以json文件为例：
        1.  匹配对应provider类型
        2.  获取dataSchema（推断数据结构），JsonFileFormat#inferSchema ※ 会提交一个job处理
        3.  生成HadoopFsRelation对象（BaseRelation对象）
    -   lookupDataSource(provider0: String): Class[_] ， 根据format查找对应的数据源provider
*   成员
    -   providingClass: Class[_] , 数据源提供者
    -   backwardCompatibilityMap ， 支持的格式对应的数据源map
    


##org.apache.spark.sql.execution.datasources.json
#####JsonFileFormat extends TextBasedFileFormat with DataSourceRegister
*   json格式的数据源对象
*   方法
    -   inferSchema ， 推断json格式类型，有哪些字段，类型是什么
    -   buildReader(): PartitionedFile => Iterator[InternalRow] , 返回一个函数，读取文件并解析json文件
```
    (file: PartitionedFile) => {
      val linesReader = new HadoopFileLinesReader(file, broadcastedHadoopConf.value.value)
      Option(TaskContext.get()).foreach(_.addTaskCompletionListener(_ => linesReader.close()))
      val lines = linesReader.map(_.toString)
      JacksonParser.parseJson(
        lines,
        requiredSchema,
        columnNameOfCorruptRecord,
        parsedOptions)
    }
```

#####JdbcRelationProvider extends RelationProvider with DataSourceRegister
*   jdbc数据源 ， 读取jdbc数据以及对应的类型schema
*   方法
    -    createRelation（）: BaseRelation，转换成 BaseRelation

#####<span id = 'LogicalRelation' >LogicalRelation </span> extends LeafNode with MultiInstanceRelation
*   是一个没有孩子节点的逻辑计划LogicPlan
*   把schema 转成attribute的集合


##[过去的迭代模型 =》Volcano Iterator Model](https://databricks.com/blog/2016/05/23/apache-spark-as-a-compiler-joining-a-billion-rows-per-second-on-a-laptop.html)
[中文翻译](http://geek.csdn.net/news/detail/77005)


#主要类层次结构
```
TreeNode
    QueryPlan
        SparkPlan
            BinaryNode
            LeafNode
            UnaryNode
            CodegenSupport
                FilterExec
                BroadcastHashJoinExec
                RowDataSourceScanExec
                WholeStageCodegenExec
        LocalNode
            BinaryLocalNode
            UnaryLocalNode
            LeafLocalNode
        LogicalPlan
            LeafNode
                UnresolvedRelation
            UnaryNode
            BinaryNode
    Expression
        BinaryExpression
        LeafExpression
            Attribute
                AttributeReference
        UnaryExpression
```