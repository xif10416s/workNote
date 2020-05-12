####  redis 和 memcached 有啥区别？
*	redis 支持复杂的数据结构
	*	redis 相比 memcached 来说，拥有更多的数据结构，能支持更丰富的数据操作。如果需要缓存能够支持更复杂的结构和操作， redis 会是不错的选择。
*	redis 原生支持集群模式
	*	memcached 没有原生的集群模式，需要依靠客户端来实现往集群中分片写入数据。
*	性能对比
	*	 redis 只使用单核，而 memcached 可以使用多核，所以平均每一个核上 redis 在存储小数据时比 memcached 性能更高。而在 100k 以上的数据中，memcached 性能要高于 redis，虽然 redis 最近也在存储大数据的性能上进行优化，但是比起 memcached，还是稍有逊色。
*	redis 的线程模型
	*	redis 内部使用文件事件处理器 file event handler，这个文件事件处理器是单线程的。它采用 IO 多路复用机制同时监听多个 socket，根据 socket 上的事件来选择对应的事件处理器进行处理。

#### redis 持久化机制
*	Redis的一种持久化方式叫快照（snapshotting，RDB）
	*	Redis可以通过创建快照来获得存储在内存里面的数据在某个时间点上的副本
	*	快照持久化是Redis默认采用的持久化方式，在redis.conf配置文件中有配置
	*	特点： 
		*	全量内存序列化备份，恢复速度快
		*	fork一个子进程，对主进程不影响，内存加倍
		*	备份时间长，性能差，间隔不能太短，丢失数据多
*	另一种方式是只追加文件（append-only file,AOF）
	*	与快照持久化相比，AOF持久化 的实时性更好，因此已成为主流的持久化方案
	*	特点：
		*	保存的是操作明细，恢复需要重放，恢复时间慢
		*	可以配置秒级别刷新，丢失数据少
*	混合方式：
	*	按小时级别保存快照，按秒级别aof
	*	重启时按最近快照加载aof，然后更新增量aof

#### fork copy-on-write
*	创建一个一模一样的进程，内存也复制
*	fork（）会产生一个和父进程完全相同的子进程，但子进程在此后多会exec系统调用，出于效率考虑，linux中引入了“写时复制“技术，也就是只有进程空间的各段的内容要发生变化时，才会将父进程的内容复制一份给子进程。
	*	 在fork之后exec之前两个进程用的是相同的物理空间（内存区），子进程的代码段、数据段、堆栈都是指向父进程的物理空间，也就是说，两者的虚拟空间不同，但其对应的物理空间是同一个
	*	写时复制技术：内核只为新生成的子进程创建虚拟空间结构，它们来复制于父进程的虚拟究竟结构，但是不为这些段分配物理内存，它们共享父进程的物理空间，当父子进程中有更改相应段的行为发生时，再为子进程相应的段分配物理空间。

#### redis使用场景
*	String 
	*	字符串 -- session，对象，小文件
	*	int -- 秒杀，限流，技术
	*	bitmap --web，离线分析
		*	统计：任意时间窗口内用户登录的次数
		*	统计活跃用户，活动 
		*	权限控制
		*	bloom 过滤器
	*	底层编码
	  *	int编码 -- 保存可以用long表示的整数
	  *	raw编码--保存长度大于44字节的字符串 -- 长字符串
	  *	emStr编码--保存长度小于44字节的字符串 -- 短字符串
	*	Redis中对于浮点型也是作为字符串保存的，在需要时再将其转换成浮点数类型
	*	编码的转换
	  *	当 int 编码保存的值不再是整数，或大小超过了long的范围时，自动转化为raw
	  *	对于 embstr 编码，由于 Redis 没有对其编写任何的修改程序（embstr 是只读的），在对embstr对象进行修改时，都会先转化为raw再进行修改，因此，只要是修改embstr对象，修改后的对象一定是raw的，无论是否达到了44个字节。
*	list --放入顺序，栈（同向） ，队列（异向），数组
	
	*	分布式 -- 数据共享，迁出状态，消息队列，利用LRANGE命令，实现基于Redis的分页功能
	*	实现数据即可狗
	  *	Stack（栈）
	    *	LPUSH+LPOP
	  *	Queue（队列）
	    - LPUSH + RPOP
	  *	Blocking MQ（阻塞队列）
	    - LPUSH+BRPOP
	*	编码
	  *	列表对象的编码可以是ziplist（压缩列表）和linkedlist（双端链表）。
	  *	编码转换
	    *	同时满足下面两个条件时使用压缩列表ziplist(长度和每个元素大小都不大的情况)：
	      *	列表保存元素个数小于512个
	      *	每个元素长度小于64字节
	    *	不能满足上面两个条件使用linkedlist（双端列表）编码
*	hashmap 
	
	*	商品详情页--固定内容，聚合场景
