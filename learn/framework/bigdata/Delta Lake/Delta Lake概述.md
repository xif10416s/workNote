#   Delta Lake概述.md
*	https://www.iteblog.com/archives/2596.html
*	 Parquet文件 + Meta 文件 + 一组操作的API = Delta Lake.

##  数据湖
*	数据湖概念的诞生，源自企业面临的一些挑战，如数据应该以何种方式处理和存储。
*	数据湖不但能存储传统类型数据，也能存储任意其他类型数据，并且能在它们之上做进一步的处理与分析，产生最终输出供各类程序消费。
*	定义
	*	数据湖是一个存储企业的各种各样原始数据的大型仓库，其中的数据可供存取、处理、分析及传输。
*	Martin Fowler datelake : https://martinfowler.com/bliki/DataLake.html
	*	特点
		*	The data lake stores raw data, in whatever form the data source provides(数据湖存储的是原始数据，不用管原始数据的格式)
		*	There is no assumptions about the schema of the data, each data source can use whatever schema it likes（没有数据schema的假设，每个数据源可以使用任何schema)
		*	It's up to the consumers of that data to make sense of that data for their own purposes(取决于数据消费者对于数据的用途)
	*	数据湖 vs 数据仓库
		*	输入数据处理：
			*	数据仓库--清洗数据，定义数据schema,存入数据仓库
			*	数据湖  --按原始格式存入
		*	数据分析处理：
			*	数据仓库 --直接按之前定义的schema读取分析，所有用户一致
			*	数据湖 -- 按用户各自需求选择和组织数据
		*	总结：
			*	数据仓库倾向于采用单一模式的概念来满足所有分析需求
			*	数据仓库另一个问题来源是确保数据质量
				*	数据质量问题是一个主观意愿，不同分析需求对于数据需求不一样（按日组织，按月组织）
*	生命周期
	*   https://blog.csdn.net/xinshucredit/article/details/88641697

##  概念
*	Delta Lake是一个可靠存储层
*	为Apache Spark和其他大数据引擎提供可伸缩的ACID事务，让用户可以基于HDFS和云存储构建可靠的数据湖为Apache Spark和其他大数据引擎提供可伸缩的ACID事务，让用户可以基于HDFS和云存储构建可靠的数据湖
*	Delta Lake还提供了内置的数据版本控制，可以方便地回滚以及重新生成报告。
*	https://blog.csdn.net/oscarun/article/details/89530621
*	使用versioned Parquet files文件格式来存储数据到云存储，记录所有事务日志，提供ACID支持
*	Delta Lake does not support multi-table transactions and foreign keys. Delta Lake supports transactions at the table level.
*	通常使用Date字段作为分区
	*	使用基数少的字段但作为分区，userid不适合
	*	至少一个分区1个g以上数据量
*	Delta Lake 就是使用 DataSource V1 版本的 API 实现的一种新的数据源

##  官网 --0.3
*	https://docs.delta.io/0.3.0/delta-update.html
*	Delta Lake保证向后兼容，新版本都支持旧版本的表读取，旧的版本不能读取新的版本
*	Delta Lake 事务处理
	*	Delta Lake的工作原理是将事务日志与数据文件一起存储在表中
	*	在table目录下，有一个_delta_log目录，存储着一系列原子操作集合的日志，称为delta文件，json格式
		*	/table/_delta_log/00000000000000000000.json
	*	为了避免每次加载table时候读取整个事务日志文件，会周期性的创建checkpoint，包含table当前版本所有状态信息
*	Delta Lake底层存储系统
	*	Delta Lake 通过LogStore API 访问文件系统，来保证事务处理
	*	Delta Lake 内建了 LogStore 的实现类， HDFS, S3 ， Azure storage services.
		*	配置：https://docs.delta.io/latest/delta-storage.html
*	Table Deletes, Updates, and Merges
	*	删除操作只是把数据从当前版本中移除，并没有从物理存储系统中移除，旧的版本还是可以查到数据，直到旧版本被清除后
*	Delta Lake tables支持两种工具命令
	*	Vacuum -- 清除不再使用的数据，物理删除
	*	History -- 获取数据历操作记录，版本历史
