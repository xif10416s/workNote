#   SparkSql
*   结构化数据处理 ==> RDD + schema
*   可以通过sql语句执行处理
*   支持spark Rdd 和外部数据的关系处理
*   高性能的关系数据操作
*   易于扩展数据源，如半结构化数据，外部数据
*   可以被其他高级算法使用如graph,machine learning

##基本流程
![](../../images/4.png)

##  基本组件
##  sql 项目的core 模块
*   定义如何将数据转换成Dataset & DataFrame，sql api 的入口对象
*   逻辑计划转成物理计划，计划执行

#### SparkSession
*   spark sql api[ Dataset & DataFrame]入口函数
    -   创建 DataFrame 来自于:
        +   rdd: RDD[A]
        +   data: Seq[A]
        +   rowRDD: RDD[Row] ， schema: StructType
        +   rows: java.util.List[Row], schema: StructType
        +   rdd: RDD[_], beanClass: Class[_]
        +   data: java.util.List[_], beanClass: Class[_]
    -   创建 Dataset 来自于:
        +   data: Seq[T]
        +   data: java.util.List[T]
*   直接加载外部数据成为Dataframe的read:DataFrameReader
    -   文件系统
    -   数据库等


####   <span id = 'Dataset'> Datasets </span>
1.  强类型的分布式数据集合
1.  可以并行函数转换处理
1.  DataSet的操作分为transformations 和 actions , Transformations 产生新的 DataSet , actions 执行计算返回结果、Dataset用一个逻辑计划描述数据如何处理计算,action调用时,优化逻辑计划,生成物理计划并行高效执行
1.  通过Encoder
    -   运行时生成的自定义bytecode序列化反序列化,加速,减少网络传输
    -   一些操作无需反序列化就可以操作
   
####   Encoder
*   将jvm类型为T的jvm对象转换为spark sql 内部的结构表现
*   通常由sparksession隐式转换而来
```
import spark.implicits._
val ds = Seq(1, 2, 3).toDS() // implicitly provided (spark.implicits.newIntEncoder)
```
*   fields
    -   schema : StructType <== 
    -   clsTag: ClassTag[T] <== 保存类型信息，构造array的时候生成该类型的元素

###  org.apache.spark.sql.sources
####   <span id = "BaseRelation">BaseRelation</span>
*   代表一组已知schema的元组tuples，spark sql处理数据的统一方式
*   统一的数据处理对象，包含了数据结构，以及如何读取数据

##     sql 项目的catalyst 模块
*   SparkSQL的优化器系统模块
*   主要功能：
    -   Parser，解析，将sql语句解析成LogicalPlan（语法树）
        +   对于sql中的属性和关系并没有检查和解析，UnresolvedAttribute，UnresolvedRelation
    -   Analyzer，分析，遍历plan的所有节点应用配置的一系列规则，一系列规则rule，检查查询语句中使用的对象是否准确
    -   Optimizer，优化，基于规则的优化策略实际上就是对语法树进行一次遍历，模式匹配能够满足特定规则的节点，再进行相应的等价转换

####    org.apache.spark.sql.catalyst.InternalRow
*   在spark sql中内部使用的抽象类，代表一行
*   只包含列名，和内部对应的类型

#####   org.apache.spark.sql.catalyst.expressions.UnsafeRow extends InternalRow
*   原始内存数据，二进制数据8-byte word per field，方便快速序列化和传输
*   Each tuple has three parts: [null bit set] [values] [variable length portion]
*   由原始内存支持的类型取代java对象，spark内部类型，方便序列化反序列化，内存管理
*   每个元组有三部分组成
    -   ［null bit set］用来跟踪字段是否是null，分配了8个字节标记，每隔字段使用一位表示
    -     ［values］每个字段存储一个8字节，保存固定长度字段的类型，对于非固定长度的字段保存offset

##  org.apache.spark.sql.catalyst.trees
#####   TreeNode[BaseType <: TreeNode[BaseType]] extends Product
*   无论是sql语句还是api接口方法都会转换成一个语法树，有了语法树就可以优化，执行等操作。
*   语法树的构建都是由一个个TreeNode组成
*   TreeNode指的是用于构建一般的树的结点
*   通过children: Seq[BaseType]描述了一个一对多的父子关系，child节点同样可以有多个子节点
*   提供一系列查找find,遍历foreach，转换map等方法

###  org.apache.spark.sql.catalyst.plans
####    QueryPlan[PlanType <: QueryPlan[PlanType]] extends TreeNode[PlanType] 
*    def output: Seq[Attribute]
*   专门指一棵执行计划树
*   QueryPlan的泛型参数仅要求为QueryPlan的子类

