# **hbase权威**
## 简介
### 表，行，列，单元格
*   行
    -   行序是按照行键的字典序排序的
    -   按照二进制逐字节从左到右依次比对每一行键
    -   与RDBMS主键索引功能相类似
    -   一行由若干列组成，若干列又构成一个列簇
        -   一个列簇的所有列存储在同一个底层文件里，HFile
*   列
    -   qualifer 由任意字节数组组成
    -   没有数量限制，没有类型和长度限制
*   单元格
    -   有时间戳，区分不同版本，按照降序排列，优先读取新的

### 自动分区
*   HBase 负载均衡的基本单元是region
    -   region 本质 是 以行键排序的连续存储区间
    -   初始表只有一个region,当大小超过配置限制时，会分裂成2个region
    -   一个region 只能被一个 region server 加载
    -   一个regionServer 可以加载多个 region

### 底层实现
*   数据存储文件称为Hfile,存储进过排序的键值映射结构
*   文件内部由连续的块组成，块的索引信息存储在文件的尾部
*   当把Hfile打开并加载到内存时，索引信息优先加载到内存，每个块的默认大小64k
*   每个HFile都有一个索引块，首先通过内存中的块索引二分查找，确定包含给定键的块，然后读取磁盘块，找到实际内容
*   数据流：
    -   ==>commit log ==>memstore  ==>flush Hfile ==>清空memstore ==>删除已提交commit log

## 客户端API
### org.apache.hadoop.hbase.client.HTable <== 主要接口，存储，重新，删除操作
*   Htable 创建 代价大，耗时 ， 一个线程一个实例
*   HTablePool ,提供复用多个Htable实例

## 协处理器
*   将一部分计算移动到数据的存放端，在region服务器上运行代码，执行region级别操作
*   权限认证
*   两大类
    -   observer
        +   与触发器类似，当特定事件发生时，执行回调函数
    -   endpoint
        +   类似存储过程，远程计算能力

## 架构
### B+树

### LSM 树

### 概览
*   hbase主要处理两种文件（有HRegionServer管理）： 
    -   预写日志（Write-Ahead log)
    -   数据文件
*   hbase启动时，HMaster将所有的region分配到HRegionServer 上
*   HRegionServer 负责打卡Region,创建HRegion实例
    -   HRegion打开后，为每个表的HColumnFamily创建一个store实例
    -   每个store包含一个或者多个StoreFile实例
    -   StoreFile实例是实际存储文件HFile的封装
    -   每个store有一个对应的MemStore
    -   一个HRegionServer共享一个HLog

### HFile -- 存储hbase 数据
####    逻辑结构 -- https://blog.csdn.net/qq_29493353/article/details/79205068
*   Scanned block section
    -   表示顺序扫描HFile时所有的数据块将会被读取，包括Leaf Index Block和Bloom Block。
*   Non-scanned block section
    -   表示在HFile顺序扫描的时候数据不会被读取，主要包括Meta Block和Intermediate Level Data Index Blocks两部分。
*   Load-on-open-section
    -   这部分数据在HBase的region server启动时，需要加载到内存中。包括FileInfo、Bloom filter block、data block index和meta block index。
*   Trailer
    -   这部分主要记录了HFile的基本信息、各个部分的偏移值和寻址信息

####  物理结构
*   HFlie会被切分为多个大小相等的block块，每个block的大小可以在创建表列簇的时候通过参数blocksize=>’65535‘进行指定，默认为64K
*   大号的Block有利于顺序Scan，小号Block利于随机查询
*   所有block块都拥有相同的数据结构,HBase将block块抽象为一个统一的HFileBlock
*   HFileBlock -- 包括两部分：BlockHeader和BlockData
    -   BlockHeader--存储block元数据
    -   BlockData -- 存储具体数据

    