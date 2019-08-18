#	hive -- https://cwiki.apache.org/confluence/display/Hive/Home
*   [hive整理总结](http://www.bijishequ.com/detail/566762?p=)
*   http://www.yiibai.com/hive/hive_partitioning.html
*	Hive是建立在Hadoop之上为了减少MapReduce jobs编写工作的批处理系统
*   hive 数据单元
    -   Databases
        +   不同的名称空间，避免表名的冲突
        +   不同组用户给不同的权限
    -   Tables
    -   Partitions
        +   每个表定义的一个或者多个key,用来决定数据如何被存储
        +   通过key可以快速查找
        +   一个分区是一个文件目录
    -   Buckets (or Clusters):
        +   在同一个分区上
        +   同一个分区中，一些列的哈希值放入bucket
        +   避免数据倾斜，桶对于数据抽样再适合不过，同时也有利于高效的map-side Join
*   formats
    -   Text File
        +   按\n分隔的一行行文本
    -   Sequence File
        +    values in binary key-value pairs
        +    merge two or more files into one file
    -   RCFile
        +   offers high row level compression rates
        +    perform multiple rows at a time t
        +    very much similar to the sequence file format. This file format also stores the data as key-value pairs.
    -   AVRO
    -   ORC File
        +    Optimized Row Columnar file format
        +     provides a highly efficient way to store data in Hive table
        +     ORC files improves performance when Hive is reading, writing, and processing data from large tables
    -   Parquet File
        +    a column-oriented binary file format
        +    highly efficient for the types of large-scale queries
*   表设计
	*	分区
*    spark sql & hive 
    -    https://www.cnblogs.com/longjshz/p/5414051.html
*   内部表 & 外部表
    -   CREATE  TABLE  & CREATE EXTERNAL TABLE 
    -   数据来源
        +   外部表可以指定存放路径
    -   删除表
        +   内部表数据+元数据 都删除
        +   外部表只删除元数据
*   hiveservice2  - with cdh 5.9 - /etc/supervisord.d
    -   /usr/lib/hive/bin/beeline
        +   17/12/20 02:57:28 WARN conf.HiveConf: HiveConf of name hive.server2.thrift.client.user does not exist
        +   17/12/20 02:57:28 WARN conf.HiveConf: HiveConf of name hive.server2.thrift.client.password does not exist
    -   !connect jdbc:hive2://localhost:10000
    -   /var/log/hive/hive-server2.log
    -   --hiveconf hive.server2.thrift.client.user=root  --hiveconf hive.server2.thrift.client.password=123456
*   hive事务,https://cwiki.apache.org/confluence/display/Hive/Streaming+Data+Ingest
    -   http://www.csdn.net/article/2014-04-23/2819438-Cloud-Hive
    *   HiveEndPoint
        +   每个事务有一个id
        +   多个事务可以组合成一个Transaction Batch，每个Transaction Batch都写入一个文件，减少文件数量
        +   建立hive连接之后，streaming client首先要请求new batch of transactions，返回为**一部分**transaction batch的Transaction Ids 
        +   客户端在一个transaction写入一个或者多个消息后，提交或者取消当前transaction之后才能处理下个transaction
    *   StreamingConnection
        -   获取各批次的transactions
    *   TransactionBatch
        -   用来写入a series of transactions
        -   每个TransactionBatch对应hdfs一个文件
    *   RecordWriter
        -   接受一个byte数组的数据并转换成支持Hive streamingg的格式并写入
    *   RecordUpdater
    *   Compactor -- 后台压缩处理，
        -   两种压缩类型
            +   minor
                *   takes a set of existing delta files and rewrites them to a single delta file per bucket.
            +   major
                *   takes one or more delta files and the base file for the bucket and rewrites them into a new base file per bucket.  Major compaction is more expensive but is more effective.
*   https://www.slideshare.net/hortonworks/hive-on-spark-is-blazing-fast-or-is-it-final
*   命令
    -   hive --hiveconf hive.root.logger=DEBUG,console  
    -   alter table post_action PARTITION(day='2018-01-04') COMPACT 'major';
        +   只有一个delta时候不执行压缩
    -   SHOW COMPACTIONS;
    -   ALTER TABLE post_distribution DROP IF EXISTS PARTITION (day='2018-01-15');

## hive
### 组件
*   Metastore组件：元数据服务组件，这个组件存储hive的元数据，hive的元数据存储在关系数据库里，hive支持的关系数据库有derby、mysql。
*   Driver组件：核心组件，整个Hive的核心，该组件包括Complier、Optimizer和Executor，它的作用是将我们写的HQL语句进行解析、编译优化，生成执行计划，然后调用底层的MapReduce计算框架。
*   Serializers/Deserializers
*   ThriftMetastore
*   HiveServer

## 性能
*    小表 join 大表 ==》you should structure your join queries so the largest table is last.


## 问题
###  hive 执行 minor compact 然后执行 major compact ，一段时间内会select 出错，Failed with exception java.io.IOException:java.lang.RuntimeException: serious problem 
*   执行一条命令，出现2个压缩任务
*   重新执行一次压缩后，问题解决
*   hive.compactor.worker.threads 多个线程

####  出现两层base目录 /user/hive/warehouse/dyd_raw_data.db/post_distribution/day=2018-01-12/base_0010076/base_0010076
*   重新执行压缩无效，删除最底层base目录

#### master1 ，master2同时执行了compactor任务
*    application_1514973560079_0485 root    uhadoop-adarbt-master2-24-compactor-dyd_raw_data.post_distribution.day=2018-01-17   MAPREDUCE   root.root   Thu Jan 18 00:00:15 +0800 2018  Thu Jan 18 00:00:36 +0800 2018  FINISHED    SUCCEEDED   N/A N/A N/A 
*   application_1514973560079_0484  root    uhadoop-adarbt-master1-24-compactor-dyd_raw_data.post_distribution.day=2018-01-17   MAPREDUCE   root.root   Thu Jan 18 00:00:15 +0800 2018  Thu Jan 18 00:00:34 +0800 2018  FINISHED    SUCCEEDED   
*   master1,master2同时起了 HiveMetaStore
    -   master1 hive.compactor.worker.threads = 0 ，只让master2执行



## hive test
```
CREATE TABLE test_orc5(id INT,dt STRING)
 ROW FORMAT DELIMITED FIELDS TERMINATED BY '\t'
 STORED AS ORC;

CREATE TABLE test_orc3(id INT)
 PARTITIONED BY(dt STRING)
 CLUSTERED BY(id) SORTED BY(id) INTO 32 BUCKETS
 ROW FORMAT DELIMITED FIELDS TERMINATED BY '\t'
 STORED AS ORC;

 INSERT INTO TABLE test_orc PARTITION (dt = '2017-12-01') VALUES (1)
DESCRIBE formatted test_orc partition (dt='2017-05-01');
DESCRIBE formatted post_action partition (day='2018-01-04');


val hiveContext = new org.apache.spark.sql.hive.HiveContext(sc)
hiveContext.sql("select * from test_orc3").show()

// with spark
import  org.apache.spark.sql.SaveMode
import spark.implicits._
val data = Array((1,"2017-12-01"), (2,"2017-12-01"), (3,"2017-12-02"), (4,"2017-12-03"),(5,"2017-12-04"))
val distData = sc.parallelize(data)
val sdf = distData.toDF("id","dt")
sdf.createOrReplaceTempView("temp")
hiveContext.sql("insert into default.test_orc3 partition(dt='2017-12-02') select id  from temp")
sdf.write.mode(SaveMode.Append).insertInto("default.test_orc")
```

