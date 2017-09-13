#Resilient Distributed Datasets (RDDs)
##基本流程
![](../images/1.jpg)
![](../images/2.png)
##driver worker 通信关系
![](../images/3.jpg)
###rdd执行时序图
![](../images/rdd.jpg)

###概述
*   *1.*   构造RDD,从不同的数据源加载数据
    -   测试方式`rdd1: RDD[Int] = sc.makeRDD(1 to 1000, 10)` <=ParallelCollectionRDD






##org.apache.spark.rdd
#####RDD  extends Serializable
*   是什么 
    -    弹性分布式数据集


#####ParallelCollectionRDD  extends RDD[T]


