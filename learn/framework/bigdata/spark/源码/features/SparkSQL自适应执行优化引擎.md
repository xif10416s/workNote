##   Spark SQL 自适应执行优化引擎 

###  背景
*	Adaptive Execution 将可以根据执行过程中的中间数据优化后续执行，从而提高整体执行效率。核心在于两点
	*	执行计划可动态调整
	*	调整的依据是中间结果的精确统计信息

###  现状与挑战
*	如何设置合适的shuffle partition数量？
	*	在Spark SQL中， shufflepartition数可以通过参数spark.sql.shuffle.partition来设置，默认值是200。
	*	如果partition太小，单个任务处理的数据量会越大，在内存有限的情况，就会写文件，降低性能，还会oom
	*   如果partition太大，每个处理任务数据量很小，很快结束，导致spark调度负担变大，中间临时文件多
*	spark sql 最佳执行计划
	* 	spark 













## 参考
*	https://issues.apache.org/jira/browse/SPARK-23128
*	https://blog.csdn.net/weixin_34006468/article/details/91894261