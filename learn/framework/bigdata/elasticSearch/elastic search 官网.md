# elastic search 官网
*   https://www.elastic.co/guide/en/elasticsearch/reference/current/index.html
*   http://cwiki.apachecn.org/pages/viewpage.action?pageId=4883367
*   定位: 全文搜索分析引擎
    -   快速（近实时）存储，查询，分析大量数据
*   应用场景
    -   电商网站将商品等信息保存在es,提供商品查询功能
    -   日志数据分析，趋势，指标，统计
    -   比价网站数据存储和搜索
    -   bi,数据分析
*   集群
    -   多个节点，保存所有数据和索引，提供查询服务
    -   相同集群使用相同的集群名称
*   节点
    -   单台机器，集群的一部分
    -   数据存储，参与集群索引与查询功能
    -   有一个随机的UUID标识，服务启动时分配
    -   通过设置集群名称加入特点的集群，默认集群为elasticsearch
*   index -- 关系型数据库里的 database
    -   定义了文档的逻辑存储和字段类型，每个索引可以包含多个文档类型，文档类型是文档的集合，文档以索引定义的逻辑存储模型，比如，指定分片和副本的数量，配置刷新频率，分配分析器等，存储在索引中的海量文档分布式存储在ElasticSearch集群中。
*   type -- database 里的 table
    -   索引中的逻辑分区，用来存储不同的类型的文档
    -   允许我们在一个 index 里存储多种类型的数据，这样就可以减少 index 的数量了
    -   使用 type 的一个好处是，搜索一个 index 下的多个 type，和只搜索一个 type 相比没有额外的开销 —— 需要合并结果的分片数量是一样的。
    -   但是，这也是有限制的：
        +   不同 type 里的字段需要保持一致。例如，一个 index 下的不同 type 里有两个名字相同的字段，他们的类型（string, date 等等）和配置也必须相同。
        +   得分是由 index 内的统计数据来决定的。也就是说，一个 type 中的文档会影响另一个 type 中的文档的得分。
    -   同一个 index 的中的 type 都有类似的映射 (mapping) 时，才应该使用 type。否则，使用多个 type 可能比使用多个 index 消耗的资源更多。
*   document -- 相当于关系表的数据行
    -   存储数据的载体，包含一个或多个存有数据的字段； 
    -   能够被查询的基本单元
*   Shards 
    -   分片，Index 存储在多个分片中，其中每一个分片都是一个独立的 Lucene Index，均匀分布在集群上
    -   功能：
        +   水平扩展
        +   分布式并行处理
    -   每个分片为Luncene index
        +   单个分片的最大文档数为2,147,483,519 (= Integer.MAX_VALUE - 128) 
*   Replicasedit -- 备份，副本
    -   高可用，容错，并行处理
    -   每个索引可以有多个分片，每个分片可以有多个副本
        +   主分片
        +   备份分片 -- 复制主分片
    -   索引创建后，副本数可以修改，分片数不能修改
    -   默认每个index 有5个分片，一个备份，总功10个分片

##  Elasticsearch filter和query的不同
### filter  -- equals
*   此文档和查询子句匹配吗？
    -   确定是否包含在检索结果中，回答只有“是”或“否”。
    -   在搜索中没有额外的相关度排名。
    -   适用于完全精确匹配，范围检索。
*   典型应用场景：
    -   时间戳timestamp 是否在2015至2016年范围内？
    -   状态字段status 是否设置为“published”？
*   经常使用的过滤器将被Elasticsearch自动缓存，以提高性能。
*   过滤（filter）的目标是减少必须由评分查询（query）检查的文档数量

####    post_filter
*   当你需要对搜索结果和聚合结果做不同的过滤时，你才应该使用 post_filter 
    -   搜索后结果过滤，聚合结果不过滤
*   post_filter 的特性是在查询 之后 执行，任何过滤对性能带来的好处（比如缓存）都会完全失去。


### query  -- like
*   此文档与此查询子句的匹配程度如何
    -   确定文档是否应该成为结果的一部分.
    -   查询子句还计算了表示文档与其他文档相比匹配程度的_score
    -   得分越高，相关度越高