### org.apache.spark.sql.catalyst.plans.logical
####    LogicalPlan  extends QueryPlan[LogicalPlan]
*   LogicalPlan比起QueryPlan扩展了resolve相关的操作，还加上了一个statistics变量。 该变量实际上就是一个BigInt，代表计划的执行代价
*   Statistics－－执行计划的代价估计。默认叶子节点的代价为1，非叶子节点的代价为各子结点代价的乘积。不同类型的执行计划通过重载其statistics函数来改变代价计算方式
*   方法
    -   resolveOperators(rule: PartialFunction[LogicalPlan, LogicalPlan]): LogicalPlan <== 遍历所有元素应用rule
    -   resolve(schema: StructType, resolver: Resolver): Seq[Attribute] <== 将StructType 解析成 Attribute

#####   LeafNode extends LogicalPlan
*   没有子节点的逻辑计划

#####   UnaryNode extends LogicalPlan
*   只有一个子节点

#####   BinaryNode extends LogicalPlan
*   2个子节点


##  org.apache.spark.sql.catalyst.expressions
#####   Expression extends TreeNode[Expression] 
*   Catalyst的表达式
*   spark sql 语法 或者 api 解析成对应的 experssion，执行时生成相应的java 代码
*   方法
    -   eval(input: InternalRow = null): Any  <== 根据输入的internalRow计算结果
    -   doGenCode(ctx: CodegenContext, ev: ExprCode): ExprCode <== 生成代码

#####   LeafExpression extends Expression
*   没有输入表达式

#####   UnaryExpression extends Expression
*   一个输入，一个输出

#####   BinaryExpression extends Expression
*   两个输入，一个输出



##### Substring(str: Expression, pos: Expression, len: Expression) extends TernaryExpression with ImplicitCastInputTypes
*   string截取操作表达式，生成对应的string截取代码

#####   Attribute extends LeafExpression with NamedExpression with NullIntolerant


#####   <span id = 'AttributeReference'>AttributeReference </span> extends Attribute with Unevaluable
*   在树中由其他操作生成的某个属性的引用
*   属性名称，属性类型（DataType），是否可空，id


##  org.apache.spark.sql.catalyst.expressions.codegen
#####   ExprCode(var code: String, var isNull: String, var value: String)
*   给定InternalRow的表达式执行生成的java代码




##  org.apache.spark.sql.catalyst.analysis
#####   <span id ='Analyzer'>Analyzer</span>(catalog: Catalog,registry: FunctionRegistry,conf: CatalystConf,maxIterations: Int = 100) extends RuleExecutor[LogicalPlan]
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

#####   UnresolvedRelation
*   sql 中解析的表名或者别名，实际不一定存在


##  org.apache.spark.sql.catalyst.encoders
#####   ExpressionEncoder
*   通用jvm对象encoder,java对象和spark sql内部数据结构 internalRow的相互转换
*   字段
    -   schema: StructType <== java类型T对应的结构类型，名称列表+类型列表
    -   serializer: Seq[Expression] <== 一组表达式 ，表示一个类型T的对象 到 internal row 的转换
    -   deserializer: Expression <==  从internalRow转换为对象的表达式
*   方法
    -   toRow(t: T): InternalRow <== 将java对象转换为InternalRow
    -   fromRow(row: InternalRow): T <== 将InternalRow转换为java对象

#####   RowEncoder
*   工厂对象，生成 将外部Row转换成spark sql 内部数据对象DataType的子类

##  org.apache.spark.sql.types
#####   DataType  -- 数据类型
*   spark sql 使用的基本数据类型的基类
*   子类
    -   BooleanType，IntegerType 等


#####   StructType -- 结构类型
*   结构类型，由StructField（名称，类型，可否空 三个字段） 数组组成


##  org.apache.spark.sql.execution
#####   SparkOptimizer extends Optimizer
*   查询规则优化
*   batches
    -   CombineUnions ，Union操作的优化
    -   OptimizeSubqueries，子查询优化
    -   ReplaceIntersectWithSemiJoin，交集操作被替换为左连接
    -   ReplaceDistinctWithAggregate，distinct操作替换为聚合操作
    -   等等

#####   SparkPlan extends QueryPlan[SparkPlan]
*   物理操作的基类
*   操作命名规则是名称+Exec结尾
*   子类：
    -   LeafExecNode extends SparkPlan    无字节点
    -   UnaryExecNode extends SparkPlan    一个子节点
    -   BinaryExecNode extends SparkPlan    两个子节点


#####   <span id = 'SparkPlanner' >SparkPlanner </span>(val sqlContext: SQLContext) extends SparkStrategies
*   将Optimized Logical Plan变为Physical Plan的规则
*   strategies , 一些列转换策略        
    -   FileSourceStrategy，计划如何扫描一组文件
    -   DataSourceStrategy，计划如何扫描数据源，使用数据源的api,如jdbc
    -   BasicOperators,基本操作
        +   Project ，`execution.ProjectExec(projectList, planLater(child)) :: Nil`
        +   Filter,` execution.FilterExec(condition, planLater(child)) :: Nil`


