# 集群安装(cdh  6.2.1)
##  cdh 安装
*   https://blog.csdn.net/u010003835/article/details/85007946
*   CDH6.1.0离线安装部署
    -   https://blog.csdn.net/xsjzdrxsjzdr/article/details/86064094
*   https://www.cloudera.com/documentation/enterprise/5-8-x/topics/installation_installation.html
*   CentOS 7下Cloudera Manager及CDH 6.0.1安装过程详解
    -   http://blog.51cto.com/wzlinux/2321433

## cdh 版本与hadoop 版本对应关系
*   https://www.cloudera.com/documentation/enterprise/6/release-notes/topics/rg_cdh_60_packaging.html
*	https://docs.cloudera.com/documentation/enterprise/6/release-notes/topics/rg_cdh_621_fixed_issues.html#fixed_issues
*	https://docs.cloudera.com/documentation/spark2/latest/topics/spark2_fixed_issues.html#fixed_issues_240_r2

## 6.1文档
*   https://www.cloudera.com/documentation/enterprise/6/6.1.html


##  centos 源
*   https://www.cnblogs.com/focusonepoint/p/7120861.html

## 服务器
*	viya02  master
*	viya01  master2
*   dm-inc-fin-1  data1
*	dm-inc-fin-2 data2
*  forecastedatamind data3

## 具体步骤

###  CDH离线包准备
*	Cloudera Manager	https://archive.cloudera.com/cm6/6.2.1/redhat7/yum/
	*	https://archive.cloudera.com/cm6/6.2.1/redhat7/yum/cloudera-manager.repo
	*	https://archive.cloudera.com/cm6/6.2.1/redhat7/yum/RPMS/x86_64/
		*	agent,daemons,server,oracle-j2sdk1.8
*	CDH  https://archive.cloudera.com/cdh6/6.2.1/parcels/
	*   CDH-6.2.1-1.cdh6.2.1.p0.1425774-el7.parcel
	*	CDH-6.2.1-1.cdh6.2.1.p0.1425774-el7.parcel.sha256
	*	manifest.json

###  配置host (所有节点)
*	vim /etc/hosts

###  ssh 免密登录（两个name节点）
```
ssh-keygen -trsa
cat ~/.ssh/id_rsa.pub >> ~/.ssh/authorized_keys
chmod 600 ~/.ssh/authorized_keys

scp ~/.ssh/authorized_keys root@dm-inc-fin-1:~/.ssh/
scp ~/.ssh/authorized_keys root@dm-inc-fin-2:~/.ssh/
scp ~/.ssh/authorized_keys root@forecastdatamind:~/.ssh/
```


### centos 7.3 准备及基础配置 jdk （所有节点）
*	源配置 -- https://blog.csdn.net/inslow/article/details/54177191
	*	mv /etc/yum.repos.d/CentOS-Base.repo /etc/yum.repos.d/CentOS-Base.repo.backup
	*	wget -O /etc/yum.repos.d/CentOS-Base.repo http://mirrors.aliyun.com/repo/Centos-7.repo
	*	 cd /etc/yum.repos.d/ & yum makecache
*	jdk安装
	* 进入centos 在home 目录执行 `
wget --no-cookies \
--no-check-certificate \
--header "Cookie: oraclelicense=accept-securebackup-cookie" \
 https://download.oracle.com/otn-pub/java/jdk/8u201-b09/42970487e3af4f5aa5bca3f542482c60/jdk-8u201-linux-x64.tar.gz \
-O jdk-8-linux-x64.tar.gz
`
	*  或者直接 https://www.oracle.com/technetwork/java/javase/downloads/jdk8-downloads-2133151.html 下载 , 放入docker共享目录
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


###   安装 MySQL JDBC 驱动(所有节点)
```
wget https://dev.mysql.com/get/Downloads/Connector-J/mysql-connector-java-5.1.46.tar.gz
tar xf mysql-connector-java-5.1.46.tar.gz

mkdir -p /usr/share/java/
cd mysql-connector-java-5.1.46
cp mysql-connector-java-5.1.46-bin.jar /usr/share/java/mysql-connector-java.jar
```


