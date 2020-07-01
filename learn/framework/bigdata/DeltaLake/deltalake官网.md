####  原文

* https://docs.databricks.com/delta/index.html

###  data lake 数据湖

#### data lake : 什么是数据湖

* **集中式的数据仓库**，即存储传统的结构化数据（有schema的），也存储非结构化的原始数据（视频，图片，二进制文件等）
* 数据湖利用廉价的对象存储和**开放格式**来使许多应用程序能够利用数据
* 让企业所有数据（业务系统数据，文件数据，图片视频数据等）以原有的形式统一的集中的存储，不需要额外的结构化处理后存储



####  data lake产生原因

* 企业的数据孤立在各个不同的系统中：数据仓库，数据库，以及其他存储系统
* 为了能通过机器学习和数据分析等方式充分挖掘数据价值，首先需要整合所有数据



##### data lake 数据湖的功能特点：

* 高效的数据分析和机器学习    -- **保留原始数据，可以快速转换**
  * 可以快速的将原始数据转换为可分析的结构化数据，提供给数据分析，机器学习使用
  * 原始数据可以低成本永久保留，以便后续分析使用
* 集中的，合并的，目录化所有数据 -- **集中管理，统一目录**
  * 集中式数据湖消除了数据孤岛的问题（例如数据重复，多种安全策略以及协作困难）
  * 为下游用户提供一个查找所有数据源的地方
* 快速，无缝地集成各种数据源和格式 -- **集成数据源，数据格式**
  * 所有数据类型都可以无限期地收集和保留在数据湖中
* 通过工具实现自主取数据分析 -- 自定义数据格式
  * 数据湖非常灵活，使具有完全不同技能，工具和语言的用户能够一次执行不同的分析任务



##### Data lake（数据湖） Vs  data warhourse 数据仓库

|              | Data lake（数据湖）                                  | data warhourse 数据仓库                  |
| ------------ | ---------------------------------------------------- | ---------------------------------------- |
| 支持数据类型 | 结构化，半结构化，非结构化                           | 结构化                                   |
| 成本         | 低                                                   | 高                                       |
| 扩展性--？   | 任何类型，任何量级的数据都可以低成本的可扩展的保存   | 供应商成本增加，扩展成本大量增加         |
| 使用者       | data analysts  ,  data  scientists                   | data  analysts                           |
| 优点         | 低成本，可扩展，灵活，运行存储原始数据，方便机器学习 | 与传统关系数据库风格一致                 |
| 缺点         | 没有工具来组织和分类数据，探索大量原始数据可能很困难 | 高成本，专有软件，不支持非结构化原始数据 |



### delta lake 

#### delta lake：实现数据湖的可靠的存储层（数据湖的一个实现方案）

* 在spark之上构建的事务ACID支持：序列化级别的隔离，确保不会读取不一致的数据
* 可扩展的元数据处理:  利用Spark的分布式处理能力来处理PB级表的所有元数据，轻松处理数十亿个文件
* 统一流处理和批处理：
  * Delta Lake中的一个表是一个批处理表同同时也可以是一个流的source和sink
  * 流数据提取，批处理数据历史回填，交互式查询，开箱即用
* schema增强：自动处理schema变化，防止在提取数据时插入坏的数据
* 时间旅行：数据支持版本的回滚，全部历史审计追踪，机器学习可以指定不同的版本重放试验
* 更新与删除： 支持合并，更新，删除操作



####   Delta Lake 简单操作

#####  建表 -- create a table

* 都是基于spark api， 区别在于 存储格式 由之前使用的 parquet, csv ,json等改成 delta

  ```
  // 案例一： 支持从json 加载的dataframe 转换为 delta 表
  events = spark.read.json("/databricks-datasets/structured-streaming/events/")
  events.write.format("delta").save("/mnt/delta/events")
  spark.sql("CREATE TABLE events USING DELTA LOCATION '/mnt/delta/events/'")
  
  
  // 案例二：通过structured streaming写入dalta 表， delta lake事务日志支持exactly-once处理
  eventsDF = ( // 流读取
    spark
      .readStream
      .schema(jsonSchema) # Set the schema of the JSON data
      .option("maxFilesPerTrigger", 1) # Treat a sequence of files as a stream by picking one file at a time
      .json(inputPath)
  )
  
  (eventsDF.writeStream // 流写入
    .outputMode("append")
    .option("checkpointLocation", "/mnt/delta/events/_checkpoints/etl-from-json")
    .table("events")
  )
  
  // 批量更新或插入
  MERGE INTO events
  USING updates
  ON events.eventId = updates.eventId
  WHEN MATCHED THEN  //  匹配更新
    UPDATE SET
      events.data = updates.data
  WHEN NOT MATCHED  // 不匹配，插入
    THEN INSERT (date, eventId, data) VALUES (date, eventId, data)
  
  ```

* 读取操作：

  * 支持读取版本，或者时间戳

  ```
df1 = spark.read.format("delta").option("timestampAsOf", timestamp_string).load("/mnt/delta/events")
df2 = spark.read.format("delta").option("versionAsOf", version).load("/mnt/delta/events")
  ```



#####  优化表

* 多次执行表的变更处理后，会形成很多小文件，可以通过命令合并小文件，提高读取性能

  * ```
    OPTIMIZE delta.`/mnt/delta/events`
    OPTIMIZE events
    ```

#####  z-order 列 -- Databricks 

* 为了进一步提高读取性能，通过Z-Ordering在同一组文件中同时定位相关信息
* Delta Lake数据跳过算法会自动使用这种共处位置，从而大大减少了需要读取的数据量

```
OPTIMIZE events
  ZORDER BY (eventType)
```