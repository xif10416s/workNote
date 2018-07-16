#   基本流程 -- spark 2.3.1
## 新定义接口 -- 中间使用了一些过度接口为了兼容老版本如：BaseStreamingSource
*   DataSource为一个类， 定义了可插拔的数据源，对应一些列旧的数据源
*   DataSourceV2 spark2.3.1新接口，只是一个接口，没有任何方法，需要配合ReadSupport或者WriteSupport接口等一起
    +   MicroBatchReadSupport -- 实现创建MicroBatchReader
        *   RateSourceProviderV2
    +   ContinuousReadSupport -- 低延迟流处理支持接口
*   DataSourceReader -- spark2.3.1接口，优化读取新能   
    *   MicroBatchReader -- 微批处理模式，从数据微批量读取数据接口
        -   子类
            -   RateStreamMicroBatchReader
        -   主要方法
            +   setOffsetRange -- 设置数据的开始结束位置，用于根据范围生成一批次数据集合
            +   commit -- 当offsetRange数据集合处理完成时调用，标记当前批次数据已经处理完成，下一次从endOffset开始新的一批数据
    *   ContinuousReader -- Continuous 处理模式
        -   KafkaContinuousReader
        -   RateStreamContinuousReader
*   DataSourceWriter --  优化写入性能
    -   SupportsWriteInternalRow -- 可以直接使用内部数据类型InternalRow直接写入，避免InternalRow到Row的转换（不稳定实验接口）
*   StreamWriteSupport--支持structured streaming写入


##  StructuredNetworkWordCount 统计来自socket的word count
###   加载数据流为 DataFrame， DataStreamReader#load
*   加入实现新接口的类的数据源特殊处理
    -    MicroBatchReadSupport接口实现类调用createMicroBatchReader创建reader,使用StreamingRelationV2包装成Dataset 
    -    ContinuousReadSupport接口实现类调用createContinuousReader创建reader,使用StreamingRelationV2包装成Dataset 

### 数据流查询启动streamingQueryManager.createQuery  
*   2.3 StreamExecution由具体类改成了抽象类，多个两个子类，ContinuousExecution，MicroBatchExecution
*   2.2 直接使用StreamExecution执行查询操作改为匹配ContinuousTrigger的情况使用ContinuousExecution，否则使用MicroBatchExecution
*   StreamExecution -- 启动线程负责执行逻辑
    -   启动QueryExecutionThread线程，负责run方法的主体结构，容错处理，实际具体处理有子类runActivatedStream实现
*   MicroBatchExecution -- 微批处理执行器
    -   triggerExecutor触发器 -- 控制执行逻辑的处理间隔
        +   ProcessingTimeExecutor -- while(ture)循环处理
            *   默认是Trigger.ProcessingTime(0L)，每个 micro-batch 处理完成后开启一个新的
        +   OneTimeExecutor -- 只调用一次处理

### dataset执行触发变化
*   2.2 通过 sink.addBatch 触发Dataset action任务
*   2.3 通过 在newAttributePlan上再包装一次生成，triggerLogicalPlan，直接掉用dataSet的collect方法

### 最终Dataset#queryExecution : IncrementalExecution结构
```
== Parsed Logical Plan ==
WriteToDataSourceV2 org.apache.spark.sql.execution.streaming.sources.MicroBatchWriter@a968e33  <== 就是ConsoleWriter
+- Aggregate [value#8], [value#8, count(1) AS count#11L] <==groupBy("value").count() 操作生成的计划
   +- SerializeFromObject [staticinvoke(class org.apache.spark.unsafe.types.UTF8String, StringType, fromString, input[0, java.lang.String, true], true, false) AS value#8]
      +- MapPartitions <function1>, obj#7: java.lang.String <== .flatMap(_.split(" ")) 操作
         +- DeserializeToObject cast(value#48 as string).toString, obj#6: java.lang.String
            +- LogicalRDD [value#48], true

== Physical Plan ==
WriteToDataSourceV2 org.apache.spark.sql.execution.streaming.sources.MicroBatchWriter@a968e33
+- *(4) HashAggregate(keys=[value#8], functions=[count(1)], output=[value#8, count#11L])
   +- StateStoreSave [value#8], state info [ checkpoint = file:/private/var/folders/vd/txsj8y2d227c538pykv3k0nr0000gn/T/temporary-153badd6-2713-43b1-ad6f-de081199eed3/state, runId = a942260c-5ba0-4c13-86b1-e688ec41dfc3, opId = 0, ver = 2, numPartitions = 200], Update, 0
      +- *(3) HashAggregate(keys=[value#8], functions=[merge_count(1)], output=[value#8, count#19L])
         +- StateStoreRestore [value#8], state info [ checkpoint = file:/private/var/folders/vd/txsj8y2d227c538pykv3k0nr0000gn/T/temporary-153badd6-2713-43b1-ad6f-de081199eed3/state, runId = a942260c-5ba0-4c13-86b1-e688ec41dfc3, opId = 0, ver = 2, numPartitions = 200]
            +- *(2) HashAggregate(keys=[value#8], functions=[merge_count(1)], output=[value#8, count#19L])
               +- Exchange hashpartitioning(value#8, 200)
                  +- *(1) HashAggregate(keys=[value#8], functions=[partial_count(1)], output=[value#8, count#19L])
                     +- *(1) SerializeFromObject [staticinvoke(class org.apache.spark.unsafe.types.UTF8String, StringType, fromString, input[0, java.lang.String, true], true, false) AS value#8]
                        +- MapPartitions <function1>, obj#7: java.lang.String
                           +- DeserializeToObject value#48.toString, obj#6: java.lang.String
                              +- Scan ExistingRDD[value#48]
```


