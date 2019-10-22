# 集群安装
##  cdh 安装 
*   https://blog.csdn.net/u010003835/article/details/85007946
*   CDH6.1.0离线安装部署
    -   https://blog.csdn.net/xsjzdrxsjzdr/article/details/86064094
*   https://www.cloudera.com/documentation/enterprise/5-8-x/topics/installation_installation.html
*   CentOS 7下Cloudera Manager及CDH 6.0.1安装过程详解
    -   http://blog.51cto.com/wzlinux/2321433

## cdh 版本与hadoop 版本对应关系
*   https://www.cloudera.com/documentation/enterprise/6/release-notes/topics/rg_cdh_60_packaging.html

## 6.1文档
*   https://www.cloudera.com/documentation/enterprise/6/6.1.html


##  centos 源
*   https://www.cnblogs.com/focusonepoint/p/7120861.html

## 具体步骤
### centos 7.6 准备及基础配置 ssh + jdk == root / root
*	数据集创建
	*	docker volume rm  cdh_name
	*	docker volume rm  cdh_data1
	*	docker volume rm  cdh_data2
	*	docker volume create cdh_name_var,docker volume create cdh_name_etc,docker volume create cdh_name_opt
		*	docker volume rm cdh_name_var  cdh_name_etc  cdh_name_opt
	*	docker volume create cdh_data1
	*	docker volume create cdh_data2
*	创建网卡 - 固定ip
	*	 docker network create -o "com.docker.network.bridge.name"="test01" --subnet 172.20.0.0/16 test01	
*	直接拉取改造
	*	docker pull bayern0815/centos7.6-jdk1.8-ssh:0.0.1
*	拉去官网centos7.6 	
*	docker run -it --privileged=true -p 18001:22 1e1148e4cc2c  /usr/sbin/init
*	docker run -it  --ip 172.20.100.120 --net test01 -h="master" --name master  -d --privileged=true  -v cdh_name_var:/var -v cdh_name_etc:/etc -v cdh_name_opt:/opt   -p 28001:22 -p 28002:7180  -p 28080:8080 -p 28070:7070 -p 28081:8081 -p 28040:4040 -p 28057:50070 -p 28088:8088 -p 28010:16010 -p 28006:60010   -p 28100:10000 -p 28888:8888  --add-host=slave1:172.20.100.121 --add-host=slave2:172.20.100.122  centos_cdh6.1:namenode1  /usr/sbin/init sh -c '/etc/rc.local;'
*	docker run -it  --ip 172.20.100.120 --net test01 -h="master" --name master  -d --privileged=true  -v cdh_name_var:/var -v cdh_name_etc:/etc -v cdh_name_opt:/opt   -p 28001:22 -p 28002:7180  -p 28080:8080 -p 28070:7070 -p 28081:8081 -p 28040:4040 -p 28057:50070 -p 28088:8088 -p 28010:16010 -p 28006:60010   -p 28100:10000 -p 28888:8888  --add-host=slave1:172.20.100.121 centos_cdh6.1:namenode1  /usr/sbin/init sh -c '/etc/rc.local;'
*	yum install wget
*	yum install vim
*	源配置 -- https://blog.csdn.net/inslow/article/details/54177191
	*	mv /etc/yum.repos.d/CentOS-Base.repo /etc/yum.repos.d/CentOS-Base.repo.backup
	*	wget -O /etc/yum.repos.d/CentOS-Base.repo http://mirrors.aliyun.com/repo/Centos-7.repo
	*	 cd /etc/yum.repos.d/ & yum makecache
*	ssh安装
	*	yum -y install openssh-server openssh-clients
	*	yum install initscripts
	*	service sshd start
*	yum install net-tools
	*	netstat -antp |grep sshd
*	ssh 自动启动
	*	vim /etc/rc.local
		*	service sshd start
