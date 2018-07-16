# hbase 实战
## 一，介绍
### 使用场景
####  互联网搜索 -- BigTable发明原因
*   定位关系的信息--页码，主题，

####  抓取增量数据
*   抓取监控指标：OpenTSDB
*   抓取用户交互数据
    -   使用hbase计数器记录网页访问次数
*   遥测技术--软件运行数据收集和分析（Mozilla）
*   广告效果与点击流 -- 用户交互数据捕获与分析

#### 内容服务

## 二，入门
###  hbase写路径
*   hbase接受到命令后，存下变化信息，执行写入
*   写入操作涉及两个地方(当两个地方都写入并确认后才完成写入动作)：
    +   write-ahead log （HLog)
    +   MemStore
        *   内存写入缓冲区，写满后会刷新到硬盘，正常HFile
*   HFile - hbase使用的底层存储格式
    -   HFile对应CloumnFamily
        +   CloumnFamily  ==1 to n ==> Hfile 
        +   CloumnFamily  ==1 to 1 ==> MemStore
*   WAL - 集群中每台机器维护一个WAL记录发生的变化
    -   底层是一个文件系统（HDFS)
    -   新的变化信息写入WAL成功之后才认为写入成功，用于失败恢复,保证数据不丢失
    -   一台服务器有一个WAL,所有表共享WAL

### hbase读路径
*   读取的操作需要结合硬盘的HFile 和 内存的 MemStore
*   BlockCache -- 缓存技术 LRU(最近最少使用)
    -   保存从HFile读入内存的频繁访问的数据，避免读取硬盘
    -   每个ColumnFamily 有自己的 BlockCache
    -   block -- hbase 从硬盘完成一次读取的数据单位，默认64k
        +   首先从索引查找到block 之后从 硬盘读取
        +   是建立索引的最小数据单位
        +   是从硬盘读取的最小单位
        +   随机查询的场景，需要细粒度的block索引，但是block变小，索引会变大，内存消耗增加
        +   顺序扫描的场景，使用大一点的block
*   hbase读取一行 
    -   1. 检查MemStore等待修改的队列
    -   2. 检查BlockCache缓存
    -   3. 最后访问HFile

### 合并工作
*   删除操作
    -   在记录上添加标记，标记的内容不出现在get和scan 结果中
*   当执行Major compaction后，标记的内容才会被删除，并释放存储
*   压缩操作 -- 
    *   Major compaction
        -   处理给定cloumnFamily中的所有Hfile,改CF中所有HFile合并成一个Hfile
        -   Major compaction 之后才能完全清理删除的数据
        -   耗费资源
    *   minor compaction 
        -   将多个小的Hfile 合并成 一个大的Hfile
            +   hbase 读取一行可能引用多个hfile, 读取文件数量影响读取性能
            +   合并操作将多个文件的一行数据合并到一个新的文件中，减少读取改行需要引用的文件数
            +   标记新文件为激活状态，删除原来的小文件

### 数据模型
*   表（table) -- Hbase用来组织数据，
    *   字符串组成，并且可以作为文件路径名称
*   行（row) -- 在表里，按行存储数据
    -   行有行键（rowKey）唯一表示
    -   rowkey没有数据类型，总是视为byte[]
*   列簇（column family) -- 行里面的数据按 CF 分组，影响hbase 数据的物理存储
    -   建表时候预先定义，不可修改
    -   字符串组成，并且可以作为文件路径名称
*   列限定名（column qualifier)-- 用来定位列
    -   不需要预定义，不同行的列不需要相同
*   单元（cell) -- rowkey + CF + cq 唯一确定一个cell
    -   存储在cell的数据成为value
*   时间版本（version) -- 单元值有时间版本，是一个long
    -    不指定时，使用当前时间

### 数据模型
*   没有关系约束，不支持多行事务

### 逻辑模型：有序映射的映射集合
*   结构是个map
*   有序：rowkey 有序， CF有序， cq 有序，时间版本有序(降序)
*   Map<RowKey,Map<ColumnFamily,Map<ColumnQualifer,Map<Vesrsion,value>>>>

