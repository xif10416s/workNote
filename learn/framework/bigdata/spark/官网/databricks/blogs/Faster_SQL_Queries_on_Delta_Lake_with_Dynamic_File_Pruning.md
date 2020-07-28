####   通过动态文件剪裁实现在delta late上的快速查询

* [原文](https://databricks.com/blog/2020/04/30/faster-sql-queries-on-delta-lake-with-dynamic-file-pruning.html)

* 两种传统的数据查询加速的方案：
  * 以更快的速度处理数据
  * 通过跳过无关的数据，减少数据量
    * Dynamic File Pruning (DFP) ： 动态文件裁剪是一种新的数据跳过（data-skipping, 直接跳过一些不相关的数据分区文件）技术
      * 对于在delta lake 表上选择 非 分区列join的时候可以显著提高查询效率
* 对于数量较大的delta lake表， 通常会选择分区策略，使得在查询和作业的时候可以跳过大量数据
* 当过滤条件中出现分区列时，分区裁剪可以发生在编译时期，也可以发生在运行时期
  * 动态分区裁剪就是运行时的分区裁剪



####  delta lake 动态裁剪工作方式

* 因为delta lake 会自动收集被管理文件的元数据信息，所以可以在运行时跳过不需要的文件
* 在动态文件裁剪（DFP）之前，仅当查询在where 中包含分区值时才进行文件修剪
* 现在join的过滤条件也可以进行文件裁剪，这意味着动态文件裁剪现在允许星型模式查询利用文件粒度的数据跳过功能。
* 说明：通常事实表数据量具体，会按日期作为分区，之前只有当where条件出现事实表的分区键过滤时，才能进行跳过不需分区表，如果是事实表join 了维度表，根据维度字段进行过滤时，原来不支持分区裁剪，现在也可以支持了。

```
查询方式一: 
    SELECT sum(ss_quantity) 
    FROM store_sales 
    WHERE ss_item_sk IN (40, 41, 42) 
   
   delta lake 为每一个基础文件中包含的所有列都保存对应的最大最小值，因此ss_item_sk字段值在（40，41，42）范围以外的文件可以直接跳过
   delta lake 使用Z-Ordering（TODO）等数据聚类技术来减少每个文件的值范围的长度,这对于动态文件修剪非常有吸引力，因为每个文件的范围更小会导致更好的跳过效果
   本次查询会执行谓词下推操作，通过元数据操作进行剪裁作为SCAN操作的一部分，先跳过不必要的分区进行读取，然后还需要filter算子执行具体的条件过滤
   当过滤器包含文字谓词时，查询编译器可以将这些文字值嵌入查询计划中
   当过滤条件作为join的一部分时，需要另外一种方式，因为当编译时期还不知道事实表的join filter
```

![](../../..\images\pdf1.png)



```
查询方式二：
SELECT sum(ss_quantity) 
    FROM store_sales  -- 事实表
    JOIN item ON ss_item_sk = i_item_sk  --维度表
    WHERE i_item_id = 'AAAAAAAAICAAAAAA'  -- 维度字段
    
  通过星型模式（事实表，维度表）表连接，并且过滤条件是 维度表的维度字段 i_item_id ， 不在事实表上
  这意味着，事实表无法进行裁剪，需要全量读取后join.
  事实表读取86亿数据，与3条维度表join，输出48k数据
```

![Example query where filtering of rows for store_sales would typically be done as part of the JOIN operation since the values of ss_item_sk are not known until after the SCAN and FILTER operations take place on the item table.](../../..\images\pdf2.png)

```
查询方式二的情况使用动态文件裁剪,从联接的构建侧创建了动态过滤器，并将其传递到store_sales的SCAN操作中
store_sales事实表从86亿降低到6.6千万
```

![Example query with Dynamic File Pruning enabled, where a dynamic filter is created from the build side of the join and passed into the SCAN operation for store_sales.](../../..\images\pdf3.png)

#### 动态文件裁剪开启条件

* 连接的表需要delta lake 格式
* 连接类型是 inner 或者 LEFT-SEMI
* 连接策略是BROADCAST HASH JOIN （ 小表）
* 大表的文件数量需要达到一定的阈值





####  [spark 3.0  动态分区剪裁](../../源码/features/ApacheSpark3.0动态分区裁剪.md)