###  添加mysql 配置(namenode节点）
*	https://www.cloudera.com/documentation/enterprise/latest/topics/install_cm_mariadb.html
*	yum  -y  install psmisc MySQL-python at bc bind-libs bind-utils cups-client cups-libs cyrus-sasl-gssapi cyrus-sasl-plain ed fuse fuse-libs httpd httpd-tools keyutils-libs-devel krb5-devel libcom_err-devel libselinux-devel libsepol-devel libverto-devel mailcap noarch mailx mod_ssl openssl-devel pcre-devel postgresql-libs python-psycopg2 redhat-lsb-core redhat-lsb-submod-security  x86_64 spax time zlib-devel
*	sudo yum install mariadb-server
*	vim /etc/my.cnf
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
	*	账号密码：root/root123456

###  使用root登陆数据库，创建以下数据库和账号。
*	mysql -u root -p
*   flush privileges;
*	初始化数据 https://www.cloudera.com/documentation/enterprise/latest/topics/install_cm_mariadb.html ，http://blog.51cto.com/wzlinux/2321433
```
CREATE DATABASE scm DEFAULT CHARACTER SET utf8 DEFAULT COLLATE utf8_general_ci;
GRANT ALL ON scm.* TO 'scm'@'%' IDENTIFIED BY '!dtmind&123';
CREATE DATABASE amon DEFAULT CHARACTER SET utf8 DEFAULT COLLATE utf8_general_ci;
GRANT ALL ON amon.* TO 'amon'@'%' IDENTIFIED BY '!dtmind&123';
CREATE DATABASE rman DEFAULT CHARACTER SET utf8 DEFAULT COLLATE utf8_general_ci;
GRANT ALL ON rman.* TO 'rman'@'%' IDENTIFIED BY '!dtmind&123';
CREATE DATABASE hue DEFAULT CHARACTER SET utf8 DEFAULT COLLATE utf8_general_ci;
GRANT ALL ON hue.* TO 'hue'@'%' IDENTIFIED BY '!dtmind&123';
CREATE DATABASE metastore DEFAULT CHARACTER SET utf8 DEFAULT COLLATE utf8_general_ci;
GRANT ALL ON metastore.* TO 'hive'@'%' IDENTIFIED BY '!dtmind&123';
CREATE DATABASE sentry DEFAULT CHARACTER SET utf8 DEFAULT COLLATE utf8_general_ci;
GRANT ALL ON sentry.* TO 'sentry'@'%' IDENTIFIED BY '!dtmind&123';
CREATE DATABASE nav DEFAULT CHARACTER SET utf8 DEFAULT COLLATE utf8_general_ci;
GRANT ALL ON nav.* TO 'nav'@'%' IDENTIFIED BY '!dtmind&123';
CREATE DATABASE navms DEFAULT CHARACTER SET utf8 DEFAULT COLLATE utf8_general_ci;
GRANT ALL ON navms.* TO 'navms'@'%' IDENTIFIED BY '!dtmind&123';
CREATE DATABASE oozie DEFAULT CHARACTER SET utf8 DEFAULT COLLATE utf8_general_ci;
GRANT ALL ON oozie.* TO 'oozie'@'%' IDENTIFIED BY '!dtmind&123';

flush privileges;
```

###  cdh manager 安装
###  文件拷贝
*	namenode : /home/datamind/cdh
	*	全部文件
*	datanode: /home/datamind/cdh
	*	cloudera-manager-agent,cloudera-manager-daemons,cloudera-manager.repo (其他节点)
	*	scp cloudera-manager-daemons-6.2.1-1426065.el7.x86_64.rpm root@viya01:/home/datamind/cdh
	*	scp cloudera-manager-agent-6.2.1-1426065.el7.x86_64.rpm root@viya01:/home/datamind/cdh
	*	scp cloudera-manager.repo root@viya01:/home/datamind/cdh