### 物理模型：面向ColumnFamily
*   ColumnFamily在硬盘上有自己的Hfile集合，每个CF的Hifle都是独立管理
*   hbase的记录安装key-value存储在Hfile里，HFIle为二进制文件，不可直接读取
*   一行中的一个CF的数据不一定存放在同一个Hfile里面


### 原子操作
*   列值递增 Increment Column values : table.incrementColumnValue(...)
    -   不用先读取值就可以改变值，hbase的incrementColumnValue == java的AtomicLong.addAndGet()

### ACID语义
*   Atomicity(原子性) -- 不可分割的操作，全成功 or 全失败
*   Consistency(一致性) -- 如果事务是并发多个，系统也必须如同串行事务一样操作。
*   Isolation (隔离性) -- 如果有两个事务，运行在相同的时间内，执行相同的功能，事务的隔离性将确保每一事务在系统中认为只有该事务在使用系统。这种属性有时称为串行化，为了防止事务操作间的混淆，必须串行化或序列化请求，使得在同一时间仅有一个请求用于同一数据。
*   Durability(持久性) -- 在事务完成以后，该事务对数据库所作的更改便持久的保存在数据库之中，并不会被回滚。
    
##  分布式HBase ,HDFS,MapReduce
### 并行计算
*   客户端计算
    *   Split[] splits = split(startRow,endRow,numSplits) // 根据rowkey分区并行计算
    *   配合线程池计算汇总
*   MapReduce分布式计算
    -   不需要手动分区，不需要线程池定义和清理
    -   计算任务分成：map 和 reduce 任务
    -   每个任务只处理一部分数据
    -   根据输入和输出数据定义任务
    -   任务以来自己的输入数据，与其他任务隔离

### 切分和分配大表
*   region -- 数据单位
    -   按照RowKey划分的数据区块
    -   hbase特殊表(-ROOT- 和 .META.)：
        +   用来查找表的region位置
        +   -ROOT-不会切分，.META.按需切分成多个region 
*   RegionServer 托管region,与DataNode部署在一起
    -   一个RegionServer托管多个region

#### -ROOT- 和 .META. 结构 与 使用
*   开始查找一行T1表的00002 ==》查找 —ROOT-表 （表名+ rowkey）确定 META信息在哪里 = RS2 的M1 ==》从RS2上读取M1信息 （表名+rowkey）确定 Region位置= RS1 的 R1 ==>从RS1的R1上读取T1表的00002数据
*   -ROOT-.表位置
    -   通过zookeeper获取地址
```
// Region表
T1-R1 (T1表的R1 region)
Rowkey,name, phone
00001,john, 131.....
00002,john1, 131.....
00003,john2, 131.....
00004,john3, 131.....

// .META. 表
.META.-M1 (META表的M1分区)
表名：开始rowkey - 表名：结束rowkey , region分区号， regionServer
T1:00001-T1:00004 , R1 ,RS1   <== T1 表的 rowkey在00001-00004范围的数据在RegionServer 1托管的 R1 Region上
T1:00005-T1:00008 , R2 ,RS2

// -ROOT-表
M:表名开始rowkey-M:表名结束rowkey , META表的分区号， Regionserver
M:T100001-M:T10009 , M1 , RS2  <== T1表的rowkey在00001-00009范围的数据信息存储在META表的M1分区上，托管在RS2

```


## CF高级配置
### 数据块缓存
*   不需要缓存的场景，关闭缓存，BLOCKCACHE= false
*   alter 'dyd:user_recommend_posts',NAME => 'f', BLOCKCACHE => 'false'

## 使用协处理器扩展hbase
*   将计算逻辑推到托管hbase 的节点上
*   hbase 分布式存储 扩展成 分布式处理系统

