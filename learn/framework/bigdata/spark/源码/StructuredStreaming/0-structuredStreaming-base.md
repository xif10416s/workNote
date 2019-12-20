#   Structured Streaming
*   在spark sql 引擎基础上建立的可扩展，容错的流处理引擎。
*   可以以处理一小批静态数据的方式处理流计算
*   当流数据到来时，spark sql 引擎会连续的增量的更新最终结果，spark sql 的所有优化都用使用到
*   通过checkpointing and Write Ahead Logs 保证端到端的 exactly-once 容错处理
*   fast, scalable, fault-tolerant, end-to-end exactly-once
*   spark 2.3 以前 使用 micro-batch processing engine，端对端的延迟在100 ms,端对段只有一次语义（exactly-once)
*   spark 2.3 以后 使用 Continuous Processing ，延迟在 1ms 端对的最少一次语义（at-least-once)

## 程序模型
*   最基本的设计是将实时数据流看成一张不断添加数据的无边界的表，与批处理方式相似
    -   新数据添加= 在无边界的表中添加行
    *   ![](https://spark.apache.org/docs/latest/img/structured-streaming-stream-as-a-table.png)

## 基本处理
*   ![](https://spark.apache.org/docs/latest/img/structured-streaming-model.png)
*   每个一定时间【1s】会触发一次数据操作，新的数据会作为行添加到input Table
*   然后触发 input 上的查询操作生成 结果result table
*   Output Modes ： 结果内容最终可以以不同模式被输出到外部存储
    -   Complete Mode ：
        +   全部更新后的result table被写入到外部存储，有外部存储接收器负责决定如何处理整个结果表
            *   aggregation queries
    -   Append Mode
        +   只处理新添加到result table的数据，适用于数据不会被更新的场景，只要处理增量就可以
            *   select, where, map, flatMap, filter, join
    -   Update Mode -Available since Spark 2.1.1
        +   只有相对于上一次变化的结果会被处理
*   Output Sinks -- 保存目的源
    -   File sink  ， 保存到目录
    -   Kafka sink ， 保存到kafka
    -   Foreach sink , 遍历处理，自己实现保存逻辑
    -   Console sink，测试用，打印输出
    -   Memory sink，测试用
*   Triggers -- 批处理任务触发时机定义
    -   unspecified（默认）
        +   不指定的情况， micro-batch 模式，每个 micro-batch 处理完成后开启一个新的 micro-batch 
    -   Fixed interval micro-batches -- 指定间隔的 micro-batches
        +   处理时间快的情况，在指定间隔内完成，等待到间隔时间开启新的micro-batches
        +   处理时间慢的情况，超过指定间隔时间了，处理完成立即开启新的micro-batches
        +   没有新数据的情况不会开启micro-batches
    -   One-time micro-batch
        +   只执行一次micro-batch，之后就停止
    -   Continuous with fixed checkpoint interval -- 低延迟持续处理（2.3.1 实验）
        +   1 ms 端对端延迟，最少一次容错语义 ，相比微批处理 100ms延迟，exactly-once容错处理
*   每一种模式都有各自适用的查询操作

##  参数
*   spark.sql.streaming.minBatchesToRetain 
    -   保存最多批次数，用于恢复，默认100
    -   超出的批次数据会被删除
    -   影响对象
        +   StreamExecution#offsetLog -- write-ahead-log，未处理批次数据 
        +   StreamExecution#commitLog -- 完成批次记录

##  基于事件发生时间的窗口处理
*   时间点说明：
    -   事件发生时间Event-time，如数据产生的时间点
    -   数据接收时间点 receive-time, spark 流处理接收到数据的时间点
*   场景及问题点：
    -   需要知道每5分钟用户的点击量
    -   之前的批处理方式的spark streaming,按照批处理数据的时间点为单位，也就是spark数据接收的时间点处理
        +   ![](https://spark.apache.org/docs/latest/img/streaming-dstream-window.png)
    -   当由于网络原因，事件发生时间和数据接收时间差异比较大的时候，如：10:01分发生数据在10:08分才收到，那么按照原来spark streaming在10:01-10:05这个批处理就统计不到这条数据
    -   ![](https://spark.apache.org/docs/latest/img/structured-streaming-late-data.png)
        +   spark2.1 开始可以使用watermarking来处理数据在一定时间范围内的延后数据做聚合操作
            *   操作代码，withWatermark("timestamp", "10 minutes")十分钟之内的延时收到的数据都能处理
*   Structured Streaming可以按照事件发生时间Event-time为窗口处理，相当于在一个表里面统计发生时间这列的数据
*   对于超时特别久的数据可以做特殊处理
*   withWatermark支持延迟数据处理
    -   Append mode
        +   会保持中间状态，直到超过watermark之后，把最终结果写入Result Table，删除中间状态，所以结果出来有个延迟时间
    -   Update mode
        +   每次处理都写入Result Table，在watermark之内的数据会更新旧的数据，在watermark之外的数据丢弃
    -   Complete mode
        +   保存所有结果  

##  end-to-end exactly-once 容错
*   sources -- offsets 跟踪读取的位置
    -   File source
        +   将某个目录的文件作为数据流，支持 text, csv, json, parquet
    -   Kafka source
        +   从kafka拉取数据
    -   Socket source
        +   测试用，从socket连接中读取文本
*   sinks -- 幂等操作
    -   File sink，存储到文件目录
    -   Console sink,到控制台，测试用
    -   Memory sink，到内存，测试用
*   execution engine -- checkpointing and write ahead logs to record the offset range of the data being processed in each trigger


## 问题
*   保存数据量，complete模式全量数据保存？update模式保存多少数据

