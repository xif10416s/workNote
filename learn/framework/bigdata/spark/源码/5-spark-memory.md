# 5-spark-memory.md

### 内存分类
*   堆内内存(On-heap Memory)
	*	默认情况下，Spark 仅仅使用了堆内内存。Executor 端的堆内内存区域大致可以分为以下四大块：
		*	Execution 内存：主要用于存放 Shuffle、Join、Sort、Aggregation 等计算过程中的临时数据
		*	Storage 内存：主要用于存储 spark 的 cache 数据，例如RDD的缓存、unroll数据；
		*	用户内存（User Memory）：主要用于存储 RDD 转换操作所需要的数据，例如 RDD 依赖等信息。
		*	预留内存（Reserved Memory）：系统预留内存，会用来存储Spark内部对象。
		*	![](../images/spark_on_heap_memory_iteblog.png)
		*	systemMemory = Runtime.getRuntime.maxMemory，其实就是通过参数 spark.executor.memory 或 --executor-memory 配置的。
		*	usableMemory = systemMemory - reservedMemory
*	堆外内存(Off-heap Memory)
	*	默认情况下，堆外内存是关闭的，我们可以通过 spark.memory.offHeap.enabled 参数启用，并且通过 spark.memory.offHeap.size 设置堆外内存大小，单位为字节
	*	这个时候 Executor 中的 Execution 内存是堆内的 Execution 内存和堆外的 Execution 内存之和
	*	![](../images/Spark_off_heap_memory_iteblog.png)


###	spark 内存构建MemoryManager -- 在 sparkEnv中 
*	MemoryManager分为2大类
	*	StaticMemoryManager -- spark.memory.useLegacyMode 为true 时（默认false)
		*	在spark1.6之前采用静态内存管理
	*	UnifiedMemoryManager 
		*	动态管理	execution 与 storage 的空间交换
		*	reservedMemory 默认 300m ， 所以 usableMemory = systemMemory - 300m
	*	相关配置
		*	spark.memory.fraction *  usableMemory -- 决定了 (execution  + storage) 的内存
			*	usableMemory * (1- spark.memory.fraction)  = userMemory
		*	spark.memory.storageFraction 为 spark.memory.fraction *  usableMemory 比例中 storage 内存占比
			*	1 -  spark.memory.storageFraction 为 execution 内存占比
		*	案例
			*	--executor-memory 10g  -- 10240m  , spark.memory.fraction = 0.6 , spark.memory.storageFraction = 0.7
			*	usableMemory = 10240m   - 300m = 9940‬m
			*   User Memory = 9940‬ * ( 1- 0.6) = 3976m
			*	execution = 9940‬ * 0.6 * (1-0.7) = 1789m
			*	storeage = 9940 * 0.6 * 0.7 =8150m
	*	spark.memory.offHeap.size  -- 堆外内存大小

### UnifiedMemoryManager
*	内存管理
	*	execution memory refers to that used for computation in shuffles, joins,sorts and aggregations,
 	*   while storage memory refers to that used for caching and propagating internal data across the cluster
*	MemoryManager --  UnifiedMemoryManager 继承自MemoryManager 
	*	MemoryManager 构造是需要指定onHeapStorageMemory 与 onHeapExecutionMemory 用户设定storage 内存 和 executoion内存大小，单位是long
	*	MemoryManager 主要有4个成员变量：
		*	onHeapStorageMemoryPool = new StorageMemoryPool
			*	堆内存储内存池，用于堆内内存分配和管理，初始化的时候会设定对于的存储内存大小
		*	offHeapStorageMemoryPool = new StorageMemoryPool
		*	onHeapExecutionMemoryPool = new ExecutionMemoryPool
			*	堆内运行内存池，用于共享合适的内存给task使用，初始化的时候设定执行内存大小
		*	offHeapExecutionMemoryPool = new ExecutionMemoryPool	
*	MemoryPool -- 内存池，管理内存的使用，有2个实现类：
	*	StorageMemoryPool -- 管理和分配给定大小的内存给缓存用
		*	acquireMemory(blockId: BlockId, numBytes: Long): Boolean -- 获取内存
			*	通过初始化给定的总内存poolSize 与 已经使用的内存 memoryUsed 计算 剩余可用的内存大小，与给定的blockid需要的内存numBytes比较：
				*	如果剩余内存够用，则返回true
				*	如果剩余内存比需要的少，则需要通过MemoryStore从现有的使用中移除evict一部分，释放内存
			*	结论：
				*	StorageMemoryPool 主要是记录实际使用了多少，能否分配给新的缓存对象，以及结合MemoryStore触发释放缓存操作以扩充可用内存
				*	实际的存储和释放都是MemoryStore操作
	*	ExecutionMemoryPool -- 分配合适的内存给每个task
		*	memoryForTask = new mutable.HashMap[Long, Long]() 记录每个task用的内存量
		*	memoryUsed =  memoryForTask.values.sum 所有任务使用的总和
		*	`acquireMemory(
      numBytes: Long,
      taskAttemptId: Long,
      maybeGrowPool: Long => Unit = (additionalSpaceNeeded: Long) => Unit,
      computeMaxPoolSize: () => Long = () => poolSize): Long  `
      		*	某个task taskAttemptId 需要的内存量
      		*	maybeGrowPool ： 回掉函数，当内存不够时扩展所需的内存
      		*	computeMaxPoolSize ： 获取当前执行内存的最大值，这个值会发生变化，如通过回收storage的内存扩充
      		*	用于管理任务所共享的内存
*	UnifiedMemoryManager  -- 作为 storeage 与 execution 内存管理的 统筹 入口
	*	acquireStorageMemory -- 是否可获取存储内存
		*	如果storage内存不够了，但是execution够用，会从execution内存借用，动态调整storage，execution 内存上限 ， 然后获取storeag内存
			*	executionPool.decrementPoolSize(memoryBorrowedFromExecution)
			*	storagePool.incrementPoolSize(memoryBorrowedFromExecution)
			*	storagePool.acquireMemory(blockId, numBytes)
	*	acquireExecutionMemory -- 获取运行内存大小
		*	这个方法当没有足够内存时会阻塞直到有内存为止
		*	如果execution内存不够，会清除storeage中存储对象，是否内存，用来扩充execution的上限
*	MemoryStore -- 存储block 
	*	一组序列化的java对象 或者  序列化的ByteBuffers
	*	TODO

##  spark cache and load block size 

## Peak Execution memory 
*   refers to the memory used by internal data structures created during shuffles, aggregations and joins.



#### 参考
*	https://www.iteblog.com/archives/2342.html