*   典型应用场景：
    -   全文检索——这种相关性的概念非常适合全文搜索，因为很少有完全“正确”的答案。
    -   语义相关检索
*   查询结果不可缓存

### version types -- 进行版本控制
*   internal
    -   要求指定的version字段和当前的version号相同
*   external or external_gt
    -   当指定时，保存文档的版本大于当前文档当版本是操作才能成功，并更新为最新的版本

### stored fields vs source
*   source字段存储的是索引的原始内容
    -   从每一个stored field中获取值都需要一次磁盘io，如果想获取多个field的值，就需要多次磁盘io，但是，如果从_source中获取多个field的值，则只需要一次磁盘io，因为_source只是一个字段而已。所以在大多数情况下，从_source中获取是快速而高效的。
    -   es中默认的设置_source是enable的，存储整个文档的值。这意味着在执行search操作的时候可以返回整个文档的信息。如果不想返回这个文档的完整信息，也可以指定要求返回的field，es会自动从_source中抽取出指定field的值返回（比如说highlighting的需求）
*   指定一些字段store为true，这意味着这个field的数据将会被单独存储。这时候，如果你要求返回field1（store：yes），es会分辨出field1已经被存储了，因此不会从_source中加载，而是从field1的存储块中加载。


## api
### _update_by_query  -- 可以根据查询到的结果进行更新


##  Mapping（映射）-- scheam
*   Mapping是用来定义一个文档（document），以及它所包含的属性（field）是如何存储和索引的。
*   每一个索引都有一个或多个映射类型，用于在一个索引中把文档划分为具有逻辑关系的分组。比如，用户文档应该存储为user 类型，博客应该放置于blogpost类型下。


##  aggregations聚合操作
*   Bucketing -- 满足特定条件的文档的集合
    -   key + 规则  ==》 遍历所有文档，将匹配规则的文档放入对应key的bucket(桶内)
        +   根据key 分组
    -   可以有sub Bucketing aggregations
*   Metric -- 指标
    -   在一组文档上计算指标
*   Matrix -- 矩阵
    -   不支持script
*   Pipeline -- 管道
    -   在上一个聚合的基础是继续聚合操作

###  metrics aggregations
*   计算字段 = 文档中的字段 or scripts 生成
*   Avg Aggregation <= 对某个字段 求均值
*   Weighted Avg Aggregation <= 有2个字段 = 统计字段 + 权重字段 ，计算公式： ∑(value * weight) / ∑(weight) 
*   Cardinality Aggregation <== 相当于 distinct count 统计不同值的个数
    -   基于HyperLogLog++算法
*   Geo Bounds Aggregation <= 地理位置边界
    -   返回一个包含所有地理位置坐标点的边界的经纬度坐标，这对显示地图时缩放比例的选择非常有用。
*   Geo Centroid Aggregation <= 给定一组地址计算中心地址
*   Max Aggregation <= 最大值
*   Percentiles Aggregation <= 百分位计算
*   Stats Aggregation <== 综合统计 count，sum,max,min,avg
*   Value Count Aggregation <== 数量统计，看看这个字段一共有多少个不一样的数值


### Bucket Aggregations
*   Adjacency Matrix Aggregation
*   Children Aggregation
*   Composite Aggregation
*   Date Histogram Aggregation
    -   日期间隔分布
*   Diversified Sampler Aggregation
*   Filter Aggregation

### 别名Aliases
#### 添加别名
```
POST /_aliases
{
    "actions" : [
        { "add" : { "index" : "test1", "alias" : "alias1" } }
    ]
}

PUT /{index}/_alias/{name}
```

#### 删除别名
```
POST /_aliases
{
    "actions" : [
        { "remove" : { "index" : "test1", "alias" : "alias1" } }
    ]
}
```

#### 重命名别名
```
POST /_aliases
{
    "actions" : [
        { "remove" : { "index" : "test1", "alias" : "alias1" } },
        { "add" : { "index" : "test2", "alias" : "alias1" } }
    ]
}
```


### xpack sql
*   不支持range ,data_range
*   http://zhengzf.com/2018/06/20/es/xpack-sqlvselasticsearch-sql/
