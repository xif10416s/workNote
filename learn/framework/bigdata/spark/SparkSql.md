#Spark Sql
##设计目的：
1.  支持spark Rdd 和外部数据的关系处理
2.  高性能的关系数据操作
3.  易于扩展数据源，如半结构化数据，外部数据
4.  可以被其他高级算法使用如graph,machine learning

##基本流程
![](images/4.png)

#####  file path + format(JSON)
1.  BaseRelation （ 解析json内容，分析schema ）
    -   path+Json+schema
4.  LogicalRelation （生成逻辑计划，将schema 转换成 属性对象）
    -   schema ==> Attribute
1.  QueryExecution ==> analyzed （分析查询计划）
1.  RuleExecutor ==> execute （规则解析并且执行）
1.   =＞Dataset.ofRows   （转换为dataframe , 类型为Row的dateset）
1.  createTemporaryView  add to Catalog （sql解析 ， 将信息保存起来）
    -   sql parser => AST
    -   DataSet API
1.  LogicalPlan ==>unresolved attribute or relations （）
    -   Dataset.ofRows
        -   withCachedData
            -   analyzed LogicalPlan ==> checked ,bind attributes 
            -   optimizedPlan
            -   sparkPlan
            -    executedPlan


##基本组件
##org.apache.spark.sql
#####Datasets
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
#####BaseRelation
*   代表一组已知schema的元组tuples


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


#####AttributeReference  extends Attribute with Unevaluable
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
#####Analyzer(catalog: Catalog,registry: FunctionRegistry,conf: CatalystConf,maxIterations: Int = 100) extends RuleExecutor[LogicalPlan]
*   sql parser出来的logicalPlan只是针对sql的解析，表是否存在，字段是否存在，字段类型都是未知的
*   通过rules 和　Catalogj解析属性
*   解析步骤
    -   Looking up relations by name from the catalog.
    -   Mapping named attributes, such as col , to the input provided　given operator’s children.
    -   Determining which attributes refer to the same value to give
        them a unique ID
*   batches
    -   ResolveRelations
        -   从catalog中检查UnresolvedRelation的表名和别名是否存在
    -   ResolveReferences
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
#####SparkPlan extends QueryPlan[SparkPlan]
*   物理操作的基类
*   操作命名规则是名称+Exec结尾
*   子类：
    -   LeafExecNode extends SparkPlan    无字节点
    -   UnaryExecNode extends SparkPlan    一个子节点
    -   BinaryExecNode extends SparkPlan    两个子节点




#####QueryExecution(val sqlContext: SQLContext, val logical: LogicalPlan)
*   执行关系查询的工作流
*   analyzed
    -   解析－－调用Analyzer的execute方法，执行各种规则，返回解析后logicPlan
*   withCachedData
    -   使用缓存
*   optimizedPlan
    -   执行各种逻辑优化
*   sparkPlan
    -   根据ｌｏｇｉｃＰｌａｎ的类型生成物理计划
    -   LogicalRelation
        -   DataSourceStrategy
            -    DataSourceScanExec
*   executedPlan
    -   在物理计划上扩展
    -   preparations
        -   CollapseCodegenStages
```
if (conf.wholeStageEnabled) {
                WholeStageCodegenExec
                ...
```


#####DataSourceScanExec extends LeafExecNode 


#####RowDataSourceScanExec extends DataSourceScanExec with CodegenSupport
*   从一个relation扫描数据的物理计划节点
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



#####CodegenSupport extends SparkPlan

#####FilterExec(condition: Expression, child: SparkPlan)extends UnaryExecNode with CodegenSupport



#####WholeStageCodegenExec(child: SparkPlan) extends UnaryExecNode with CodegenSupport
*   把计划的所有子树转操作表达式换成java代码封装到一个java方法里
*   


##org.apache.spark.sql.execution.joins
#####HashJoin
*   self: SparkPlan =>   子类必须是sparkPlan的子类
*   spark sql表连接相关的join操作

#####class BroadcastHashJoin() extends BinaryNode with HashJoin
*   执行两个sparkplan的inner hash join
*   异步spark job用来计算需要broadcast的关系


##org.apache.spark.sql.execution.datasources
#####LogicalRelation  extends LeafNode with MultiInstanceRelation
*   把schema 转成attribute的集合


