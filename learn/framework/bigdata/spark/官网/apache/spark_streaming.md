#Spark Streaming
*   通过网络接受的数据默认保存2个节点,容错
*   把计算结果写入外部存储
*   问题一:
connection不可能被序列化传递
```
dstream.foreachRDD { rdd =>
 val connection = createNewConnection()  // 在driver中被创建
 rdd.foreach { record =>
   connection.send(record) // rdd的操作在worker中执行
 }
}
```
*   问题二:
每个线程创建一个connection,资源浪费
```
dstream.foreachRDD { rdd =>
 rdd.foreach { record =>
   val connection = createNewConnection()
   connection.send(record)
   connection.close()
 }
}
```
*   推荐方式:
每个partion创建一个connection
```
dstream.foreachRDD { rdd =>
 rdd.foreachPartition { partitionOfRecords =>
   val connection = createNewConnection()
   partitionOfRecords.foreach(record => connection.send(record))
   connection.close()
 }
}
```