####  官方文档  --  https://kafka.apache.org/intro  -- v2.5

##### 概要
* 分布式流数据平台，包含以下三个能力：
  * 发布和订阅数据流中的记录，类似消息队列与企业消息系统 -- 消息系统
  * 以容错和持久的方式存储数据流的记录 --- 高可用持久
  * 当数据流中的记录发生时处理它们    --- 实时处理
*  主要应用于两大类应用程序
  * 构建实时流数据管道，在系统或应用程序之间可靠地获取数据  --- 应用程序之间实时通信
  * 实时应用系统，实时转换或处理流数据 --- 实时处理
*	主要概念
	*	kakfa可以作为集群运行，多个集群可以跨数据中心 -- 分布式集群部署
	*	kafka的数据被分类存储在称为topics中
	*	每个记录由一个键、一个值和一个timestam组成
* 核心api
  *  [Producer API](https://kafka.apache.org/documentation.html#producerapi)  生产者api,向一个或多个topic发送消息
  *  [Consumer API](https://kafka.apache.org/documentation.html#consumerapi)消费者api, 消费订阅的topic中到达的消息
  *  [Streams API](https://kafka.apache.org/documentation/streams)流处理api，从一个输入流消费，并输出到一个或多个topic的输出流，高效的处理输入流的消息并传输到输出流上
  *  [Connector API](https://kafka.apache.org/documentation.html#connect) 创建可重用的生产者和消费者
  * [Admin API](https://kafka.apache.org/documentation.html#adminapi) 管理和查看topics,brokers等

#####  核心概念

###### [Topics and Logs](https://kafka.apache.org/intro#intro_topics)

* topic 逻辑上是一个分类名称，消息会被发送到指定的topic，每个topic表示同一类的消息
* 每个topic可以有一个，或多个订阅者，消费这个topic收到的数据
* 在分布式系统中，数据需要分片，提高性能和吞吐量，每个topic的分片就是partition
* 一个topic会有一个或者多个partition
* 每个partition是一个commit log结构，连续末尾追加的方式写入，有序且不可改变
  * 每个partition中的记录都会被分配一个别称为offset的序列号，在当前partition中唯一
  * 每个partiton会对应一个物理文件
* consumer会维护一个offset表示消费数据的位置，所以数据的消费完全由consumer控制

###### [Distribution](https://kafka.apache.org/intro#intro_distribution)

* topic的所有partition 分布在kafka集群的多台服务器上，每台服务器都处理在partition上的数据和请求
* 每个分区又可以配置多个副本，同一个分区的多个副本分布在不同的服务器上，实现容错
* 每个分区在多个服务器上的副本，其中一个副本会成为leader, 其他的成为followers -- 主从
* leader负责该partition所有的读写请求,其他followers被动的同步leader的数据 -- 副本同步
* 当leader挂掉后，会从follower中选择一个新的leader -- 高可用
* 集群中的所有服务器都会扮演某些分区的leader, 以及某些分区的follower- - 负载均衡

###### [Producers](https://kafka.apache.org/intro#intro_producers)

* 生产者发送数据到指定的topic
* 生产者需要决定数据被发送到topic的哪个分区--通常根据key+算法（hash%partition数量)

###### [Consumers](https://kafka.apache.org/intro#intro_consumers)

* 通过给消费者指定相同的group名，指定为同一组消费者
* 一条topic的消息只能被同一个组订阅该topic的消费者消费 --- 消息队列的queue
* topic的所有数据会均衡的分配到同一组的消费者实例处理
* 如果不是同一组的consumer,会各自处理各自的消息 --  消息队列的topic 

###### [Multi-tenancy](https://kafka.apache.org/intro#intro_multi-tenancy)

* kafka支持多租户部署
* 多租户是通过配置哪些主题可以产生或消耗数据来实现的。
* 也有对配额的操作支持。管理员可以在请求上定义和执行配额，以控制客户端使用的代理资源。

###### [Guarantees](https://kafka.apache.org/intro#intro_guarantees)

* 在高级别Kafka提供以下保证：
  *  发送给同一个分区的数据是有序的，但是发送给不同分区的数据不保证有序
  * 一个消费者实例是按照数据在日志中存储的顺序查看的
  * 对于具有复制因子N的主题，我们将容忍多达N-1服务器故障而不丢失提交到日志的任何记录。



#####  使用案例

* [Messaging](https://kafka.apache.org/uses#uses_messaging) -- 消息系统
  * kakfa 有高的吞吐量，内建分区，副本容错机制，使得非常适合大型可扩展的消息处理应用
* [Website Activity Tracking](https://kafka.apache.org/uses#uses_website)---网站行为跟踪
* [Metrics](https://kafka.apache.org/uses#uses_metrics) -- 统计
* [Log Aggregation](https://kafka.apache.org/uses#uses_logs)--分布式系统日志收集
* [Stream Processing](https://kafka.apache.org/uses#uses_streamprocessing)--流处理
* [Event Sourcing](https://kafka.apache.org/uses#uses_eventsourcing)--日志，事件追踪
* [Commit Log](https://kafka.apache.org/uses#uses_commitlog)--



##### kafka设计动机

* kafka被设计成为处理所有实时数据流的统一平台
* 需要高吞吐量能支持高容量的数据流，如实时的日志收集
* 它需要优雅地处理大量的积压数据，以便能够支持来自离线系统的定期数据加载分析
* 支持分区的，分布式的，实时处理的数据流，并创建新的数据流
* 保证机器出现故障时的容错能力

##### 数据持久保存

* Kafka严重依赖于文件系统来存储和缓存消息
* 合理设计的磁盘结构可以和网络传输速度一样快
* 对于普通磁盘使用顺序读写方式比随机读写快几千倍
* 现代操作系统提供了预读和后写技术，这种技术可以将大块的数据进行多次预取，并将较小的逻辑写操作分组为较大的物理写操作

##### 端到端的数据压缩

* 通常情况系统性能瓶颈不是cpu或者磁盘，而是网络带宽
* kafka支持批处理的方式，并进行批量压缩后进行网络传输，最终消费者会解
* 支持压缩：GZIP，LZ4，snappy

##### producer负载均衡

* producer 直接把数据发送给broker上对应的leader parition, 不需要任何中间路由层
* producer可以向任一kafka节点请求元数据信息，包括哪些服务器存活在，topic的partition的leader节点位置
* 客户端控制消息发送到哪个partition, 通过一些算法实现负载均衡.
* 选择合适的key, 如用户id作为分区键，可以把所有相同id的数据发送到同一个partition

##### 生产者异步发送

* 批量处理是一个重要的提升性能的方式，kafka producer 会积累消息在内存，通过一个请求发送批量数据
* 发送的阈值可以通过配置：固定的数据量（64k) 或者 固定的时间间隔（10ms)

##### Consumer的 push vs pull

* kafka consumer 是通过从kafka服务器的leader 分区拉去需要消费的数据
* consumer 通过指定log中的offset，每次请求就获取offset之后的一段日志
  * consumer 对于如何消费数据是完全自己控制的，通过offset实现
* 通常消费者需要最快速的消费数据
  * 基于服务端推送的消息系统对于不同的consumer实例不方便控制消费速度
  * 基于consumer拉去的方式，consumer可以根据自己的处理速度连续处理
* consumer 拉去的方式，可以设置自己合适的批处理数据量

##### consumer position

* kafka的每个topic被划分成多个有序的partition，每个partition会有一个consumer group中的consumer消费。
* 消费者在每个分区中的位置只是一个整数，offset记录了下一次消费的开始位置
* consumer可以控制offset位置，重新消费数据

##### offline data load

* 可伸缩持久性允许那些只定期使用批数据加载的使用者，这些批数据加载会定期批量加载数据到离线系统(如Hadoop或关系数据仓库)

##### 消息投递语义

* at most once (最多一次) --- 消息可能丢失

* at least once(最少一次) ---消息可能被多次投递

* Exactly once(精确一次，只一次) -- 每个消息只投递一次

  * producer 保证 ： 0.11.0.0版本之后，broker会给每个producer 分配一个id，并且每个producer发送的消息都有唯一序列号；producer支持类似事务的方式发送消息给多个topic，要么所有topic都成功，要么失败
  * consumer保证：consumer的位置被存储为topic中的消息，因此我们可以在接收处理过的数据的输出主题的相同事务中将偏移量写入Kafka
    * consumer处理数据后还是写到kafka对应的其他topic，consumer的offset也需要更新到topic，producer可以支持发送多个topic事务
    * 如果要把结果写到外部系统，通过kafka connect实现，

  


##  kafka 数据丢失问题
### producer
*   https://blog.csdn.net/zhangjun5965/article/details/78218169
*   https://www.zybuluo.com/tinadu/note/949867
*   https://3gods.com/bigdata/Kafka-Message-Delivery-Semantics.html


## kafka问题汇总
*	https://mp.weixin.qq.com/s?__biz=MzA5MTc0NTMwNQ==&mid=2650718181&idx=1&sn=69667c8316358ac66a8f5d5412429135&chksm=887da293bf0a2b85469c388312c1cba191c5bccf8946db2808cd06a7aa9c56c7ae595f5db2b9&scene=21#wechat_redirect

##  kafka 事务
*	http://www.jasongj.com/kafka/transaction/