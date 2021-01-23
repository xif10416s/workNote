##    Spark SQL 自适应执行优化引擎 

###  背景
*	Adaptive Execution 将可以根据执行过程中的中间数据优化后续执行，从而提高整体执行效率。核心在于两点
	*	执行计划可动态调整
	*	调整的依据是中间结果的精确统计信息
*	spark 2.3 开始试验功能
*	spark 3.0 正式发布 自适应查询执行（Adaptive Query Execution）



#### 功能

* 动态合并shuffle 分区
* 动态选择join策略
* 动态优化数据倾斜join



###  现状与挑战
*	如何设置合适的shuffle partition数量？
	*	在Spark SQL中， shufflepartition数可以通过参数spark.sql.shuffle.partition来设置，默认值是200。
	*	如果partition太小，单个任务处理的数据量会越大，在内存有限的情况，就会写文件，降低性能，还会oom
	*   如果partition太大，每个处理任务数据量很小，很快结束，导致spark调度负担变大，中间临时文件多
	*   开启AOE后，会合并少量数据的分区为一个分区
*	spark sql 最佳执行计划
	*	spark join 策略
		*	broadcast hash join :  性能最高，要求join的其中一张表数据量不大，如小于10M
		  *	存在一些无法估算的情况：
		    *	存在比较复杂的过滤器
		    *	连接关系是一系列复杂的运算符，而不仅仅是扫描
		*	sort merge join:  性能差
	*	Spark SQL的Catalyst优化器的核心工作就是选择最佳的执行计划,主要依靠：
	  *	早起基于规则的优化器RBO
	  *	spark2.2 加入基于代价的优化CBO 
	*	执行计划在计划阶段确定后，不会改变，如果能够获取运行时信息，就可能得到一个更加的执行计划
*	数据倾斜如何处理
	*	数据倾斜是指某一个partition的数据量远远大于其它partition的数据，导致个别任务的运行时间远远大于其它任务，因此拖累了整个SQL的运行时间。
	*	常见手段：
		*	增加shuffle partition数量，让热点partition的数据分散一些，但是对于同一个key没有作用
		*	增加 BroadcastHashJoin的阈值，在某些场景下可以把SortMergeJoin转化成BroadcastHashJoin而避免shuffle产生的数据倾斜。
		*	手动过滤倾斜key,加入前缀，join表也对key膨胀处理，再join
	*	spark 能否运行时自动处理join中的数据倾斜

### 自适应执行架构
*	基础流程
	*	sql -> 解析 -> 逻辑计划 -> 物理计划 -> rdd -> job -> dag -> stage -> task run
		*	一旦执行计划确定，无法更新
	*	
