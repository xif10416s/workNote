#Spark Cassandra

→代表一个cassandra table
→需要连接cassandra，所以需要配置连接信息
→把数据切分成小的分区，在各个集群本地处理，每个分区由一个或者多个不连续的Token range
→每个分区是批处理读取，配置参数，split.size和page.row.size
→通过getpreferedlocations判断Partition分发到哪个集群节点计算