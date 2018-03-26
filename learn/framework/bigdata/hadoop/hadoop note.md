# hadoop
## block size
*   Block size is just an indication to HDFS how to split up and distribute the files across the cluster - there is not a physically reserved number of blocks in HDFS (you can change the block size for each individual file if you wish)
*   A final note - having lots to small files means that your name node will require more memory to track them (blocks sizes, locations etc), and its generally less efficient to process 128x1MB files than single 128MB file (although that depends on how you're processing it)

### http://blog.csdn.net/clerk0324/article/details/50888016
### https://www.cnblogs.com/Dhouse/p/6901028.html?utm_source=itdadao&utm_medium=referral

##  Hbase vs hive
*   Hbase主要解决实时数据查询问题，Hive主要解决数据处理和计算问题，一般是配合使用。
*   Hbase： Hadoop database 的简称，
    *   也就是基于Hadoop数据库，是一种NoSQL数据库，主要适用于海量明细数据（十亿、百亿）的随机实时查询，如日志明细、交易清单、轨迹行为等。
    *   数据应用从HBase查询数据；
*   Hive:Hive是Hadoop数据仓库 不是数据库
    *   主要是让开发人员能够通过SQL来计算和处理HDFS上的结构化数据，适用于离线的批量数据计算。
    *   HIve清洗处理后的结果，如果是面向海量数据随机查询场景的可存入Hbase   

