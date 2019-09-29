hadoop.md
# command
*	http://hadoop.apache.org/docs/r3.0.0/hadoop-project-dist/hadoop-common/FileSystemShell.html
* hadoop fs -rmr /var/log/hadoop-yarn/apps/root/logs/
* hadoop fs -get /var/log/spark/application_1514973560079_8600 /tmp/application_1514973560079_8600 
* spark on yarn 日志查看：
    - yarn logs -applicationId application_1514973560079_8600


##  基础文件操作
*	创建目录
	*	hadoop fs -mkdir /rawfile
	*	hadoop fs -mkdir -p /rawfile/business/retail/huadi/201911/
*	上传本地路径文件到hdfs
	*	 hadoop fs -copyFromLocal <localsrc> URI
	*	 hadoop fs -copyFromLocal  /sas_work/data_dump/yn_data.dmp  /rawfile/business/retail/huadi/201911/
	*	 hadoop fs -copyFromLocal  /sas_work/supermarket/huadi/original_data/* /rawfile/business/retail/huadi/201911/
	*	hadoop fs -copyFromLocal  /sas_work/electric/taigu/original_data/* /rawfile/business/electric/taigu/201907/
*	复制hdfs文件到本地
	*	  hadoop fs -copyToLocal /rawfile/business/retail/huadi/201911/text.txt /tmp