*	set -- 集合，去重，无序  ，成本高
	*	SRANDMEMBER  随机抽取几个元素，--抽奖
	*	推荐系统
	*	去重，
*	zset -- 有序集合
	* object encoding key :ziplist ,skiplist
	
	* 排行榜,评论+分页
	
	* 编码
	
	  * 有序集合的编码可以使ziplist或者skiplist
	
	    * ziplist  -- 数量和每个元素都不大
	
	      *	保存的元素数量小于128
	      *	保存的所有元素长度都小于64字节
	
	    *	skiplist编码的依序集合对象使用zset结构作为底层实现，一个zset结构同时包含一个字典和一个跳跃表
	
	      *	```
	        typedef struct zset{
	         //跳跃表
	        zskiplist *zsl;
	        //字典
	        dict *dice;
	        }zset
	        字典的键保存元素的值，字典的值保存元素的分值，跳跃表节点的object属性保存元素的成员，跳跃表节点的score属性保存元素的分值。这两种数据结构会通过指针来共享相同元素的成员和分值，所以不会产生重复成员和分值，造成内存的浪费。
	        ```

#### redis分布式集群

*	高可用 -- 主从备份，切换
*	压力 -- 分片集群
*	一致性hash算法  （Canssandra也是）

#### AKF -- 根据业务划分数据到不同的redis
*	x：冗余备份，高可用
*	y: 业务治理分库
*	z: 单业务分片

#### redis内存管理--过期机制
*	主动--定期删除
	*	默认10s检查一次，删除过期数据
	*	频率设置： config get hz
*	被动--惰性删除
	*	当客户端请求时，处理
*	如果redis内存占满（没有设置ttl)

#### Redis数据类型、编码、数据结构的关系

* Redis内部使用一个redisObject对象来表示所有的key和value，每次在Redis数据块中创建一个键值对时，一个是键对象，一个是值对象，而Redis中的每个对象都是由redisObject结构来表示。

  ```
  redisobject源码
  typedef struct redisObject{
       //类型:string, list , set ,zet..
       unsigned type:4;
       //编码
       unsigned encoding:4;
       //指向底层数据结构的指针
       void *ptr;
       //引用计数
       int refcount;
       //记录最后一次被程序访问的时间
       unsigned lru:22;
  }robj
  
  ```

* encoding类型

* | 编码常量（Encoding）      | 对应的底层数据结构        | redis类型              |
  | :------------------------ | ------------------------- | ---------------------- |
  | REDIS_ENCODING_INT        | long类型的整数            | String                 |
  | REDIS_ENCODING_EMBSTR     | emStr编码的简单动态字符串 | String                 |
  | REDIS_ENCODING_RAW        | 简单动态字符串SDS         | String                 |
  | REDIS_ENCODING_HT         | 字典，hashtable           | SET,HASH               |
  | REDIS_ENCODING_LINKEDLIST | 双向链表                  | List                   |
  | REDIS_ENCODING_ZIPLIST    | 压缩列表                  | List,HASH(数量小),ZSET |
  | REDIS_ENCODING_INTSET     | 整数集合                  | SET                    |
  | REDIS_ENCODING_SKIPLIST   | 跳表和字典                | ZSET                   |
  |                           |                           |                        |



#####  简单动态字符串SDS 优势、

* sds本质分为三部分：**header、buf、null结尾符**，其中header可以认为是整个sds的指引部分，给定了使用的空间大小、最大分配大小等信息
  * ![img](.\sds.png)

* **O(1)获取长度**: C字符串需要遍历而sds中有len可以直接获得；
* **防止缓冲区溢出bufferoverflow**: 当sds需要对字符串进行修改时，首先借助于len和alloc检查空间是否满足修改所需的要求，如果空间不够的话，SDS会自动扩展空间，避免了像C字符串操作中的覆盖情况；
* **有效降低内存分配次数**：C字符串在涉及增加或者清除操作时会改变底层数组的大小造成重新分配、sds使用了**空间预分配和惰性空间释放机制**，说白了就是每次在扩展时是成倍的多分配的，在缩容是也是先留着并不正式归还给OS，这两个机制也是比较好理解的；
* **二进制安全**：C语言字符串只能保存ascii码，对于图片、音频等信息无法保存，sds是二进制安全的，写入什么读取就是什么，不做任何过滤和限制；



