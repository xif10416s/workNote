#	hive 权限管理.md
## HIVE的权限控制和超级管理员的实现  -- 误操作
*	一、在Cloudera Manager中配置
hive-site.xml 的 Hive 客户端高级配置代码段（安全阀）
hive-site.xml 的 HiveServer2 高级配置代码段（安全阀）
https://blog.csdn.net/zqqnancy/article/details/51852794
```
<property>
<name>hive.users.in.admin.role</name>
<value>hive</value>
<description>定义超级管理员 启动的时候会自动创建Comma separated list of users who are in admin role for bootstrapping.
More users can be added in ADMIN role later.</description>
</property>
<property>
<name>hive.metastore.authorization.storage.checks</name>
<value>true</value>
</property>
<property>
<name>hive.metastore.execute.setugi</name>
<value>false</value>
</property>
<property>
<name>hive.security.authorization.enabled</name>
<value>true</value>
<description>开启权限 enable or disable thehive client authorization</description>
</property>
<property>
<name>hive.security.authorization.createtable.owner.grants</name>
<value>ALL</value>
<description>表的创建者对表拥有所有权限the privileges automaticallygranted to the owner whenever a table gets created. An example like"select,drop" will grant select and drop privilege to the owner ofthe table</description>
</property>
<property>
<name>hive.security.authorization.task.factory</name>
<value>org.apache.hadoop.hive.ql.parse.authorization.HiveAuthorizationTaskFactoryImpl</value>
<description>进行权限控制的配置。</description>
</property>
<property>
<name>hive.semantic.analyzer.hook</name>
<value>com.hive.HiveAdmin</value>
<description>使用钩子程序，识别超级管理员，进行授权控制。</description>
</property>
<property>



dfs.replication|hive.input.format..*|tez.queue.name| mapreduce..*|mapreduce.job.split.metainfo.maxsize|hive.security.authorization.sqlstd.confwhitelist.append


hive.security.authorization.sqlstd.confwhitelist.append

## hadoop 添加组 --https://www.cnblogs.com/zlslch/p/5842961.html
*	 groupadd   readonly
*	useradd  -m -g readonly test02

## cdh
*	https://blog.csdn.net/qq_30982323/article/details/80704720
*	sentry -- Hue和Sentry集成后，就要求用户组同时在Hue和OS里都要创建。
	*	https://baijiahao.baidu.com/s?id=1617556915401338251&wfr=spider&for=pc
	*	https://blog.csdn.net/qq_30982323/article/details/80704720
	*	https://www.cloudera.com/documentation/enterprise/latest/topics/sg_sentry_service_config.html#xd_583c10bfdbd326ba--69adf108-1492ec0ce48--7dbd
*	beeline hive ,hive连接 , 添加角色
	*	https://www.cnblogs.com/xiashiwendao/p/9173362.html
*	权限配置
	*	https://blog.csdn.net/woloqun/article/details/83585887
	*	create role db01_role;
	*	GRANT ROLE db01_role TO GROUP hue;
	*	 create database db01;
	*	GRANT ALL ON DATABASE db01 TO ROLE db01_role;
	*	GRANT SELECT ON table test TO ROLE db01_role;

```
create role db01_role1;
GRANT ROLE db01_role1 TO GROUP readonly;
GRANT ALL ON DATABASE default TO ROLE db01_role1;
GRANT ALL ON DATABASE db01 TO ROLE db01_role1;
GRANT ALL ON table default.test TO ROLE db01_role1;


SHOW ROLE GRANT GROUP readonly;


```	


```
*	beeline -u "jdbc:hive2://172.20.100.120:10000 -n admin -p admin"
*	!connect jdbc:hive2://172.20.100.120:10000/default
*	创建角色
	*	CREATE ROLE test;



```
CREATE  TABLE test (  
      id int)
    ROW FORMAT DELIMITED FIELDS TERMINATED BY ',' ;


```	

##  Kerberos认证
*	http://www.databps.com/gsdt/169.html
*	https://makeling.github.io/bigdata/39395030.html


## 权限测试
*	hive 
*	set role=admin;


## TODO
*	hive权限体系
*	cdh hive 权限 sentry
*	hue 权限功能，
*	hue用户 + hadoop用户（组）+sentry中用的组的关系