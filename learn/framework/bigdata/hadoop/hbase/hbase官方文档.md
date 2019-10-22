##   [hbase官方文档](https://hbase.apache.org/apache_hbase_reference_guide.pdf) - Version 3.0.0-SNAPSHOT

#### 启动组件
*	QuorumPeerMain
	*	zookeeper集群的启动入口类，是用来加载配置启动QuorumPeer线程的
	*	首先启动hadoop，然后问题就来了：zookeeper和hbase的启动顺序是什么？
		*	先启动hbase:hbase有内置的zookeeper，如果没有装zookeeper，启动hbase的时候会有一个HQuorumPeer进程。
			*	HQuorumPeer表示hbase管理的zookeeper
		*	先启动zookeeper：如果用外置的zookeeper管理hbase，则先启动zookeeper，然后启动hbase，启动后会有一个QuorumPeerMain进程。
			*   QuorumPeerMain表示zookeeper独立的进程
*	HRegionServer


#### 基础配置
*	Managed Splitting  -- hbase-default.xml ， hbase-site.xml
	*	hbase.regionserver.region.split.policy
	*	hbase.hregion.max.filesize,
	*	hbase.regionserver.regionSplitLimit