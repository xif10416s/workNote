##嵌套的排序过的map结构

    cassandra的column families ==>map[rowKey,sortedmap[column_name,value]]==>key 排序过

    数据模型的建立根据查询业务来

    CQL的schema定义的列名相当于一个主体,主体的值才算列名

 

CREATE TABLE events (

        key text,

        column1 int,

        column2 int,

        value text,

        PRIMARY KEY(key, column1, column2)

    ) WITH COMPACT STORAGE

key的值才是cassandra的列名,所以说列是动态增加,但是主体key必须是固定的,

###更复杂的例子

一个更复杂一点的例子:

CREATE TABLE example (
    partitionKey1 text,
    partitionKey2 text,
    clusterKey1 text,
    clusterKey2 text,
    normalField1 text,
    normalField2 text,    PRIMARY KEY (
        (partitionKey1, partitionKey2),
        clusterKey1, clusterKey2
        )
    );

这里我们用字段在内部存储时的类型来命名字段。而且我们已经包含了所有情形。这里的主键不仅仅是复合主键，而且是复合partition key （在PRIMARY KEY部分的前半段，用括号括起的部分）和复合cluster key。

然后插入一些行:

INSERT INTO example (
    partitionKey1,
    partitionKey2,
    clusterKey1,
    clusterKey2,
    normalField1,
    normalField2
    ) VALUES (    'partitionVal1',    'partitionVal2',    'clusterVal1',    'clusterVal2',    'normalVal1',    'normalVal2');

数据怎么存储呢？

RowKey: partitionVal1:partitionVal2
=> (column=clusterVal1:clusterVal2:, value=, timestamp=1374630892473000)
=> (column=clusterVal1:clusterVal2:normalfield1, value=6e6f726d616c56616c31, timestamp=1374630892473000)
=> (column=clusterVal1:clusterVal2:normalfield2, value=6e6f726d616c56616c32, timestamp=1374630892473000)

    注意partitionVal1和partitionVal2，我们可以发现RowKey（也称为partition key）是这两个字段值的组合

    clusterVal1和clusterVal2这两个cluster key的  值（注意是值不是字段名称）的组合，成为了每一个非主键列名的前缀

    非主键列的值，比如normalfield1和normalfield2的值，是列名加上cluster key的值之后的列的值

    每一个row中的列，是按列名排序的，而因为cluster key的值成为非主键列名的前缀，每个row下的所有的列，实际上首先按照cluster key的值排序，然后再按照CQL里的列名排序


##模型设计

    集群负载均衡最终依靠rowkey选择；相反地，rowkey也决定row的长度。所以在设计模型的时候要谨记负载均衡。

    因为row不会跨节点分割，所以单个row必须适应节点磁盘大小

    一种减轻问题的方式是添加一些信息到rowkey——事件类型、机器ID，或者适应你用例的类似值。
    确保column key和row key是唯一的
    cassandra所有操作都是upsert（不存在插入，存在则更新）操作。
    保持column name简短--key的值
    对某个row key的一次mutation操作是原子的。所以当你需要事务性的时候，尝试设计你的模型，让它一次永远只更新一行。
    在Cassandra中，TTL(存活时间)不是设置在column family上，它设置于每个column value，一旦设置后就很难改变。或者说，如果创建column时候没有设置，那之后就很难对其设置TTL。
    composite columns 优于 super columns



