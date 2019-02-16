# hbase.md
*   http://www.postgres.cn/downfiles/pg2016conf_day2_s3_am1.pdf 
*   /home/hadoop/hbase/bin/hbase-daemon.sh start thrift
*   -Dhbase.log.dir=/data/hbase/logs

##  数据模型 -- sparse, distributed, persistent multidimensional sorted map, which is indexed by a row key, column key, and a timestamp 
###  基本组件
*   Table 
    *    一个字符串名称，在文件系统中对应有个目录
*   Row
    -   在一个表中，数据的存储位置由row决定
    -   rows由唯一的row key标识 ，并且是一个byte数组
*   Column Family
    -   在同一个行中的数据由  Column Family 分组
    -   Column Family 影响数据的物理存储，所以必须预先定义好，不会轻易修改
    -   同一个表中的所有行的Column Family都相同
    -   每一行并不需要存储所有的Column Family数据
    -   在文件系统中有对应路径
    -   不同的family是在同一个region下面。
        -   而每一个family都会分配一个memstore，所以更多的family会消耗更多的内存
    -   由于hfile是以family为单位的，因此对于多个family来说，数据被分散到了更多的hfile中，减小了split发生的机率
*   Column Qualifer
    -   列修饰符被加到一个列簇，以提供对一个数据片段的索引
    -   HBase中的列是二级列,可以每行不一样
*   Cell
    -   rowKey,column family,column qualifier组合唯一表示一个cell
*   Timestamp
    -   cell默认的版本通过Timestamp表示

## hbase 表创建
*   TTL=>的更新超时时间是指：该列最后更新的时间，到超时时间的限制，而不是第一次创建，到超时时间。
    -   TTL（Time-To-Live）：每个Cell的数据超时时间（当前时间 - 最后更新的时间）
    -   MinVersion：如果当前存储的所有时间版本都早于TTL，至少MIN_VERSION个最新版本会保留下来。这样确保在你的查询以及数据早于TTL时有结果返回。
    -   CELL TTL 检查
        +   
*   hbase的预分配region
    -   在create一个表时如果不指定预分配region，则默认会先分配一个region，这样在大数据并行载入时性能比较低，因为所有的数据都往一个region灌入，容易引起单节点负载升高，从而影响入库性能，一个好的方法时在建立表时预先分配数个region
    -   ,  {NUMREGIONS => 9, SPLITALGO => 'HexStringSplit'}
*   Hbase split 自带了两种pre-split的算法
    -   HexStringSplit => 如果我们的row key是十六进制的字符串作为前缀的，就比较适合用HexStringSplit
    -   UniformSplit =>某个hbase的表查询只是以随机查询为主，可以用UniformSplit的方式进行，它是按照原始byte值（从0x00~0xFF）右边以00填充
*   rowkey --https://blog.bcmeng.com/post/hbase-rowkey.html#%E7%83%AD%E7%82%B9

###  宽表 高表 -- https://www.jianshu.com/p/44ffeac1601a
*   hbase中的宽表是指很多列较少行，即列多行少的表，一行中的数据量较大，行数少；高表是指很多行较少列，即行多列少，一行中的数据量较少，行数大。
*   hbase的row key是分布式的索引，也是分片的依据。
*   查询性能：
    -   高表更好，因为查询条件都在row key中, 是全局分布式索引的一部分。高表一行中的数据较少。所以查询缓存BlockCache能缓存更多的行，以行数为单位的吞吐量会更高。
*   分片能力
    -   高表分片粒度更细，各个分片的大小更均衡。因为高表一行的数据较少，宽表一行的数据较多。HBase按行来分片。
*   元数据开销
    -   高表元数据开销更大。高表行多，row key多，可能造成region数量也多，- root -、 .meta表数据量更大。过大的元数据开销，可能引起HBase集群的不稳定、master更大的负担
*   事务能力
    -   宽表事务性更好。HBase对一行的写入（Put）是有事务原子性的，一行的所有列要么全部写入成功，要么全部没有写入。但是多行的更新之间没有事务性保证。
*   数据压缩比
    -   如果我们对一行内的数据进行压缩，宽表能获得更高的压缩比。

