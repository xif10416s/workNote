### 总体结构



####  名词解释





####  物理存储结构

* 每个topic的partition对应一个目录，partition目录命名规则 = topic名称+有序序号 ， 从0开始到 partition数量 - 1
* 每个partition按照config/server.properties(log.segment.bytes)的配置大小，分割成很多个segment数据文件，这种特性可以方便old segment file的快速删除。
* partition中的segment file的组成(4个文件）:
  * segment file **组成**：由2部分组成，分别为index file和data file，这两个文件是一一对应的，后缀”.index”和”.log”分别表示索引文件和数据文件；
  * 0.8版本之后，多了个.**timeindex**文件，是kafka的具体时间日志
  * segment文件命名规则：partion全局的第一个segment从0开始，后续每个segment文件名为上一个segment文件最后一条消息的offset值。数值最大为64位long大小，19位数字字符长度，没有数字用0填充。
  * leader-epoch-checkpoint中保存了每一任leader开始写入消息时的offset 会定时更新
     follower被选为leader时会根据这个确定哪些消息可用
  * ![image-20200505180304436](./images\segment.png)
* 说明segment中index<—->data file对应关系物理结构如下：
  * <img src=".\images\smap.png" alt="这里写图片描述" style="zoom: 50%;" />
  * 索引文件每一条是一个message索引，包含两个4个字节数字，一个表示相对offset，一个表示position
    * 3,497 表示：第三条消息的物理偏移位置offset是497
    * position 不是消息的offset, 而是message在segment中的物理位置
  * message数据结构
    * <img src=".\images\message.png" alt="这里写图片描述" style="zoom:50%;" />
  * 数据查询过程--读取offset=368776的message  --TODO
    1. 定位是哪个segment文件： 通过segment文件名二分查找，比如：当offset=368776时定位到00000000000000368769.index|log
    2. 读取message: 解析index文件，根据offset定位到position, 通过position从log文件读取数据
* 存储设计特点
  * Kafka把topic中一个parition大文件分成多个小文件段，通过多个小文件段，就容易定期清除或删除已经消费完文件，减少磁盘占用。
  * 通过索引信息可以快速定位message和确定response的最大大小。
  * 通过index元数据全部映射到memory，可以避免segment file的IO磁盘操作。
  * 通过索引文件稀疏存储，可以大幅降低index文件元数据占用空间大小





#### Kafka创建Topic时如何将分区放置到不同的Broker中

* 副本因子不能大于 Broker 的个数；
* 第一个分区（编号为0）的第一个副本放置位置是随机从 brokerList 选择的
  * 第一个放置的分区副本一般都是 Leader，其余的都是 Follow 副本
* 其他分区的第一个副本放置位置相对于第0个分区依次往后移。也就是如果我们有5个 Broker，5个分区，假设第一个分区放在第四个 Broker 上，那么第二个分区将会放在第五个 Broker 上；第三个分区将会放在第一个 Broker 上；第四个分区将会放在第二个 Broker 上，依次类推；
* 剩余的副本相对于第一个副本放置位置其实是由 nextReplicaShift 决定的，而这个数也是随机产生的；





##### kafka特性，高吞吐

* 使用了page cache
* 顺序读写；
* 零拷贝
* 文件分段
* 批量发送
* 数据压缩。



#### kafka缺点

* 由于是批量发送，数据并非真正的实时；
* 对于mqtt协议不支持；
* 不支持物联网传感数据直接接入；
* 仅支持统一分区内消息有序，无法实现全局消息有序；
* 监控不完善，需要安装插件；
* 依赖zookeeper进行元数据管理；



#####  producer 发送数据

* Producer向Leader发送数据时,可以通过acks参数设置数据可靠性的级别
  * 0: 不论写入是否成功,server不需要给Producer发送Response,如果发生异常,server会终止连接,触发Producer更新meta数据
  * 1: Leader写入成功后即发送Response,此种情况如果Leader fail,会丢失数据
  * -1: 等待所有ISR接收到消息后再给Producer发送Response,这是最强保证 
     仅设置acks=-1也不能保证数据不丢失,当Isr列表中只有Leader时,同样有可能造成数据丢失。要保证数据不丢除了设置acks=-1, 还要保 证ISR的大小大于等于2,具体参数设置:
    * min.insync.replicas: 设置为大于等于2,保证ISR中至少有两个Replica 

#### Kafka集群partitions/replicas默认分配解析



#### kafka增加分区，减少分区  -- TODO test

* 
* 删除分区的问题
  * 删除掉的分区中的消息该作何处理







####  consumer 验证

* 将导致consumer不断在循环中轮询，直到新消息到t达。为了避免这点，Kafka有个参数可以让consumer阻塞知道新消息到达(当然也可以阻塞知道消息的数量达到某个特定的量这样就可以批量发



##### Kafka分区分配策略(Partition Assignment Strategy) -- 同一个 Consumer Group 里面的 Consumer 是如何知道该消费哪些分区里面的数据

* Kafka 内部存在两种默认的分区分配策略：Range 和 RoundRobin。当以下事件发生时，Kafka 将会进行一次分区分配：
  * 同一个 Consumer Group 内新增消费者
  * 消费者离开当前所属的Consumer Group，包括shuts down 或 crashes
  * 订阅的主题新增分区
* 假设我们有个名为 T1 的主题，其包含了10个分区，然后我们有两个消费者（C1，C2）来消费这10个分区里面的数据，而且 C1 的 num.streams = 1，C2 的 num.streams = 2。
  * range strategy
    * partitions的个数除于消费者线程的总数来决定每个消费者线程消费几个分区。如果除不尽，那么前面几个消费者线程将会多消费一个分区
    * 如果消费2个topic，前面的消费线程可能会多消费分区
  * RoundRobin strategy
    * 使用RoundRobin策略有两个前提条件必须满足：	
      * 同一个Consumer Group里面的所有消费者的num.streams必须相等；
      * 每个消费者订阅的主题必须相同。
    * RoundRobin策略的工作原理：将所有主题的分区组成 TopicAndPartition 列表，然后对 TopicAndPartition 列表按照 hashCode 进行排序,最后按照round-robin风格将分区分别分配给不同的消费者线程。



##### kafka监控

* [Apache Kafka监控之KafkaOffsetMonitor](http://mp.weixin.qq.com/s?__biz=MzA5MTc0NTMwNQ==&mid=200581484&idx=1&sn=3e14a01cf1eb514dfdde73a6f032643d&chksm=1e77bc1a2900350cf546522ff9ce4a6387c0f797ba056ff18f19180c06ad6c8817e4ba337974&scene=21#wechat_redirect)
* [雅虎开源的Kafka集群管理器(Kafka Manager)](http://mp.weixin.qq.com/s?__biz=MzA5MTc0NTMwNQ==&mid=203369107&idx=1&sn=badcbedd5dd6bc1975307175d03e1a4f&chksm=199c37e52eebbef35106892a80b641394d6467478537ca0a0ff5a1ac605e336e3f18d45d2282&scene=21#wechat_redirect)
* [Apache Kafka监控之Kafka Web Console](http://mp.weixin.qq.com/s?__biz=MzA5MTc0NTMwNQ==&mid=200584408&idx=1&sn=30a4ac3f68972bcfd8375708ba78de74&chksm=1e77b1ae290038b82bd8fbe9032ade23b7952ec8f8e07acd3d12dc29537ea0612b115956f471&scene=21#wechat_redirect)

























#### 参考

* https://blog.csdn.net/lizhitao/article/details/39499283
* https://mp.weixin.qq.com/s?__biz=MzA5MTc0NTMwNQ==&mid=2650716375&idx=1&sn=4e1ac64aa6f7eaa5a210c3fb51977b7d&chksm=887da5a1bf0a2cb7855e99aecc0e003c3728c34dedc741c70ec9ca18345e541906b9c85c8b00&scene=21#wechat_redirect
* 问题集合：https://mp.weixin.qq.com/s?__biz=MzA5MTc0NTMwNQ==&mid=2650719847&idx=1&sn=5cbc20ae39756e39552a9a056c133a80&chksm=887ddb11bf0a52074f71202b954fb5c6c59d738bab798050f532d280f1b122f4b18306b22bcc&scene=126&sessionid=1588727501&key=99ef7414318fbe1864fb2209e363da1be27454816b0d10f1ac71fc45092acbb5b9f988de5cbaa2737fa564f3bc92db0e80051f07f6d04060537a2c66e0e45ebdd07298f7965380fff46ca4609cf942c3&ascene=1&uin=Mjk1NTAwNzcwMg%3D%3D&devicetype=Windows+10&version=62080079&lang=zh_CN&exportkey=AXdKG%2FgDSdQH82K6MqPnbVk%3D&pass_ticket=LPSbkDJNYtM03WvFhUCwCDhlPxk2J8JL7vu0h%2FKRQNaVG30YE5Z7z3K%2FQ4ckpqvB