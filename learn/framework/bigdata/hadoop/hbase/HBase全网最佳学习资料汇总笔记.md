#   HBase全网最佳学习资料汇总
*   https://yq.aliyun.com/articles/169085?spm=5176.100239.blogrightarea176102.19.pH0StL

##  学术界关于HBase在物联网/车联网/互联网/金融/高能物理等八大场景的理论研究
*   https://yq.aliyun.com/articles/214619?spm=a2c4e.11153940.blogcont169085.21.c346c364tIxX2d

### 基于HBase的海量GIS数据分布式处理实践
*   栅格数据
*   经纬度转换geo hash
*   row key 设计：表名；空间数据类型；经纬度geohash
*   使得在按空间位置检索矢量地理信息时,能通过HBase的rowkey迅速定位需要返回的数据

### 基于 HBase的分布式空间数据库技术
*   针对在大型地理信息系统（GIS）中,需要对海量矢量数据和栅格数据进行存储并对高并发的用户查询请求提供高效响应,传统的设计方案难以满足需求的问题,提出一种使用基于内存存储的分布式数据库HBase存储空间数据,并设计基于GeoHash的分布式空间索引,实现了矢量空间数据与栅格空间数据的分布式存储与快速查询

### 基于HBase的全天候全域出租车聚集实时监测方法

### 基于HBase的车联网传感数据管理系统设计


## hbase应用场景
*   对象存储：我们知道不少的头条类、新闻类的的新闻、网页、图片存储在HBase之中，一些病毒公司的病毒库也是存储在HBase之中
*   时序数据：HBase之上有OpenTSDB模块，可以满足时序类场景的需求
*   推荐画像：特别是用户的画像，是一个比较大的稀疏矩阵，蚂蚁的风控就是构建在HBase之上
*   时空数据：主要是轨迹、气象网格之类，滴滴打车的轨迹数据主要存在HBase之中，另外在技术所有大一点的数据量的车联网企业，数据都是存在HBase之中
*   CubeDB OLAP：Kylin一个cube分析工具，底层的数据就是存储在HBase之中，不少客户自己基于离线计算构建cube存储在hbase之中，满足在线报表查询的需求
消息/订单：在电信领域、银行领域，不少的订单查询底层的存储，另外不少通信、消息同步的应用构建在HBase之上
*   Feeds流：典型的应用就是xx朋友圈类似的应用
*   NewSQL：之上有Phoenix的插件，可以满足二级索引、SQL的需求，对接传统数据需要SQL非事务的需求

## hbase之上的应用
依然还需要基于HBase之上构建一些专业的能力，如：
*   OpenTSDB 时序数据存储，提供基于Metrics+时间+标签的一些组合维度查询与聚合能力
*   GeoMesa 时空数据存储，提供基于时间+空间范围的索引能力
    -   https://www.geomesa.org/documentation/tutorials/geomesa-quickstart-hbase.html
    -   GeoMesa 是由locationtech开源的一套地理大数据处理工具套件。其可在分布式计算系统上进行大规模的地理空间查询和分析。使用GeoMesa开源帮助用户管理、使用来自于物联网、社交媒体、手机应用的海量的时空（spatio-temporal）数据。
*   JanusGraph 图数据存储，提供基于属性、关系的图索引能力
    -   https://blog.csdn.net/gobitan/article/details/80939224

##  Accordion:HBase一种内存压缩算法
*   减少flush到HDFS的频率，从而降低了写入放大和磁盘占用。 随着flush次数的减少，MemStore写入磁盘的频率会降低，进而提高HBase写入性能。
*   basic 级别
    -   有数据更新的优化
    -   basic压缩不会消除冗余数据版本以避免物理复制，它只是重新排列KeyValue对象的引用
*   eager 级别
    -   高数据流的应用非常有用，如生产-消费队列，购物车，共享计数器等。
    -   eager 级压缩优化可能会增加计算开销（更多内存副本和垃圾收集），这可能会影响数据写入的响应时间。如果MemStore使用堆内MemStore-本地分配缓冲区（MSLAB），这会导致开销增大。
    -   会过滤出冗余数据，但这是以额外的计算和数据迁移为代价

##  MemStore存储结构
### ConcurrentSkipListMap跳表


##  目前常用的key-value数据结构有三种：Hash表、红黑树、SkipList，它们各自有着不同的优缺点（不考虑删除操作）
*   Hash表：插入、查找最快，为O(1) 如使用链表实现则可实现无锁；数据有序化需要显式的排序操作。
    -   需求的功能有插入、查找、迭代、修改
*   红黑树：插入、查找为O(logn)，但常数项较小；无锁实现的复杂性很高，一般需要加锁；数据天然有序。
    -   而红黑树的插入很可能会涉及多个结点的旋转、变色操作，因此需要在外层加锁，这无形中降低了它可能的并发度
*   SkipList：插入、查找为O(logn)，但常数项比红黑树要大；底层结构为链表，可无锁实现；数据天然有序。
    -   可以实现为lock free，同时它还有着不错的性能（单线程下只比红黑树略慢），非常适合用来实现我们需求的那种key-value结构。

## HFILE结构
*   https://blog.csdn.net/javaman_chen/article/details/47833715

## hbase 2.0
*   https://mp.weixin.qq.com/s/ZaW1edCB-9wuqN7rL99teA


## https://yq.aliyun.com/live/616

## hbase tutorial
* https://www.yiibai.com/hbase/
* https://www.tutorialspoint.com/hbase/index.htm
* https://www.guru99.com/hbase-tutorials.html
* https://data-flair.training/blogs/hbase-tutorial/



