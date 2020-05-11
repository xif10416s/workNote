#### [Apache Spark 3.0 动态分区裁剪](https://www.iteblog.com/archives/8590.html)

#### 静态分区裁剪（Static Partition Pruning）

* spark sql 在执行查询的时候根据过滤条件实现谓词下推，分区剪裁，跳过不必要的分区，减少读取数据量

  *  select *  from  sales where day_of_week = "mon"
  * spark 在编译时会把filter算子下推到数据源，转成数据源查询过滤条件，达到减少读取数据量的目的
* 编译时进行对事实表的分区剪裁优化
* 有星型模式连接查询的情况：
  * select *  from sales join date on sales.date.id = date.id where date.day_of_week = 'mon'
  * 在没有开启动态剪裁的情况，谓词下推的优化发生在维度表date中，事实表sales还是会全量加载，然后执行join
* ![Apache Spark 3.0 动态分区裁剪（Dynamic Partition Pruning）介绍](../..\images\sfp.jpg)



#### 动态分区剪裁

* 动态分区裁剪这个功能是 [Spark 3.0](https://www.iteblog.com/archives/tag/spark-3-0/) 引入的，详见 [SPARK-11150](https://www.iteblog.com/redirect.php?url=aHR0cHM6Ly9pc3N1ZXMuYXBhY2hlLm9yZy9qaXJhL2Jyb3dzZS9TUEFSSy0xMTE1MA==&article=true)、[SPARK-28888](https://www.iteblog.com/redirect.php?url=aHR0cHM6Ly9pc3N1ZXMuYXBhY2hlLm9yZy9qaXJhL2Jyb3dzZS9TUEFSSy0yODg4OA==&article=true)。

* 基于运行时推断信息进一步分区剪裁
* 开启动态剪裁的情况，上面的计划会在scan sales 生成一个过滤器，先进行过滤



#### 执行动态分区剪裁的条件

* `spark.sql.optimizer.dynamicPartitionPruning.enabled` 参数必须设置为 true
* 需要裁减的表必须是分区表，而且分区字段必须在 join 的 on 条件里面
* Join 类型必须是 INNER, LEFT SEMI （左表是分区表）, LEFT OUTER （右表是分区表）, or RIGHT OUTER （左表是分区表）。
* 满足上面的条件也不一定会触发动态分区裁减，还必须满足
  * `spark.sql.optimizer.dynamicPartitionPruning.useStats` 和 `spark.sql.optimizer.dynamicPartitionPruning.fallbackFilterRatio` 两个参数综合评估出一个进行动态分区裁减是否有益的值，满足了才会进行动态分区裁减。 --TODO



#### 相关配置

* spark.sql.optimizer.dynamicPartitionPruning.enabled  -- 动态分区剪裁开关
* spark.sql.optimizer.dynamicPartitionPruning.useStats （默认 true) -- 



#### 参考

* https://www.iteblog.com/archives/8590.html
* https://www.iteblog.com/archives/8589.html

