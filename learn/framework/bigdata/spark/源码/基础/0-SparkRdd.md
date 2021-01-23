#   Resilient Distributed Datasets (RDDs)

##  类定义
```
abstract class RDD[T: ClassTag](
    @transient private <var></var> _sc: SparkContext,
    @transient private var deps: Seq[Dependency[_]]
  ) extends Serializable with Logging
```
*   RDD与RDD之间有依赖关系deps，新建RDD时指定
*   类似Java中的IO流，装饰器模式
    -   最基础的RDD[java io 中的节点流 如 fileInputStream]负责从数据源读取数据
        +   ParallelCollectionRDD,内存中的数据
        +   HadoopRDD ,文件系统读取数据
    -   RDD的操作处理，会指定依赖关系，包装成新的RDD
        +   MapPartitionsRDD
        +   ShuffledRDD
```
data = 1 until n 
sparkContext.parallelize(data , slices)   |    new ParallelCollectionRDD
.map(t => (cleanF(t), t))                 |    new MapPartitionsRDD[U, T](ParallelCollectionRDD,xx)
.groupbyKey                               |    new ShuffledRDD(MapPartitionsRDD,XXXX)
-----------------------------------------
ShuffledRDD -prev-> MapPartitionsRDD -prev-> ParallelCollectionRDD -data-> 1 until n  
ShuffledRDD #deps = nil         
MapPartitionsRDD#deps = OneToOneDependency[_rdd= ParallelCollectionRDD]
ParallelCollectionRDD#deps = nil  
```

## 关联关系
### 构造RDD时，prev字段 与  deps 关系
*   prev字段为子类字段，deps为RDD基类定义字段
*   通常情况prev直接传递给deps
    -  如MapPartitionsRDD 构造时传入的prev rdd 会调用父类构造器
    this(oneParent.context, List(new OneToOneDependency(oneParent)))转换为deps
*   prev字段有，deps为nil的情况
    -   ShuffledRDD，通过重写getDependencies方法使用prev

### 用途
*   DagScheduler划分stage时需要调用RDD#getDependencies方法获取所有依赖关系，查找ShuffleDependence为边界划分stage

##  Dependency，依赖
### 子类 -- 不同的转换操作会有对应的依赖关系
*   NarrowDependency，
*   ShuffleDependency
    -   groupbyKey
*   OneToOneDependency extends NarrowDependency
    -   map
*   RangeDependency extends NarrowDependency
    -   union

##  Partition ,分区，逻辑上的数据块，并行处理的基础单位，一个partition对应一个并行处理任务
*   带一个索引，表示分区号
*   描述如何切分物理数据成逻辑数据块

###  常用partition
*   ParallelCollectionPartition
    -   对应ParallelCollectionRDD
    -   重写了writeObject，readObject，发送task执行时调用，如何序列化partition
*   org.apache.spark.rdd.JdbcPartition
    -   根据某个long key 范围平均划分partition数的数据
        +   JdbcPartition(idx: Int, val lower: Long, val upper: Long)
*   org.apache.spark.sql.execution.datasources.jdbc.JDBCPartition
    -   根据where条件划分
        -   JDBCPartition(whereClause: String, idx: Int)
*   HadoopPartition
    -   HadoopPartition(rddId: Int, override val index: Int, s: InputSplit)

## 主要方法
*   def compute(split: Partition, context: TaskContext): Iterator[T]
    -   定义如何计算某个partition
*   persist,cache 
    -   缓存中间结果，迭代重复使用
*   def iterator(split: Partition, context: TaskContext): Iterator[T] 
    -   task计算时调用，

##  基本操作
###   Transformations操作 -- 返回RDD
####  map\[U: ClassTag\](f: T => U): RDD\[U\]
*   调用函数f ,将每个T类型元素转换成U类型元素，iter.map(cleanF)
*   通过包装成 MapPartitionsRDD
*   依赖关系OneToOneDependency

####  flatMap\[U: ClassTag\](f: T => TraversableOnce\[U\]): RDD\[U\]
*   调用函数f ,将每个T类型元素转换成U类型元素，并扁平化结果集，iter.flatMap(cleanF)
*   通过包装成 MapPartitionsRDD
*   依赖关系OneToOneDependency

####   filter(f: T => Boolean): RDD\[T\]
*   每个元素调用函数f,返回为false的元素过滤，iter.filter(cleanF)
*   通过包装成 MapPartitionsRDD

####    union(other: RDD\[T\): RDD\[T\]
*   union(Seq(first) ++ rest) -> new UnionRDD(this, rdds) 
*   将两个rdd合并成一个seq转换成UnionRDD
*   UnionRDD
    -   getPartitions定义如何把rdd数组看成一个rdd获取分区
        +    for ((rdd, rddIndex) <- rdds.zipWithIndex; split <- rdd.partitions)
        +    第一层rdd数组，第二层rdd对应的partition , new UnionPartition(pos, rdd, rddIndex, split.index)
    
####   repartition(numPartitions: Int)  ==》 coalesce(numPartitions, shuffle = true)
*   重新划分partition
    *   numPartitions 减少 不会有shuffle操作
    *   numPartitions 增加 需要指定  shuffle = true
*   numPartitions 减少非shuffle 的情况
    -   new CoalescedRDD(this, numPartitions, partitionCoalescer)、
        +   通过DefaultPartitionCoalescer，缩减partition //TODO
*   shuffle = true的情况
    -   new CoalescedRDD(
        new ShuffledRDD\[Int, T, T\](mapPartitionsWithIndex(distributePartition),
        new HashPartitioner(numPartitions)),
        numPartitions,
        partitionCoalescer).values

####   groupBy[K](f: T => K)(implicit kt: ClassTag[K]): RDD[(K, Iterable[T])]
*   转为k,v形式 this.map(t => (cleanF(t), t))，new MapPartitionsRDD
*   groupByKey(p)
    -   new ShuffledRDD



### actions操作 -- 返回的是值
####    reduce(f: (T, T) => T): T 
*   需要准备reducePartition函数，如何合并partition迭代结果，在远程端执行
*   mergeResult函数，如何合并partition返回结果，在driver端