### hbase test --http://debugo.com/hbase-shell-cmds/
*   create_namespace 'test';
*   list_namespace;
*   create 'test:mt', 'f'
*   describe 'test:mt'
    -   {NAME => 'f', BLOOMFILTER => 'ROW', VERSIONS => '1', IN_MEMORY => 'false', KEEP_DELETED_CELLS => 'FALSE', DATA_BLOCK_ENCODING =>
 'NONE', TTL => 'FOREVER', COMPRESSION => 'NONE', MIN_VERSIONS => '0', BLOCKCACHE => 'true', BLOCKSIZE => '65536', REPLICATION_S
COPE => '0'}
*   create 'test:mt4', {NAME => 'f',  COMPRESSION => 'SNAPPY',TTL => '200000000'}, {NAME=>'t',TTL => '86400'},  {NUMREGIONS => 2, SPLITALGO => 'HexStringSplit'}
    -   put 'test:mt2',123,'f:20141224',6 ,1432483200000
    -   scan 'test:mt2'
    -   put 'test:mt2',123,'f:20141224',6
    -   scan 'test:mt2'
    -   put 'test:mt2','a123-1','f:20141224',0
    -   put 'test:mt2','a123-2','f:20141224',0
    -   put 'test:mt2','b223','f:20141224',6
    -    scan 'test:mt2', { STARTROW => 'a'}
    -    scan 'test:mt2', { STARTROW => 'a',FILTER=>"PrefixFilter('a')"}
    -    scan 'test:mt2', {Raw =>'true'}
    -    scan 'test:mt2', { STARTROW => 'a123-',STOPROW=> 'a123.', FILTER=>"PrefixFilter('a123-') AND ValueFilter(=,'binary:0')" }
    -    scan 'test:mt2' ,{LIMIT =>1 , FILTER=>"ValueFilter(=,'binary:6')"}
    -    get 'test:mt2' ,'a123'
    -    get 'test:mt2' ,'a123',FILTER=>"ValueFilter(=,'binary:6')"
    -    get 'test:mt2' ,'row-key-6',FILTER=>"SKIP ValueFilter(!=,'binary:7')"
    -    FILTER=>"ColumnPrefixFilter('birth') AND ValueFilter ValueFilter(=,'substring:1987')"
    -    scan 'test:mt2', { STARTROW => 'b223-',STOPROW=> 'b223.', FILTER=>"FirstKeyOnlyFilter() AND ValueFilter(>=,'binary:0') ",LIMIT => 3 }
    -    scan 'test:mt2', { STARTROW => 'row-key-',STOPROW=> 'row-key.', FILTER=>"SKIP ValueFilter(>=,'binary:0') ",LIMIT => 3 }
    -    scan 'test:mt2', { STARTROW => 'row-key-',STOPROW=> 'row-key.', FILTER=>" RowFilter( = ,'regexstring:y') ",LIMIT => 3 }
    -    scan 'test:mt2',{REVERSED => TRUE}
    -    scan 'test:mt2', { STARTROW => 'row-key-',STOPROW=> 'row-key.', FILTER=>"SKIP ValueFilter(!=,'binary:-1') AND QualifierFilter(=,'binary:ugs')",LIMIT => 100 }
    -    put 'test:mt2','row-key-15','f:20141224',0 , {'TTL'=>10000}
    -    scan 'dyd:user_recommend_posts', { STARTROW => '1e156267adf0a575-',STOPROW=> '1e156267adf0a575.', FILTER=>" SingleColumnValueFilter('f','d',=,'binary:-1',true,true)  AND QualifierFilter(=,'binary:cb')",LIMIT => 5000 }
    -    compact 'dyd:user_recommend_posts', 'f'
    -    major_compact 'dyd:user_recommend_posts', 'f'
    -    get 'test:mt2','3a811367adf0a575' , { FILTER=> "(ValueFilter(=,'substring:cus_cf') AND ColumnCountGetFilter(100)   )OR (ValueFilter(=,'substring:ugs') AND ColumnCountGetFilter(100)  ) "}
*   disable 'test:mt2'
    -   drop  'test:mt2'
    -  delete 'test:mt2','a123-2', 'f:20141224'
*   hbase   org.apache.hadoop.hbase.mapreduce.RowCounter 'dyd:user_recommend_posts'  
*   