#####   ProjectExec(projectList: Seq[NamedExpression], child: SparkPlan) extends UnaryExecNode with CodegenSupport
*   投影逻辑计划的物理操作
*   投影功能的java代码生成



#####   FilterExec(condition: Expression, child: SparkPlan) extends UnaryExecNode with CodegenSupport  
*   过滤计划的物理操作
*   过滤功能的java代码生成

#####   SparkStrategies extends QueryPlanner[SparkPlan]
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


#####   <span id = 'QueryExecution'>QueryExecution(val sqlContext: SQLContext, val logical: LogicalPlan)</span>
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


#####   DataSourceScanExec extends LeafExecNode 


#####   RowDataSourceScanExec extends DataSourceScanExec with CodegenSupport
*   从一个relation扫描数据的物理计划节点



#####   CodegenSupport extends SparkPlan
*   支持代码生成
*   方法
    -   produce(ctx: CodegenContext, parent: CodegenSupport): String ， 添加注释代码，并调用doProduce生成代码
    -   doProduce(ctx: CodegenContext): String ，生成代码，由子类实现具体方法
    -   consume(ctx: CodegenContext, outputVars: Seq[ExprCode], row: String = null): String ，
    -   doConsume(ctx: CodegenContext, input: Seq[ExprCode], row: ExprCode): String


######  InputAdapter(child: SparkPlan) extends UnaryExecNode with CodegenSupport





#####   <span id = "WholeStageCodegenExec">WholeStageCodegenExec(child: SparkPlan) extends UnaryExecNode with CodegenSupport </span>
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



##  org.apache.spark.sql.execution.joins
#####   HashJoin
*   self: SparkPlan =>   子类必须是sparkPlan的子类
*   spark sql表连接相关的join操作

#####   class BroadcastHashJoin() extends BinaryNode with HashJoin
*   执行两个sparkplan的inner hash join
*   异步spark job用来计算需要broadcast的关系


##  org.apache.spark.sql.execution.datasources
#####   <span id = "DataSource">DataSource</span>
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
    


##  org.apache.spark.sql.execution.datasources.json
#####   JsonFileFormat extends TextBasedFileFormat with DataSourceRegister
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

#####   JdbcRelationProvider extends RelationProvider with DataSourceRegister
*   jdbc数据源 ， 读取jdbc数据以及对应的类型schema
*   方法
    -    createRelation（）: BaseRelation，转换成 BaseRelation

#####   <span id = 'LogicalRelation' >LogicalRelation </span> extends LeafNode with MultiInstanceRelation
*   是一个没有孩子节点的逻辑计划LogicPlan
*   把schema 转成attribute的集合


## org.apache.spark.sql.catalyst.plans.physical
#### Distribution
*   在多台机器并行查询操作时，指定元组tuples如何在分布式中共享共通表达式
*   有两种物理属性：
    -   Inter-node partitioning of data（不同物理机器上的分区数据）：描述元组tuples如何在不同的物理机器集群上分区
        +   一些操作如Aggregate，可以利用这个信息可以为分区执行本地操作，而不是从集群全局操作
    -   Intra-partition ordering of data（内部分区数据）：某个分区的数据可能是分布式的
        +   需要被shuffle的数据可能分布在不同机器上但是属于同一个分区
*   分类
    -   UnspecifiedDistribution，
    -   AllTuples，只有一个partition,并且dataset的数据在同一个地方
    -   ClusteredDistribution，集群中相同值的数据在同一个地方
    -   OrderedDistribution，数据将按照排序表达式排序，
    -   BroadcastDistribution，数据将被广播到所有节点


####   Partitioning
*   描述操作的输出结果如何分布到各个partition上
*   satisfies -- 描述partitionings 与 distributions 的关系
*   compatibleWith  -- 描述 child 输出 partitions之间的关系
*   guarantees -- 描述 child 输出 partition 与 其他 partition的关系
```
*  Diagrammatically:
 *
 *            +--------------+
 *            | Distribution |
 *            +--------------+
 *                    ^
 *                    |
 *               satisfies
 *                    |
 *            +--------------+                  +--------------+
 *            |    Child     |                  |    Target    |
 *       +----| Partitioning |----guarantees--->| Partitioning |
 *       |    +--------------+                  +--------------+
 *       |            ^
 *       |            |
 *       |     compatibleWith
 *       |            |
 *       +------------+

```
*   UnknownPartitioning
*   RoundRobinPartitioning
*   SinglePartition
*   HashPartitioning
*   RangePartitioning
*   BroadcastPartitioning


##  [过去的迭代模型 =》Volcano Iterator Model](https://databricks.com/blog/2016/05/23/apache-spark-as-a-compiler-joining-a-billion-rows-per-second-on-a-laptop.html)
[中文翻译](http://geek.csdn.net/news/detail/77005)


#   主要类层次结构
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
