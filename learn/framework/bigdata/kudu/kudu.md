# kudu
*	https://www.jianshu.com/p/d91761c63a45

## 基本概念
*   Kudu是Cloudera开源的新型列式存储系统，专门为了对快速变化的数据进行快速的分析
*   Kudu是对HDFS和HBase功能上的补充，能提供快速的分析和实时计算能力，并且充分利用CPU和I/O资源，支持数据原地修改，支持简单的、可扩展的数据模型。
*   Kudu是基于全新的设计，因此可以更充分地利用RAM、I/O资源，并优化CPU利用率。
    -   Kudu相比与以往的系统，CPU使用降低了，I/O的使用提高了，RAM的利用更充分了。
*   解决问题：
    -  对数据扫描(scan)和随机访问(random access)同时具有高性能，简化用户复杂的混合架构
    -  高CPU效率，使用户购买的先进处理器的的花费得到最大回报
    -  高IO性能，充分利用先进存储介质
    -  支持数据的原地更新，避免额外的数据处理、数据移动
    -  支持跨数据中心replication
*   它定位OLAP和少量的OLTP工作流，如果有大量的random accesses，官方建议还是使用HBase最为合适
*   2016年7月25日正式宣布毕业

## 基本架构
*   Kudu采用了类似log-structured存储系统的方式，增删改操作都放在内存中的buffer，然后才merge到持久化的列式存储中。Kudu还是用了WALs来对内存中的buffer进行灾备。
*   Kudu没有借助于HDFS存储实际数据，而是自己直接在本地磁盘上管理分片数据，包括数据的Replication机制，kudu的Tablet server直接管理Master分片和Slave分片，自己通过raft协议解决一致性问题等，多个Slave可以同时提供数据读取服务
*   Kudu的分区数必须预先指定（对Range的分区策略也有这个要求，估计是先简单化统一处理），不支持动态分区分裂，合并等，因此表的分区一开始就需要根据负载和容量预先进行合理规划。
*   在Kudu中，对于Flush到磁盘上的DiskRowSet（DRS）数据，实际上是分两种形式存在的，一种是Base的数据，按列式存储格式存在，一旦生成，就不再修改，另一种是Delta文件，存储Base数据中有变更的数据，一个Base文件可以对应多个Delta文件，这种方式意味着，插入数据时相比HBase，需要额外走一次检索流程来判定对应主键的数据是否已经存在，Kudu是牺牲了写性能来换取读取性能的提升