## filter
*   https://www.cloudera.com/documentation/enterprise/5-5-x/topics/admin_hbase_filtering.html
*   http://hbase.apache.org/0.94/book/thrift.html
*   http://blog.csdn.net/cnweike/article/details/42920547

## 存储结构 -http://blog.csdn.net/qq280929090/article/details/56302851

### key
```
//md5
MessageDigest md = MessageDigest.getInstance("MD5");
byte[] digest = md.digest(Bytes.toBytes(s));



```


## hbase spark
*   https://hbase.apache.org/book.html#spark
*   https://www.iteblog.com/archives/1891.html
*   https://hbase.apache.org/book.html#_bulk_load
*   https://github.com/apache/hbase/blob/master/hbase-spark/src/main/scala/org/apache/hadoop/hbase/spark/example/hbasecontext/HBaseBulkGetExample.scala

```
// load data
import org.apache.hadoop.hbase.spark.HBaseContext
import org.apache.spark.SparkContext
import org.apache.hadoop.hbase.{CellUtil, TableName, HBaseConfiguration}
import org.apache.hadoop.hbase.util.Bytes
import org.apache.hadoop.hbase._
import org.apache.hadoop.hbase.client.Get
import org.apache.hadoop.hbase.client.Result

val a = scala.collection.mutable.ArrayBuffer[Array[Byte]]()
for(i <- 0 to 2){
    val k = s"cb1f97e101647575-ffc81547adf0a575"
    a.append(Bytes.toBytes(k))
}

val rdd = sc.parallelize(a)
val conf = HBaseConfiguration.create()

val hBaseContext = new HBaseContext(sc, conf)
val tableName = "dyd:user_recommend_posts"
val getRdd = hBaseContext.bulkGet[Array[Byte], String](
        TableName.valueOf(tableName),
        1000,
        rdd,
        record => {
          val get = new Get(record)
          get.addColumn(Bytes.toBytes("f"),Bytes.toBytes("test"))
          get.addColumn(Bytes.toBytes("f"),Bytes.toBytes("ugs"))
        },
        (result: Result) => {
         val b = new StringBuilder
          if(result != null && result.listCells() != null) {
          val it = result.listCells().iterator()
          
          while (it.hasNext) {
            val cell = it.next()

            val i =CellUtil.tagsIterator(cell.getTagsArray(), cell.getTagsOffset(), cell.getTagsLength())
             b.append("cell ttl " + ":" + i.hasNext() +"     ")
              while (i.hasNext()) {
                val t = i.next()
                if (TagType.TTL_TAG_TYPE == t.getType()) {
                  val ts = cell.getTimestamp()
                  val ttl = Bytes.toLong(t.getBuffer(), t.getTagOffset(), t.getTagLength())
                  b.append("cell ttl " + ":" + ttl)
                }
              }
            b.append(Bytes.toString(result.getRow) + ":")
            val q = Bytes.toString(CellUtil.cloneQualifier(cell))
            if (q.equals("counter")) {
              b.append("(" + q + "," + Bytes.toLong(CellUtil.cloneValue(cell)) + ")")
            } else {
              b.append("(" + q + "," + Bytes.toString(CellUtil.cloneValue(cell)) + ")")
            }
          }
          }
          b.toString()
        })
      getRdd.collect().foreach(v => println(v))



```
```
//save data
import org.apache.hadoop.hbase.spark.HBaseContext
import org.apache.spark.SparkContext
import org.apache.hadoop.hbase.{TableName, HBaseConfiguration}
import org.apache.hadoop.hbase.util.Bytes
import org.apache.hadoop.hbase.client.Put

import org.apache.spark.SparkConf

val tableName = "dyd:user_recommend_posts"
val columnFamily = "f"

val rdd = sc.parallelize(Array(
        (Bytes.toBytes("cb1f97e101647575-ffc81547adf0a575"),
          Array((Bytes.toBytes(columnFamily), Bytes.toBytes("test343"), Bytes.toBytes("1"))))
      ))

val conf = HBaseConfiguration.create()
val hbaseContext = new HBaseContext(sc, conf)      
hbaseContext.bulkPut[(Array[Byte], Array[(Array[Byte], Array[Byte], Array[Byte])])](rdd,
        TableName.valueOf(tableName),
        (putRecord) => {
          val put = new Put(putRecord._1)
          put.setTTL(5000)
          putRecord._2.foreach((putValue) =>
            put.addColumn(putValue._1, putValue._2, putValue._3))
          put
        });

//---
val df = sc.parallelize(Array(
UserPostIdArray(6293258517833034308L,Array(6293274501954834378L,6293274501954967548L,6293274501955275403L)),
UserPostIdArray(6293207802437871924L,Array(6293274501954834378L,6293274501954967548L,6293274501955275403L)),
UserPostIdArray(6293245510875449232L,Array(6293274501954834378L,6293274501954967548L,6293274501955275403L))
)).toDF()

doResultSave(df,spark,"20180119")

//========================bulkload=============
// 一行一列插入
// 行要rowkey排序，列要排序
val conf = HBaseConfiguration.create()
val tableName = "test:mt2"
val table = new HTable(conf, tableName) 
  
conf.set(TableOutputFormat.OUTPUT_TABLE, tableName)
lazy val job = Job.getInstance(conf)
job.setMapOutputKeyClass (classOf[ImmutableBytesWritable])
job.setMapOutputValueClass (classOf[KeyValue])
HFileOutputFormat.configureIncrementalLoad (job, table)
  
// Generate 10 sample data:
val num = sc.parallelize(1 to 10)
val rdd = num.map(x=>{
    val kv: KeyValue = new KeyValue(Bytes.toBytes(x), "f".getBytes(), "c1".getBytes(), "value_xxx".getBytes() )
    (new ImmutableBytesWritable(Bytes.toBytes(x)), kv)
})
  
// Directly bulk load to Hbase/MapRDB tables.
job.getConfiguration.set("mapred.output.dir", "/tmp/hbase/test/mt3")
rdd.saveAsNewAPIHadoopDataset(job.getConfiguration)

val bulkLoader = new LoadIncrementalHFiles(conf)
bulkLoader.doBulkLoad(new Path("/tmp/hbase/test/mt2"), table)

========batch check and put---
import java.util
import org.apache.hadoop.hbase.protobuf.generated.HBaseProtos.CompareType
import org.apache.hadoop.hbase.client.{HTable, ConnectionFactory, Delete, Scan}
import org.apache.hadoop.hbase.filter.CompareFilter.CompareOp
import org.apache.hadoop.hbase.filter._
import org.apache.hadoop.hbase.spark.HBaseContext
import org.apache.hadoop.hbase.util.Bytes
import org.apache.hadoop.hbase.{HBaseConfiguration, TableName}
import org.apache.hadoop.hbase.client.{Get, ConnectionFactory, Put}
val tableName = "test:mt2"
val conf = HBaseConfiguration.create()
val connection = ConnectionFactory.createConnection(conf)
val table = connection.getTable(TableName.valueOf(tableName)).asInstanceOf[HTable]   
table.setAutoFlushTo(false)
table.setWriteBufferSize(64*1024*1024)
val put=new Put(Bytes.toBytes("row-key-6"))
put.addColumn(Bytes.toBytes("f"),Bytes.toBytes("000132345"),Bytes.toBytes("8"))
table.checkAndPut(Bytes.toBytes("row-key-6"),Bytes.toBytes("f"),Bytes.toBytes("000132345"),org.apache.hadoop.hbase.filter.CompareFilter.CompareOp.NOT_EQUAL,Bytes.toBytes(0), put)


val put2=new Put(Bytes.toBytes("row-key-6"))
put2.addColumn(Bytes.toBytes("f"),Bytes.toBytes("000212345"),Bytes.toBytes(4))
table.checkAndPut(Bytes.toBytes("row-key-6"),Bytes.toBytes("f"),Bytes.toBytes("000212345"),org.apache.hadoop.hbase.filter.CompareFilter.CompareOp.NOT_EQUAL,Bytes.toBytes(3), put2)

val put3=new Put(Bytes.toBytes("row-key-16"))
put3.addColumn(Bytes.toBytes("f"),Bytes.toBytes("000212345"),Bytes.toBytes(3))
table.checkAndPut(Bytes.toBytes("row-key-16"),Bytes.toBytes("f"),Bytes.toBytes("000212345"),org.apache.hadoop.hbase.filter.CompareFilter.CompareOp.EQUAL,null, put3)

val m = connection.getBufferedMutator(TableName.valueOf(tableName))
val puts = new java.util.ArrayList[Put]()
val put=new Put(Bytes.toBytes("row-key-6"))
put.addColumn(Bytes.toBytes("f"),Bytes.toBytes("000132345"),Bytes.toBytes("5"))
puts.add(put)
m.checkAndMutate(Bytes.toBytes("row-key-6"),Bytes.toBytes("f"),null,org.apache.hadoop.hbase.filter.CompareFilter.CompareOp.NOT_EQUAL,Bytes.toBytes(3),puts)



```