*	passwd修改密码
*	jdk安装
	* 进入centos 在home 目录执行 `
wget --no-cookies \
--no-check-certificate \
--header "Cookie: oraclelicense=accept-securebackup-cookie" \
 https://download.oracle.com/otn-pub/java/jdk/8u201-b09/42970487e3af4f5aa5bca3f542482c60/jdk-8u201-linux-x64.tar.gz \
-O jdk-8-linux-x64.tar.gz
`
	*  或者直接 https://www.oracle.com/technetwork/java/javase/downloads/jdk8-downloads-2133151.html 下载 , 放入docker共享目录
		*	docker-machine ssh default 登录docker vb 虚拟机共享目录
		*	scp jdk-8u201-linux-x64.tar.gz root@172.17.0.2:/home
	*	yum install tar
	*	mkdir /opt/java
	*	mv /home/jdk-8u201-linux-x64.tar.gz /opt/java
	*	tar -zxvf jdk-8u201-linux-x64.tar.gz
	*   ln -s jdk1.8.0_201/ jdk8
	*	环境变量设置：
		*	sudo vim /etc/profile
		*	`export JAVA_HOME=/opt/java/jdk8
			 export PATH=$JAVA_HOME/bin:$PATH
			 export CLASSPATH=.:$JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/tools.jar`
		*   source /etc/profile
		*	java -version
*   安装 MySQL JDBC 驱动(所有节点)
```
wget https://dev.mysql.com/get/Downloads/Connector-J/mysql-connector-java-5.1.46.tar.gz
tar xf mysql-connector-java-5.1.46.tar.gz

mkdir -p /usr/share/java/
cd mysql-connector-java-5.1.46
cp mysql-connector-java-5.1.46-bin.jar /usr/share/java/mysql-connector-java.jar
```	
*  commit 基础image , node 节点可用
	*	docker commit f2a95320beb3   centos_base:ssh-jdk	
	*	测试：docker run -it --privileged=true -p 28001:22 centos_base:ssh-jdk   sh -c   /usr/sbin/init  '/etc/rc.local;'
		*	问题：D:\Docker Toolbox\docker.exe: Error response from daemon: cgroups: cannot find cgroup mount destination: unknown
			*   解决：
			*	docker-machine ssh default
			*	sudo mkdir /sys/fs/cgroup/systemd
			*	sudo mount -t cgroup -o none,name=systemd cgroup /sys/fs/cgroup/systemd

###  在原来container基础上添加mysql 配置namenode
*	https://www.cloudera.com/documentation/enterprise/latest/topics/install_cm_mariadb.html
*	yum  -y  install psmisc MySQL-python at bc bind-libs bind-utils cups-client cups-libs cyrus-sasl-gssapi cyrus-sasl-plain ed fuse fuse-libs httpd httpd-tools keyutils-libs-devel krb5-devel libcom_err-devel libselinux-devel libsepol-devel libverto-devel mailcap noarch mailx mod_ssl openssl-devel pcre-devel postgresql-libs python-psycopg2 redhat-lsb-core redhat-lsb-submod-security  x86_64 spax time zlib-devel
*	sudo yum install mariadb-server
*	编辑配置文件/etc/my.cnf
```
https://www.cloudera.com/documentation/enterprise/latest/topics/install_cm_mariadb.html

innodb_buffer_pool_size = 4G  ==>128m
innodb_thread_concurrency = 8
innodb_flush_method = O_DIRECT
#innodb_log_file_size = 512M    ==32m
```
*	启动数据库
	*	systemctl enable mariadb
	*	systemctl start mariadb
	*	vim /etc/rc.local 添加启动命令
*	初始化数据库
	*	 /usr/bin/mysql_secure_installation
		*	问题：ERROR 1045 (28000): Access denied for user 'root'@'localhost' (using password: YES)
			*	/etc/my.cnf 配置 [mysqld]下 添加 skip-grant-tables
		*	出问题看日志： tail -n500 /var/log/mariadb/mariadb.log
		*	问题：Error: log file ./ib_logfile0 is of different size 0 5242880 bytes
			*	# innodb_log_file_size  = 256M 注释