*	![](https://mmbiz.qpic.cn/mmbiz_png/wvkocF2MXjWndLdS72JOqymLBJU99EfEEeXYwMq7ZRRibKg57wOYXLtvOdhjxc2jI1C4Xgwypv3CoUrz2E6KkeQ/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)
*	自适应划分依据
	*	按照每个reducer处理partition数据内存大小分，每个64m
	*	按照每个reducer处理partition数据条数分，100000条

### 动态调整执行计划
*	在运行时动态调整join的策略，在满足条件的情况下，即一张表小于Broadcast阈值，可以将SortMergeJoin转化成BroadcastHashJoin。
	*	SortMergeJoin,每个reducer通过网络shuffle读取属于自己的数据；会出现不同程度的数据倾斜问题；
	*	BroadcastHashJoin，每一个reducer读取一个mapper的整个shuffle output文件，shuffle读变成了本地读取，没有数据通过网络传输；数据量一般比较均匀，也就避免了倾斜；

### 动态处理数据倾斜
*	在运行时很容易地检测出有数据倾斜的partition，当执行某个stage时，我们收集该stage每个mapper 的shuffle数据大小和记录条数
*	如果某一个partition的数据量或者记录条数超过中位数的N倍，并且大于某个预先配置的阈值，我们就认为这是一个数据倾斜的partition，需要进行特殊的处理

### spark 使用

#### 配置参数
*	org.apache.spark.sql.internal.SQLConf
*	spark.sql.adaptive.enabled=true 
	*	倾斜处理开关   
*	spark.sql.adaptive.shuffle.targetPostShuffleInputSize
	*	动态调整 reduce 个数的 partition 大小依据。如设置 64MB，则 reduce 阶段每个 task 最少处理 64MB 的数据。默认值为 64MB。
*	spark.sql.adaptive.minNumPostShufflePartitions -- v2.4 有 3.0 已经去掉
	*	动态调整 reduce 个数的 partition 条数依据。如设置 20000000，则 reduce 阶段每个 task 最少处理 20000000 条的数据。默认值为 20000000。
*	spark.sql.adaptive.forceApply -- V3.0
	*	自适应执行在没有需要shuffle或者子查询的时候将不适用，当设为true始终使用
*	spark.sql.adaptive.logLevel --v3.0
	*	自适应执行时产生的日志等级
*	spark.sql.adaptive.advisoryPartitionSizeInBytes -- v3.0
	*	倾斜数据分区拆分，小数据分区合并优化时，建议的分区大小 
	*	与spark.sql.adaptive.shuffle.targetPostShuffleInputSize含义相同
*	spark.sql.adaptive.coalescePartitions.enabled -- v3.0
	*	是否开启合并小数据分区默认开启，调优策略之一
*	spark.sql.adaptive.coalescePartitions.minPartitionNum -- v3.0
	*	合并后最小的分区数
*	spark.sql.adaptive.fetchShuffleBlocksInBatch -- v3.0
	*	是否批量拉取blocks,而不是一个个的去取
	*	给同一个map任务一次性批量拉取blocks可以减少io	提高性能
*	spark.sql.adaptive.skewJoin.enabled
	*	自动倾斜处理，处理 sort-merge join中的倾斜数据
*	spark.sql.adaptive.skewJoin.skewedPartitionFactor 
	*	判断分区是否是倾斜分区的比例
	*	当一个 partition 的 size 大小大于该值（所有 parititon 大小的中位数）且大于spark.sql.adaptive.skewedPartitionSizeThreshold，或者 parition 的条数大于该值（所有 parititon 条数的中位数）且大于 spark.sql.adaptive.skewedPartitionRowCountThreshold，才会被当做倾斜的 partition 进行相应的处理。默认值为 10
*	spark.sql.adaptive.skewJoin.skewedPartitionThresholdInBytes
	*	单个分区大于默认256MB










## 参考
*	https://issues.apache.org/jira/browse/SPARK-23128
*	https://blog.csdn.net/weixin_34006468/article/details/91894261
*	https://www.cnblogs.com/zz-ksw/p/11254294.html
*	https://mp.weixin.qq.com/s?__biz=MzA5MTc0NTMwNQ==&mid=2650722029&idx=3&sn=c39cb00ee962bd54c1f99ed4504c940c&chksm=887dd39bbf0a5a8d6ef1a753eda9ab816df98f91a6a3113381f95a21403805a14949398cb97f&scene=126&sessionid=1594686073&key=6f737cad8e87f27eb03c998ed87cd576f834065c2946c0bcc4246c87886047a63fb8d084216d67e123493e3c4936e61e809c5eb695f7d9de377e70b2e3df6c5dd751ede7b6f4d4b42b4f21b59de2e53b&ascene=1&uin=Mjk1NTAwNzcwMg%3D%3D&devicetype=Windows+10+x64&version=62090070&lang=zh_CN&exportkey=AQSj2CRZl0rC8tmmGS%2BtKzg%3D&pass_ticket=4M%2BOtcq%2F6GV06SFEcCdtzzblo8n18XKfhcY5enzpRGSzUWB%2BTZfvBLvwxyhoqylD