## hbase python  --http://hbase.apache.org/
*   sudo pip install thrift
*   sudo pip install hbase-thrift
*   
```
from thrift import Thrift
from thrift.transport import TSocket
from thrift.transport import TTransport
from thrift.protocol import TBinaryProtocol

from hbase import Hbase
from hbase.ttypes import *

transport = TSocket.TSocket('10.10.135.104', 9090) 
transport = TTransport.TBufferedTransport(transport)
protocol = TBinaryProtocol.TBinaryProtocol(transport)

client = Hbase.Client(protocol)
transport.open()
client.getTableNames()

tableName = 'test:mt2'
rowKey = 'a123-1'
 
result = client.getRow(tableName, rowKey)
for r in result:
    print 'the row is ', r.row
    print 'the values is ', r.columns.get('f:20141224').value



```

###  happybase-- https://github.com/wbolster/happybase/blob/master/doc/user.rst
*   https://www.jianshu.com/p/d2a40f8fd4f6
*   pip install happybase

```
import happybase
pool = happybase.ConnectionPool(size=3, host='10.10.135.104', port=9090, table_prefix=b'test', table_prefix_separator=b':')

with pool.connection() as connection:
    print(connection.tables())

with pool.connection() as connection:
    table = connection.table('mt2')
    row = table.row(b'a123-1')

with pool.connection() as connection:
    table = connection.table('mt2')
    for key, data in table.scan(row_start=b'b223-', row_stop=b'b223.', filter=b'SKIP ValueFilter(>=,'binary:0') ' , limit=6):
        print(key, data)

with pool.connection() as connection:
    table = connection.table('mt2')
    with table.batch() as b:
        b.put(b'row-key-1', {b'f:col1': b'1'})
        b.put(b'row-key-2', {b'f:col1': b'2'})
        b.put(b'row-key-3', {b'f:col1': b'3'})
        b.put(b'row-key-4', {b'f:col1': b'4'})
        b.put(b'row-key-5', {b'f:col1': b'5'})
    

```   