### 两种协处理器
*   observer 协处理器
    -   理解成关系数据库的触发器
    -   RegionObserver -- 数据访问和操作阶段添加处理
    -   WALObserver -- 预写日志
    -   MasterObserver -- DDL事件拦截处理
*   endpoint 协处理器 -- 通用扩展器 --类似 关系数据的 存储过程
    -   用来实现分散和聚合算法

## Hbase 客户端
###  hbase rest服务
*   启动 -- hbase rest start -p 9999

### 通过python 使用hbase thrift
*   thrift 客户端 通过 单台thrift 服务接口 和 hbase 集群 通信，吞吐量有限制


## 应用系统实例
### openTSDB
*   基于hbase构建的分布式的，可扩展的时间序列数据库
*   存储，索引，服务从大规模计算机系统（网络设备，操作系统，应用程序）采集来的监控指标数据，并使这些数据易于访问和可视化

#### 主要的两张hbase 表 tsdb and tsdb-uid
##### tsdb-uid 
*   对于一条数据来说，应该至少含有一个指标和一个标签，这样的数据才是有意义的，因此，在OpenTSDB的表设计上，就把“指标”（metrics）和“标签”（Tag）统一放在了tsdb-uid表中存储
*   表结构：
    -   CF:name    <== 将一个UID映射到一个字符串
        +   metrics
        +   tagk
        +   tagv
    -   CF:id     <== 将字符串映射到UID (反向关联关系)
        +   metrics
        +   tagk
        +   tagv
*   字段含义
    -   metrics - 诸如sys.cpu.0或trades.per.second之类的度量标准
    -   tagk - 标签名称，例如主机或符号。 这总是标签键/值对中的“键”（第一个值）。
    -   tagv - 标签值，如web01或goog。 这总是标签键/值对中的“值”（第二个值）。
*   UID
    -   在OpenTSDB中，每一个metric、tagk或者tagv在创建的时候被分配一个唯一标识叫做UID，他们组合在一起可以创建一个序列的UID或者TSUID。在OpenTSDB的存储中，对于每一个metric、tagk或者tagv都存在从0开始的计数器，每来一个新的metric、tagk或者tagv，对应的计数器就会加1。当data point写到TSD时，UID是自动分配的。你也可以手动分配UID，前提是auto metric被设置为true。默认地，UID被编码为3Bytes，每一种UID类型最多可以有16,777,215个UID
    -   使用UID可以帮助节省空间

##### tsdb
*   TSUID： 当一个data point被写到OpenTSDB时，其row key格式为：<metric_UID><timestamp><tagk1_UID><tagv1_UID>[...<tagkN_UID><tagvN_UID>]，不考虑时间戳的话，将其余部分都转换为UID，然后拼在一起，就可以组成为TSUID。
*   表结构
    -   CF:t
        +   低序时间戳
*  rowkey
    -   指标UID(指标+标签的某个组合）+ 数据生成时间（取整点时间）+标签1-Key的UID+标签1-Vlaue的UID+...+标签N-Key的UID+标签N-Vlaue的UID

#### 设计
#####   针对Hot Spot的应对策略
*   业务字段“metrics"来替代了“哈希”字段为rowkey的开始

#####   rowkey的设计思想
*   一.为了能够检索特定的metrics,tag name,tag name的data point, 将 metrics,tag name,tag name编入rowkey是显然的事情,但是直接使用它们来组成rowkey有两个明显的问题:
    -   会占用大量的存储空间(因为这些值会大量重复地出现在很多的rowkey中)
    -   由于每一个metrics,tag key,tag value的长度都是不固定的,这不利于通过字节偏移量来直接定位它们.(否则需要使用特定的分隔符,而且为了避免输入信息中可能存在特定的分隔符导致解析出错,还要对所有输入信息的分割符进行转义处理)
    *   为这些标签和标签值分配一个定长的ID,在rowkey中使用它们的ID来指代它们,这样rowkey就可以规范化,方便从rowkey中直接通过偏移截取需要的"部分".
*   二.Tall-Narrow和Wide-Flat两种表设计风格相结合

