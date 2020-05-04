## redis 安装

#### centos 7
*	 yum -y install gcc-c++
*	wget http://download.redis.io/releases/redis-5.0.9.tar.gz
*	tar -zxvf redis-5.0.9.tar.gz
*	make & make install
*	cp src/redis-server /usr/local/bin
*	 cp src/redis-cli /usr/local/bin
*	新建配置目录 ，拷贝配置文件
	*	redis-server redis.conf
	*	redis-cli -a root
		*	auth root


#### redis 常用配置
*	daemonize yes  -- 后台启动
	*	默认关闭
	*	后台启动会在 /var/run/redis.pid创建pid文件
*	dir  -- 工作目录
	*	 rdb文件 ，aop文件在此创建
*	bind -- 绑定端口，默认本地ip
*	requirepass -- 要求密码 -- root
*	rename-command -- 重命名key , ""为删除
*	maxmemory <bytes> //设置内存的可用大小
*	maxmemory-policy noeviction --默认，缓存不过期，满了无法写入

#### redis 主从读写分离  
*	rdb 数据复制，同步
  *	数据冗余备份
  *	故障恢复
  *	负载均衡，提高读取性能
*	修改从机redis.conf
	*	replicaof 127.0.0.1 6379
	*	masterauth root
	*	replica-read-only yes 从机只读
	*	replica-serve-stale-data yes 主从复制过程中，从服务器是否可以响应客户端请求

#### redis 高可用故障转移哨兵模式  

1. **哨兵节点：** 哨兵系统由一个或多个哨兵节点组成，哨兵节点是特殊的 Redis 节点，不存储数据；
2. **数据节点：** 主节点和从节点都是数据节点；

在复制的基础上，哨兵实现了 **自动化的故障恢复** 功能，下方是官方对于哨兵功能的描述：

*	集群监控： 哨兵会不断地检查主节点和从节点是否运作正常。
*	故障转移：当 **主节点** 不能正常工作时，哨兵会开始 **自动故障转移操作**，它会将其中一个 **从节点升级为新的主节点**，并让其他从节点改为复制新的主节点。
*	消息通知： 哨兵可以将故障转移的结果发送给客户端。
*	配置中心：客户端在初始化时，通过连接哨兵来获得当前 Redis 服务的主节点地址。

#### 高可用集群模式：

* redis集群是一个由多个主从节点群组成的分布式服务器群，它具有复制、高可用和分片特性。
* Redis集群不需要sentinel哨兵也能完成节点移除和故障转移的功能。
* 这种集群模式没有中心节点，可水平扩展，
* redis集群的性能和高可用性均优于之前版本的哨兵模式，且集群配置非常简单。

安装步骤

```
一，修改配置文件并启动
然后按照上面的方法，创建六个配置文件，分别命名为：redis_7000.conf/redis_7001.conf.....redis_7005.conf，然后根据不同的端口号修改对应的端口值就好了：

# 后台执行
daemonize yes
# 端口号
port 7000
# 为每一个集群节点指定一个 pid_file
pidfile ~/Desktop/redis-cluster/redis_7000.pid
# 启动集群模式
cluster-enabled yes
# 每一个集群节点都有一个配置文件，这个文件是不能手动编辑的。确保每一个集群节点的配置文件不通
cluster-config-file nodes-7000.conf
# 集群节点的超时时间，单位：ms，超时后集群会认为该节点失败
cluster-node-timeout 5000
# 最后将 appendonly 改成 yes(AOF 持久化)
appendonly yes

二，创建集群
redis-cli --cluster create --cluster-replicas 1 127.0.0.1:7000 127.0.0.1:7001 127.0.0.1:7002 127.0.0.1:7003 127.0.0.1:7004 127.0.0.1:7005
```





#### 参考

* https://gitee.com/SnailClimb/JavaGuide/blob/master/docs/database/Redis/redis-collection/Redis(9)%E2%80%94%E2%80%94%E9%9B%86%E7%BE%A4%E5%85%A5%E9%97%A8%E5%AE%9E%E8%B7%B5%E6%95%99%E7%A8%8B.md
* https://www.jianshu.com/p/8045b92fafb2