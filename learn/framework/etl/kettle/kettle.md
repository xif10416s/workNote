##  kettle.md

## KETTLE集群搭建  --  http://www.kettle.net.cn/1601.html
*	Kettle集群是由一个主carte服务器和多个从carte服务器组成的，类似于master-slave结构，不同的是’master’处理具体任务，只负责任务的分发和收集运行结果。
*	Master carte结点收到请求后，把任务分成多个部分交给slave carte执行，slave执行完毕后把结果交给mater 进行汇总，再由mster返回结果。

### 集群的优点
*	1)多服务器运行，加快处理速度，对于大数据量的操作更明显
*	2)防单点失败，一台服务器故障后其它服务器还可以运行

### 集群的缺点
*	1)采用主从结构，不具备自动切换主从的功能。所以一旦主节点宕机，整个系统不可用
*	2)对网络要求高，节点之间需要不断的传输数据
*	3)需要更多的服务器，而且主节点没有处理能力


### 适用场景
*	1)需求kettle能时刻保持正常运行的场景
*	2)大批量处理数据的场景

###  集群架构
*	https://anonymousbi.wordpress.com/2012/07/31/pdi-clusters-part-1-how-to-build-a-simple-pdi-cluster/

http://ddrv.cn/a/76491


## kettle 转换机制


## kettle 使用 https://edu.51cto.com/center/course/lesson/index?id=82415



