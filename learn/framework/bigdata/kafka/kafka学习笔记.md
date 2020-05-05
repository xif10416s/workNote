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



##### kafka特性

* 使用了page cache



#####  producer 发送数据

* Producer向Leader发送数据时,可以通过acks参数设置数据可靠性的级别
  * 0: 不论写入是否成功,server不需要给Producer发送Response,如果发生异常,server会终止连接,触发Producer更新meta数据
  * 1: Leader写入成功后即发送Response,此种情况如果Leader fail,会丢失数据
  * -1: 等待所有ISR接收到消息后再给Producer发送Response,这是最强保证 
     仅设置acks=-1也不能保证数据不丢失,当Isr列表中只有Leader时,同样有可能造成数据丢失。要保证数据不丢除了设置acks=-1, 还要保 证ISR的大小大于等于2,具体参数设置:
    * min.insync.replicas: 设置为大于等于2,保证ISR中至少有两个Replica 

#### Kafka集群partitions/replicas默认分配解析







































#### 参考

* https://blog.csdn.net/lizhitao/article/details/39499283