## hbase 原理
### scan http://forum.huawei.com/enterprise/zh/thread-327647-1-1.html

## 问题
###   ValueFilter 过滤出了所有匹配的包括旧的value
*   FirstKeyOnlyFilter() 放在最前面

### spark 接入hbase 使用 hbase-spark库，用了1.6.0 spark 和scala 2.10编译，线上环境2.0.1 -2.11 spark编译不过
*   线上环境hbase用的<artifactId>hbase-spark</artifactId><version>1.2.0-cdh5.8.0</version> 也是基于1.6.0 和2.10的，但是直接运行spark shell可以使用
*   开发环境编译打包的时候报错：compiled against an incompatible version ， org.apache.spark

### spark 批量写入慢 ,Spark写入HBase（Bulk方式）
* gc时间长

### hBaseContext.bulkGet ==> collectAsMap/collect
*    ERROR ResourceLeakDetector: LEAK: ByteBuf.release() was not called before it's garbage-collected. Enable advanced leak reporting to find out where the leak occurred.


## md5 - python
```
import hashlib

data =  'This a md5 test!'
hash_md5 = hashlib.md5(data)

hash_md5.hexdigest() # 按16位输出
```
## md5 - java-scala 
```
import java.security.MessageDigest
val digest = MessageDigest.getInstance("MD5")

val text = "MD5 this text!"
digest.digest(text.getBytes).map("%02x".format(_)).mkString

```

## 应用推荐结果保存
### 目标
*   通过Userid + 日期可以获取前多少个推荐postid
*   按推荐结果权重排序
*   派发postid更新推荐结果，下次过滤不被查询出来
*   重复计算的推荐结果已经被派发更新过的也不能被查询
*   重复计算的新的结果能优先排序