#### 内存回收和内存共享

  * 内存回收 ---  **因为c语言不具备自动内存回收功能，当将redisObject对象作为数据库的键或值而不是作为参数存储时其生命周期是非常长的，为了解决这个问题，Redis自己构建了一个内存回收机制，通过redisobject结构中的refcount实现.这个属性会随着对象的使用状态而不断变化。**
    * 创建一个新对象，属性初始化为1
    * 对象被一个新程序使用，属性refcount加1
    * 对象不再被一个程序使用，属性refcount减1
    * 当对象的引用计数值变为0时，对象所占用的内存就会被释放
  * 内存共享 **refcount属性除了能实现内存回收以外，还能实现内存共享**
    * 将数据块的键的值指针指向一个现有值的对象
    * 将被共享的值对象引用refcount加1 Redis的共享对象目前只支持整数值的字符串对象。之所以如此，实际上是对内存和CPU（时间）的平衡：共享对象虽然会降低内存消耗，但是判断两个对象是否相等却需要消耗额外的时间。对于整数值，判断操作复杂度为o(1),对于普通字符串，判断复杂度为o(n);而对于哈希，列表，集合和有序集合，判断的复杂度为o(n^2).虽然共享的对象只能是整数值的字符串对象，但是5种类型都可能使用共享对象。



#### redis I/O模型 -- TODO



#### redis 客户端

| 名称     | 特点                                                         | 优缺点                                                       | 适用场景                                           |
| -------- | ------------------------------------------------------------ | ------------------------------------------------------------ | -------------------------------------------------- |
| jedis    | API提供了比较全面的Redis命令的支持,Java方法基本和Redis的API保持着一致 | 比较全面的Redis命令的支持<br />不支持异步操作。Jedis客户端实例不是线程安全的，需要通过连接池来使用Jedis。 | springboot 1.5.x版本的默认的Redis客户端            |
| Redisson | 实现了分布式和可扩展的Java数据结构，提供很多分布式相关操作服务，例如，分布式锁，分布式集合，可通过Redis支持延迟队列 | 功能较为简单，不支持字符串操作，不支持排序、事务、管道、分区等Redis特性 |                                                    |
| Lettuce  | 基于Netty框架的事件驱动的通信层，其方法调用是异步的。Lettuce的API是线程安全的，所以可以操作单个Lettuce连接来完成各种操作。 | 用于线程安全同步，异步和响应使用，支持集群，Sentinel，管道和编码器 | springboot 2.x版本中默认客户端是用 lettuce实现的。 |
|          |                                                              |                                                              |                                                    |



####  redis sprintboot集成

#####  redisson
* 参考：
  * https://github.com/redisson/redisson/blob/master/redisson-spring-boot-starter/README.md
  * https://www.ithere.net/article/174
* 客户端
  * RedissonClient
  * RedisTemplate
  * ReactiveRedisTemplate
  



#### Lettuce --TODO



#### 测试代码地址  
*	https://github.com/xif10416s/java_test
   

  



##### 参考：

*	https://gitee.com/SnailClimb/JavaGuide/blob/master/docs/database/Redis/Redis.md
*	https://www.icodingedu.com/course/41
*	https://www.runoob.com/redis/redis-conf.html
*	https://gitee.com/SnailClimb/JavaGuide/blob/master/docs/database/Redis/redis-collection/Redis(10)%E2%80%94%E2%80%94Redis%E6%95%B0%E6%8D%AE%E7%B1%BB%E5%9E%8B%E3%80%81%E7%BC%96%E7%A0%81%E3%80%81%E6%95%B0%E6%8D%AE%E7%BB%93%E6%9E%84%E7%9A%84%E5%85%B3%E7%B3%BB.md
*	https://cloud.tencent.com/developer/article/1500854
*	Jedis api 在线网址：http://tool.oschina.net/uploads/apidocs/redis/clients/jedis/Jedis.html
*	redisson 官网地址：https://redisson.org/
*	redisson git项目地址：https://github.com/redisson/redisson
*	lettuce 官网地址：https://lettuce.io/
*	lettuce git项目地址：https://github.com/lettuce-io/lettuce-core
*	https://www.cnblogs.com/yangzhilong/p/7605807.html
*	https://github.com/redisson/redisson/wiki/目录
*	https://github.com/redisson/redisson/wiki/%E7%9B%AE%E5%BD%95
*	https://mp.weixin.qq.com/s?__biz=MzA5MTc0NTMwNQ==&mid=2650719859&idx=1&sn=b55cd8838329737c3056681a4ed1e430&chksm=887ddb05bf0a52138f027cf0b20a8b3a80c6f256baaf290b121f6ac74396169355ae9e083bbd&scene=126&sessionid=1588727501&key=99ef7414318fbe187906b9d55955ae67700442af3a87c5d6ae82c4d182176b0f85cd13de5d5439fceab6b2ba4bc6b91782b12b149cd872059c18184f02e017fbd2b1621f9606ff8878ecf4c18e0e49c2&ascene=1&uin=Mjk1NTAwNzcwMg%3D%3D&devicetype=Windows+10&version=62080079&lang=zh_CN&exportkey=AfrwBXccwRBcndnocCkYjcE%3D&pass_ticket=LPSbkDJNYtM03WvFhUCwCDhlPxk2J8JL7vu0h%2FKRQNaVG30YE5Z7z3K%2FQ4ckpqvB

