## redis
*	https://gitee.com/SnailClimb/JavaGuide/blob/master/docs/database/Redis/Redis.md

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
		*	备份时间长，性能差，间隔不能太短，丢失数据多
*	另一种方式是只追加文件（append-only file,AOF）
	*	与快照持久化相比，AOF持久化 的实时性更好，因此已成为主流的持久化方案
	*	特点：
		*	保存的是操作明细，恢复需要重放，恢复时间慢
		*	可以配置秒级别刷新，丢失数据少
*	混合方式：
	*	按小时级别保存快照，按秒级别aof
	*	重启时按最近快照加载aof，然后更新增量aof

#### redis使用场景
*	String 
	*	字符串 -- session，对象，小文件
	*	int -- 秒杀，限流，技术
	*	bitmap --web，离线分析
		*	统计：任意时间窗口内用户登录的次数
		*	统计活跃用户，活动 
		*	权限控制
		*	bloom 过滤器
*	list --放入顺序，栈（同向） ，队列（异向），数组
	*	分布式 -- 数据共享，迁出状态，
*	hashmap 
	*	商品详情页--固定内容，聚合场景
*	set -- 集合，去重，无序  ，成本高
	*	SRANDMEMBER  随机抽取几个元素，--抽奖
	*	推荐系统
*	zset -- 有序集合
	*	object encoding key :ziplist ,skiplist
	*	排行榜,评论+分页

#### redis分布式集群
*	高可用 -- 主从备份，切换
*	压力 -- 分片集群

#### AKF -- 根据业务划分数据到不同的redis
*	x：冗余备份，高可用
*	y: 业务治理分库
*	z: 单业务分片