#### rowkey
*   userId
*   postId
*   统一批次权重
*   时间批次

## 问题点
*   拉取的时候要按批次+权重排序， 
*   派发更新的时候要能定位到 对应的userid+postid
    -   scan的时候保存key,派发直接通过key删除
*   重新计算更新 相同postid 在同一个地方 
    -   比较已经存在的postid,重复的不写入，保证一个user+posti只有一条记录
        +   读取一次
        +   比较排除


## yichang
2018-04-11 09:32:31.008  WARN 22225 --- [le-pool1-t18935] o.a.h.hbase.util.DynamicClassLoader      : Failed to check remote dir status /tmp/hbase-root/hbase/lib

java.io.FileNotFoundException: File /tmp/hbase-root/hbase/lib does not exist
  at org.apache.hadoop.fs.RawLocalFileSystem.listStatus(RawLocalFileSystem.java:376) ~[hadoop-common-2.6.0.jar:na]
  at org.apache.hadoop.fs.FileSystem.listStatus(FileSystem.java:1485) ~[hadoop-common-2.6.0.jar:na]
  at org.apache.hadoop.fs.FileSystem.listStatus(FileSystem.java:1525) ~[hadoop-common-2.6.0.jar:na]
  at org.apache.hadoop.fs.ChecksumFileSystem.listStatus(ChecksumFileSystem.java:570) ~[hadoop-common-2.6.0.jar:na]
  at org.apache.hadoop.hbase.util.DynamicClassLoader.loadNewJars(DynamicClassLoader.java:203) [hbase-common-1.2.0-cdh5.8.0.jar:na]
  at org.apache.hadoop.hbase.util.DynamicClassLoader.tryRefreshClass(DynamicClassLoader.java:168) [hbase-common-1.2.0-cdh5.8.0.jar:na]
  at org.apache.hadoop.hbase.util.DynamicClassLoader.loadClass(DynamicClassLoader.java:140) [hbase-common-1.2.0-cdh5.8.0.jar:na]
  at java.lang.Class.forName0(Native Method) [na:1.8.0_144]
  at java.lang.Class.forName(Class.java:348) [na:1.8.0_144]
  at org.apache.hadoop.hbase.protobuf.ProtobufUtil.toException(ProtobufUtil.java:1545) [hbase-client-1.2.0-cdh5.8.0.jar:na]
  at org.apache.hadoop.hbase.protobuf.ResponseConverter.getResults(ResponseConverter.java:125) [hbase-client-1.2.0-cdh5.8.0.jar:na]
  at org.apache.hadoop.hbase.client.MultiServerCallable.call(MultiServerCallable.java:133) [hbase-client-1.2.0-cdh5.8.0.jar:na]
  at org.apache.hadoop.hbase.client.MultiServerCallable.call(MultiServerCallable.java:53) [hbase-client-1.2.0-cdh5.8.0.jar:na]
  at org.apache.hadoop.hbase.client.RpcRetryingCaller.callWithoutRetries(RpcRetryingCaller.java:200) [hbase-client-1.2.0-cdh5.8.0.jar:na]
  at org.apache.hadoop.hbase.client.AsyncProcess$AsyncRequestFutureImpl$SingleServerRequestRunnable.run(AsyncProcess.java:733) [hbase-client-1.2.0-cdh5.8.0.jar:na]
  at java.util.concurrent.Executors$RunnableAdapter.call(Executors.java:511) [na:1.8.0_144]
  at java.util.concurrent.FutureTask.run(FutureTask.java:266) [na:1.8.0_144]
  at java.util.concurrent.ThreadPoolExecutor.runWorker(ThreadPoolExecutor.java:1149) [na:1.8.0_144]
  at java.util.concurrent.ThreadPoolExecutor$Worker.run(ThreadPoolExecutor.java:624) [na:1.8.0_144]
  at java.lang.Thread.run(Thread.java:748) [na:1.8.0_144]


## 参考
*   http://0b4af6cdc2f0c5998459-c0245c5c937c5dedcca3f1764ecc9b2f.r43.cf2.rackcdn.com/9353-login1210_khurana.pdf
*   https://strongyoung.gitbooks.io/hbase-reference-guide/content/data_model.html
