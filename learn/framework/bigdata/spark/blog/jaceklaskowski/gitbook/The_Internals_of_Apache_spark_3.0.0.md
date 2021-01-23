##   The Internals Of Apache Spark 3.0.0 笔记 --TODO

####   原文（https://books.japila.pl/apache-spark-internals/apache-spark-internals/index.html）



####  Resilient  Distributed Dataset

* 弹性分布式数据集，spark core模块对于数据集的核心抽象
* RDD 就是对于分布式环境中一组可以容错的、弹性计算的数据集合的描述：
  * 容错：指当有一台服务器有问题、或者某个计算出错时，可以在其他服务器重新计算
  * 弹性 ：通过RDD血缘图（依赖关系）可以重新计算一些丢失或者损坏的分区
  * 分布式： 数据集分布在集群中的很多服务器节点上。
* 