###  使用root登陆数据库，创建以下数据库和账号。
*	mysql -u root -p
*   flush privileges;
*	初始化数据 https://www.cloudera.com/documentation/enterprise/latest/topics/install_cm_mariadb.html ，http://blog.51cto.com/wzlinux/2321433
 
###  cdh manager 安装
#### 安装 CM Server 和 Agent
*	scp cloudera-manager.repo root@172.17.0.2:/home
*	mkdir /opt/cloudera-manager/
*	cp cloudera-manager.repo /etc/yum.repos.d/
*	rpm --import https://archive.cloudera.com/cm6/6.1.0/redhat7/yum/RPM-GPG-KEY-cloudera
*   yum install cloudera-manager-daemons cloudera-manager-agent cloudera-manager-server
*	scp cloudera-manager-agent-6.1.0-769885.el7.x86_64.rpm root@172.17.0.2:/home
*	scp cloudera-manager-daemons-6.1.0-769885.el7.x86_64.rpm root@172.17.0.2:/home
*	scp cloudera-manager-server-6.1.0-769885.el7.x86_64.rpm root@172.17.0.2:/home
*   rpm -ivh   cloudera-manager-daemons-6.1.0-769885.el7.x86_64.rpm
*   rpm -ivh   cloudera-manager-agent-6.1.0-769885.el7.x86_64.rpm
*   rpm -ivh   cloudera-manager-server-6.1.0-769885.el7.x86_64.rpm

#### 设置 Cloudera Manager 数据库
*	 /opt/cloudera/cm/schema/scm_prepare_database.sh mysql scm scm

