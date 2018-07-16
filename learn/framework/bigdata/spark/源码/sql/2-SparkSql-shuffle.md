# spark sql shuffle
```
    val df = spark.read.json("examples/src/main/resources/people.json")
    import spark.implicits._
    val qf = df.groupBy("name").count()
    qf.collect()
```

## 基本流程
*   初始化DataSet
*   DataSet#groupBy操作
    -   包装成RelationalGroupedDataset
*   RelationalGroupedDataset#count
    -   新建Dataset,logicalPlan包装成Aggregate
    -   Dataset.ofRows(df.sparkSession, Aggregate(groupingExprs, aliasedAgg, df.logicalPlan))
```
DataSet的queryExecution结构
//逻辑计划
logical:Aggregate
 + child :LogicalRelation
  + relation : HadoopFsRelation

// 物理计划
sparkPlan:HashAggregateExec
 + child : HashAggregateExec
  + child :FileSourceScanExec
    + relation : HadoopFsRelation

// 执行计划
executedPlan:WholeStageCodegenExec
  + child : HashAggregateExec
   + child : InputAdapter
    + child : ShuffleExchange        // 将
     + child : WholeStageCodegenExec
      + child : HashAggregateExec   //在各个partition执行聚合操作
       + child :FileSourceScanExec  //从文件系统读取数据
        + relation : HadoopFsRelation

// RDD
byteArrayRdd:MapPartitionsRDD // 数据编码及压缩
 + prev : MapPartitionsRDD  // shuffle结果 聚合
   + prev : ShuffleRowRDD   // shuffle结果合并
    + dependency: ShuffleDependency
     + _rdd : MapPartitionsRDD 
      + prev : MapPartitionsRDD // partition数据聚合
       + prev :FileScanRDD
```
*   逻辑计划 ==》 物理计划 ，SparkPlanner中的strategies实现
    -   Aggregate 逻辑计划对应有 Aggregation extends Strategy 来负责转换，**每个聚合操作会生成 2个 HashAggregateExec 物理计划** //TODO
        +    创建一个partial aggregations聚合操作，
            *    操作的对象为原始输入数据，当所有输入行处理完成， aggregation buffers结果返回
            *    requiredChildDistributionExpressions = None不依赖其他表达式
        +    创建一个final aggregations聚合操作
            *    操作对象为aggregation buffers中间结果，合并这些中间结果返回
*   物理计划 ==》 执行计划，通过preparations定义的一些rule做一些增强处理
    -   CollapseCodegenStages，负责全局代码生成WholeStageCodegenExec与InputAdapter物理计划插入
    -   EnsureRequirements,通过插入ShuffleExchange操作来保证，org.apache.spark.sql.catalyst.plans.physical.Partitioning的数据满足需要的org.apache.spark.sql.catalyst.plans.physical.Distribution 
        +   **ShuffleExchange 在此处理是被添加** //TODO
            *   生成对应的ShuffleDependency
            *   返回一个ShuffledRowRDD
