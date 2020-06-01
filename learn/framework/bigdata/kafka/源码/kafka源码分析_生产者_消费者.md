#### kafka - 2.5

#### [kafka测试代码地址](https://github.com/xif10416s/java_test)

####  producer 



#### consumer
* Java客户端是围绕一个由poll() API驱动的事件循环设计的
* java consumer 没有后台线程，所有io操作都依赖：poll ()
  * 加入consumer group ，以及处理partitoin rebanances
  * 定期发送心跳
  * 当autocommit开启时，定期提交offset
  * 给指定分区发送拉去请求并接受数据
* 因为是单线程模型，当处理接收到的返回消息时是不能发送心跳
  * 所以要保证消息可以及时处理
* 线程不安全
* 一个partition只能分配给一个consumer，一个consumer可以处理多个partition
* 新版本的将kafka consumer的消费位置保存在了“__consumer_offsets”的内部topic
  * 当出现消费者上线或者离线时，consumer group触发rebalance



#####  消费者offset提交

* 自动提交，开启enable.auto.commit, 设置提交间隔时间每次consumer poll()会检查是否需要提交
* 手动提交：
  * commitSync
  * commitAsync



####  kafka rebalance

######  旧方案，zookeeper 监控与通知

* 每个consumer监控 /consumers/[group_id]/ids 与 /brokers/ids ，注册一个watcher;当节点发生变化时触发rebalance
* 问题：
  * 羊群效应：一个被watch的节点变化，导致大量watcher通知需要被发送给客户端，期间会造成其他操作延迟
  * 闹裂：每个consumer都是通过zookeeper中保存的元数据判断consumer group的状态，broker的状态，rebanlance的结果，由于zookeeper保证最终一致性，不同的consumer在同一时刻可能看到不一样的元数据

###### 新方案

* 将Consumer group分成多个子集，每个Consumer group子集 
* 每个consumer group子集对应一个GroupCoordinator进行管理
* 每个consumer group 在zookeeper的consumers目录有个文件，只有GroupCoordinator在zookeeper上添加了watcher
  * 当触发rebalance时只有GroupCoordinator收到通知，开始处理rebalance
* 分区的分配操作放在了consumer端



#####  数据流

###### 数据流可以分为四种不同的类型

* producer与 transaction coordinator的交互

  * initTransactions API向协调器注册一个transactional.id
  * 当生产者将在事务中第一次将数据发送到分区时，首先要向协调器注册该分区。
  * 当应用程序调用commitTransaction或abortTransaction时，会将请求发送到协调器以开始两阶段提交协议

* #### coordinator 与 transaction log 交互

  * 随着事务的进行，生产者发送上述请求以更新协调器上的事务状态
  * 事务协调器将其拥有的每个事务的状态保存在内存中，并将该状态写入事务日志
  * 事务协调器是唯一从事务日志中读取和写入的组件
  * 如果给定的代理挂了，会选举新的协调器作事务日志分区的leader，并且它将从传入分区中读取消息，以为这些分区中的事务重建其内存中状态

* producer写入消息到目标topic的partition

  * 在协调器在事务中注册新分区之后，生产者像往常一样将数据发送到实际分区

* 事务协调器与目标topic的partition交互

  * 生产者发起提交（或中止）之后，协调器开始两阶段提交协议
    * 在第一阶段，协调器将其内部状态更新为“ prepare_commit”，并在事务日志中更新此状态；
      * 一旦完成，无论发生什么事情，事务都将得到保证
    * 第二阶段，协调者将提交到主题分区的数据标记为事务已提交
      * 标记对应producer是不可见的，主要给consumer根据不同的隔离等级（*read_committed* ）过滤数据用



######  事务实战

* 开启事务导致写入放大，额外的写操作：
  * 对于每个事务，我们都有额外的RPC在协调器中注册分区；这些是批处理的，rpc数量比分区数少。
  * 完成交易时，必须将一个交易标记写入参与交易的每个分区
  * 我们将状态更改写入事务日志；包括对添加到事务中的每批分区的写入，“ prepare_commit”状态和“ complete_commit”状态
* 消费者事务执行， 不会有明显的性能损坏
  * 过滤掉属于取消事务的消息
  * 过滤未提交的事务的消息





















#### 参考

* [demo文档](https://docs.confluent.io/current/clients/java.html)
* [kafka事务confluent.io](https://www.confluent.io/blog/transactions-apache-kafka/)