#### 安装 CM Server 和 Agent 
#####  namenode
*	mkdir /opt/cloudera-manager/
*	cp cloudera-manager.repo /etc/yum.repos.d/
*	yum search cloudera-manager-daemons cloudera-manager-agent cloudera-manager-server
*	rpm --import https://archive.cloudera.com/cm6/6.2.1/redhat7/yum/RPM-GPG-KEY-cloudera
*   //yum install cloudera-manager-daemons cloudera-manager-agent cloudera-manager-server
*	 yum -y install  cyrus-sasl-gssapi      bind-utils psmisc     libxslt   cyrus-sasl-plain  openssl cyrus-sasl-gssap  fuse  portmap    fuse-libs /lib/lsb/init-functions   httpd   mod_ssl   openssl-devel   python-psycopg2     MySQL-python     libpq.so.5
*   rpm -ivh   cloudera-manager-daemons
*   rpm -ivh   cloudera-manager-agent
*   rpm -ivh   cloudera-manager-server

#### 设置 Cloudera Manager 数据库
*	 /opt/cloudera/cm/schema/scm_prepare_database.sh mysql scm scm
	*	!dtmind&123

### 配置CDH的软件包 parcels(namenode01)
*	拷贝 CDH-6.2.1-el7.parcel ==》 /opt/cloudera/parcel-repo
*	//拷贝 CDH-6.2.1-el7.parcel.sha256 ==》  /opt/cloudera/parcel-repo
*	拷贝 manifest.json  ==》  /opt/cloudera/parcel-repo
*	mv CDH-6.2.1-1.cdh6.2.1.p0.1425774-el7.parcel  manifest.json /opt/cloudera/parcel-repo/
*	重要，不然选择不到版本，接下来打开manifest.json文件，里面是json格式的配置，我们需要的就是与我们系统版本相对应的 hash码，
	*      "hash": "e3b31081a200ee9ee10aa64e48a0a3d62b25a7b1",
            "parcelName": "CDH-6.2.1-1.cdh6.2.1.p0.1425774-el7.parcel",
            "replaces": "IMPALA, SOLR, SPARK, KAFKA, IMPALA_KUDU, KUDU, SPARK2"
	*	echo "e3b31081a200ee9ee10aa64e48a0a3d62b25a7b1" >  CDH-6.2.1-1.cdh6.2.1.p0.1425774-el7.parcel.sha
	*	chown cloudera-scm.cloudera-scm /opt/cloudera/parcel-repo/*



### 启动 cm  (namenode - viya02)
*	vim /etc/cloudera-scm-agent/config.ini
	*	server_host = viya02
*	systemctl start cloudera-scm-server
*	systemctl status cloudera-scm-server
*	tail -f /var/log/cloudera-scm-server/cloudera-scm-server.log
*	问题： Loaded: loaded (/usr/lib/systemd/system/cloudera-scm-server.service; enabled; vendor preset: disabled)
	*	journalctl -n 20 查看service启动日志 Cloudera Manager requires Oracle JDK 1.8 or later.
	*	journalctl -f
*	scp oracle-j2sdk1.8-1.8.0+update141-1.x86_64.rpm root@172.17.0.2:/home
*	rpm -ivh  oracle-j2sdk1.8-1.8.0+update141-1.x86_64.rpm

## slave1-3
*	cdh源配置
		*	mkdir /opt/cloudera-manager/
		*	cp cloudera-manager.repo /etc/yum.repos.d/
		*   yum -y install  cyrus-sasl-gssapi      bind-utils psmisc     libxslt   cyrus-sasl-plain  openssl cyrus-sasl-gssap  fuse  portmap    fuse-libs /lib/lsb/init-functions   httpd   mod_ssl   openssl-devel   python-psycopg2     MySQL-python     libpq.so.5
		*   rpm -ivh   cloudera-manager-daemons-6.2.1
		*   rpm -ivh   cloudera-manager-agent-6.2.1
		*	vim /etc/cloudera-scm-agent/config.ini 修改server_host
		*	jdk安装
			*	docker-machine ssh default
			*	scp oracle-j2sdk1.8-1.8.0+update141-1.x86_64.rpm root@172.17.0.2:/home
			*	rpm -ivh  oracle-j2sdk1.8-1.8.0+update141-1.x86_64.rpm




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
*	cdh 安装页面：http://172.19.48.189:7180/cmf/login
	*	admin/admin
	*	CDH 版本:CDH-6.1.0-1.cdh6.1.0.p0.770702
	*	数据库
		*	https://www.cloudera.com/documentation/enterprise/latest/topics/install_cm_mariadb.html
		*	密码：root
		*	metastore  hive root
		*	amon,amon,root
		*	oozie,oozie,root
		*	hue,hue,root

*	 安装过程中如果遇到：/opt/cloudera/parcels/CDH-5.8.2-1.cdh5.8.2.p0.3/meta/parcel.json 文件无法找到
sudo /bin/systemctl stop cloudera-scm-agent
sudo rm /var/lib/cloudera-scm-agent/*
sudo /bin/systemctl start cloudera-scm-agent
*	主机运行状况不良
cd /var/lib/cloudera-scm-agent/
rm -rf cm_guid 
service cloudera-scm-agent restart
*	某几台机器无法发布parecle , 已分配，/opt/cloudera/parcels没有内容
	*	tail -n400 /var/log/cloudera-scm-agent/cloudera-flood.log
	*	tail -n500  /var/log/cloudera-scm-agent/cloudera-scm-agent.log
		*   host名称配错了

### 运行测试
*	spark-shell:
	*	 Permission denied: user=root, access=WRITE, inode="/user":hdfs:supergroup:drwxr-xr-x
		*	usermod -s /bin/bash hdfs
	*	adduser hdfs  用hdfs账号登录

    
### hive 测试
*	Given NMToken for application : appattempt_1556161534641_0003_000002 is not valid for current node manager.expected : slave2:8041 found : slave1:8041 
	*	ip改变了，对不上
```
CREATE  TABLE test (   id int) ROW FORMAT DELIMITED FIELDS TERMINATED BY ','  ;

insert into test values(1);
```

##  web页面
*	https://www.cloudera.com/documentation/enterprise/latest/topics/cm_ig_ports_cm.html
*	https://www.cloudera.com/documentation/enterprise/latest/topics/cdh_ports.html
*	hadoop resource manager
	*	http://172.19.48.189:8088/cluster


##  单独升级spark
*	https://blog.csdn.net/Abysscarry/article/details/79550746
*	https://docs.cloudera.com/documentation/spark2/latest/topics/spark2_installing.html
*	https://docs.cloudera.com/documentation/spark2/latest/topics/spark2_packaging.html#versions
	*	SPARK2-2.4.0.cloudera2-1.cdh5.13.3.p0.1041012-el7.parcel.sha1
	*	SPARK2-2.4.0.cloudera2-1.cdh5.13.3.p0.1041012-el7.parcel
	*	manifest.json
*	/opt/cloudera/csd  
	*	cp SPARK2_ON_YARN-2.4.0.cloudera2.jar /opt/cloudera/csd/SPARK2_ON_YARN-2.4.0.cloudera2.jar
	*	chown cloudera-scm /opt/cloudera/csd/SPARK2_ON_YARN-2.4.0.cloudera2.jar
	*	chgrp cloudera-scm /opt/cloudera/csd/SPARK2_ON_YARN-2.4.0.cloudera2.jar
    *	然后将CSD的jar包放在集群所有机器的该目录，并修改文件所属用户和组（注意如果本目录下有其它jar包，请删掉或者移到其他目录）
*	/opt/cloudera/parcel-repo
	*	mv manifest.json manifest.json.old
	*	SPARK2-2.4.0.cloudera2-1.cdh5.13.3.p0.1041012-el7.parcel.sha1
	*	SPARK2-2.4.0.cloudera2-1.cdh5.13.3.p0.1041012-el7.parcel
	*	manifest.json
	*	去掉 SPARK2-2.2.0.cloudera1-1.cdh5.12.0.p0.142354-el7.parcel.sha1 后面的1
		*	mv SPARK2-2.4.0.cloudera2-1.cdh5.13.3.p0.1041012-el7.parcel.sha1 SPARK2-2.4.0.cloudera2-1.cdh5.13.3.p0.1041012-el7.parcel.sha 	
	*	systemctl restart cloudera-scm-server
	*	systemctl restart cloudera-scm-agent