### 配置CDH的软件包 parcels(namenode01)
*	scp CDH-6.1.0-1.cdh6.1.0.p0.770702-el7.parcel root@172.17.0.2:/opt/cloudera/parcel-repo
*	scp CDH-6.1.0-1.cdh6.1.0.p0.770702-el7.parcel.sha256 root@172.17.0.2:/opt/cloudera/parcel-repo
*	scp manifest.json root@172.17.0.2:/opt/cloudera/parcel-repo
*	重要，不然选择不到版本，接下来打开manifest.json文件，里面是json格式的配置，我们需要的就是与我们系统版本相对应的 hash码，
	*       "hash": "ebe32d2d9c20cb7eab778b81d234f8802591ac97",
            "parcelName": "CDH-6.1.0-1.cdh6.1.0.p0.770702-el7.parcel",
            "replaces": "IMPALA, SOLR, SPARK, KAFKA, IMPALA_KUDU, KUDU, SPARK2"
	*	echo "ebe32d2d9c20cb7eab778b81d234f8802591ac97" > CDH-6.1.0-1.cdh6.1.0.p0.770702-el7.parcel.sha
	*	chown cloudera-scm.cloudera-scm /opt/cloudera/parcel-repo/*

#### docker commit f2a95320beb3   centos_cdh6.1:namenode	

### 启动 cm
*	vim /etc/cloudera-scm-agent/config.ini
	*	server_host 修改
*	systemctl start cloudera-scm-server
*	systemctl status cloudera-scm-server
*	tail -f /var/log/cloudera-scm-server/cloudera-scm-server.log
*	问题： Loaded: loaded (/usr/lib/systemd/system/cloudera-scm-server.service; enabled; vendor preset: disabled)
	*	journalctl -n 20 查看service启动日志 Cloudera Manager requires Oracle JDK 1.8 or later.
	*	journalctl -f
*	scp oracle-j2sdk1.8-1.8.0+update141-1.x86_64.rpm root@172.17.0.2:/home
*	rpm -ivh  oracle-j2sdk1.8-1.8.0+update141-1.x86_64.rpm

## slave1-3
*	docker datanode 准备 ，在基础ssh+jdk上
	*	docker run -it --privileged=true  centos_base:ssh-jdk  /usr/sbin/init
	*	安装 MySQL JDBC 驱动(所有节点) --参考master
	*	cdh源配置
		*	mkdir /opt/cloudera-manager/
		*	cp cloudera-manager.repo /etc/yum.repos.d/
		*   yum -y install  cyrus-sasl-gssapi      bind-utils psmisc     libxslt   cyrus-sasl-plain  openssl cyrus-sasl-gssap  fuse  portmap    fuse-libs /lib/lsb/init-functions   httpd   mod_ssl   openssl-devel   python-psycopg2     MySQL-python     libpq.so.5
		*   rpm -ivh   cloudera-manager-daemons-6.1.0-769885.el7.x86_64.rpm
		*   rpm -ivh   cloudera-manager-agent-6.1.0-769885.el7.x86_64.rpm
		*	vim /etc/cloudera-scm-agent/config.ini 修改server_host
		*	jdk安装
			*	docker-machine ssh default
			*	scp oracle-j2sdk1.8-1.8.0+update141-1.x86_64.rpm root@172.17.0.2:/home
			*	rpm -ivh  oracle-j2sdk1.8-1.8.0+update141-1.x86_64.rpm
	*	docker commit e13154d4ff26     centos_cdh6.1:datanode
*	docker node 启动
	*	docker run -it --ip 172.20.100.121 --net test01  -h="slave1" --name slave1  -d --privileged=true  -v cdh_data1:/var --add-host=master:172.20.100.120 centos_cdh6.1:datanode   /usr/sbin/init sh -c '/etc/rc.local;'
	*	docker run -it --ip 172.20.100.122 --net test01  -h="slave2" --name slave2  -d --privileged=true  -v cdh_data2:/var --add-host=master:172.20.100.120 centos_cdh6.1:datanode   /usr/sbin/init sh -c '/etc/rc.local;'
	*	docker run -it  -h="slave3" --name slave3  -d --privileged=true  -v cdh_data3:/data  --add-host=master:172.17.0.2 centos_cdh6.1:datanode   /usr/sbin/init sh -c '/etc/rc.local;'
	*	hosts修改
```
172.17.0.2      master
172.17.0.3      slave1
172.17.0.4      slave2
172.17.0.5      slave3
```	

## 通过网页安装
*	选择CDH版本这里会显示你放在/opt/cloudera/parcel-repo/下的parcel包，若未显示
*	org.apache.hadoop.hdfs.server.namenode.SafeModeException: Cannot create directory /tmp. Name node is in safe mode.*
```
HDFS 根目录 /data/hbase

DataNode 数据目录 /data/dfs/dn

NameNode 数据目录 /data/dfs/nn

HDFS 检查点目录 /data/dfs/snn

Hive 仓库目录 /data/user/hive/warehouse

ShareLib 根目录 /data/user/oozie

NodeManager 本地目录 /data/yarn/nm
```

## 端口页面
*	cdh 安装页面：http://192.168.99.100:28002/cmf/login  
	*	admin/admin
	*	CDH 版本:CDH-6.1.0-1.cdh6.1.0.p0.770702
	*	数据库
		*	https://www.cloudera.com/documentation/enterprise/latest/topics/install_cm_mariadb.html
		*	密码：root
		*	metastore  hive root
		*	amon,amon,root
		*	oozie,oozie,root
		*	hue,hue,root


### 运行测试
*	spark-shell:
	*	 Permission denied: user=root, access=WRITE, inode="/user":hdfs:supergroup:drwxr-xr-x
		*	usermod -s /bin/bash hdfs
	*	用hdfs账号登录

    
### hive 测试
*	Given NMToken for application : appattempt_1556161534641_0003_000002 is not valid for current node manager.expected : slave2:8041 found : slave1:8041 
	*	ip改变了，对不上
```
CREATE  TABLE test (   id int) ROW FORMAT DELIMITED FIELDS TERMINATED BY ','  ;

insert into test values(1);
```


 *   docker-machine ssh default
    *   sudo mkdir /sys/fs/cgroup/systemd
        sudo mount -t cgroup -o none,name=systemd cgroup /sys/fs/cgroup/systemd