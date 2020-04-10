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
*	另一种方式是只追加文件（append-only file,AOF）
	*	与快照持久化相比，AOF持久化 的实时性更好，因此已成为主流的持久化方案