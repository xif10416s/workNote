####   在yarn上运行的基本操作
* 程序提交的服务器需要包含HADOOP_CONF_DIR 或者 YARN_CONF_DIR环境变量，指向包含hadoop集群配置的文件目录
  
*	 因为程序提交时，不会指定 与 HDFS , YARN resourceMamager交互的地址和端口，是从服务器环境变量找到对于的配置文件去读取相应的配置
  
* spark on  yarn的两种运行方式：

  *	cluster 模式：客户端初始化程序后退出了，driver程序会运行在集群上的Application master中被yarn管理 
    *	日志需要登录到集群节点查看，client初始完成就结束了
    *	适用于生产环境
    *	yarn-cluster 不支持spark-shell  ， 也不支持本地启动，只能通过spark-submit
    *	![img](../../../images/yarn_cluster.png)
      *	基础流程：
        *	客户端程序向ResourceManager提交申请
  *	client模式：driver运行在客户端，application master只是用来从yarn申请资源
    *	适用于调试，能直接看到driver的日志，但是client断了，任务就结束了
    *	适用于交互与调试
    *	![img](../../../images\yarn_client.png)

*	日志查看方式：

  *	```
    yarn logs -applicationId <app ID>
    ```
    *	 `yarn.log-aggregation-enable`  需要开启
    *	所有application的containers产生的日志都会被打印
    
  *	也可以直接查看hdfs日志文件
  
    *	文件位置：yarn.nodemanager.remote-app-log-dir and yarn.nodemanager.remote-app-log-dir-suffix
  
  *	 Spark Web UI 也可以查看
  
    *	Spark history server 与 MapReduce history server 需要同时启动
  
  *	如果日志收集没有开启`yarn.log-aggregation-enable` =false
  
    *	日志会被保存在本地目录，配置地址YARN_APP_LOGS_DIR
    *	默认地址/tmp/logs` or `$HADOOP_HOME/logs/userlogs
  
* 启动命令

```
./bin/spark-submit --class org.apache.spark.examples.SparkPi \
    --master yarn \
    --deploy-mode cluster \
    --driver-memory 4g \
    --executor-memory 2g \
    --executor-cores 1 \
    --queue thequeue \
    examples/jars/spark-examples*.jar \
    10

 ./bin/spark-shell --master yarn --deploy-mode client
 
 
$ ./bin/spark-submit --class my.main.Class \
    --master yarn \
    --deploy-mode cluster \
    --jars my-other-jar.jar,my-other-other-jar.jar \
    my-main-jar.jar \
    app_arg1 app_arg2
```



####  sparkcontext初始化过程--on yarn

```
1. driver端 初始化 sparkcontext, 会根据master配置适用不同的SchedulerBackend实现类, 以及taskScheduler的实现类
val (sched, ts) = SparkContext.createTaskScheduler(this, master, deployMode)

2. SparkContext##createTaskScheduler 初始化 
2.1  通过SPI加载外部ExternalClusterManager实现类 ， spark 支持kubernates,mesos，yarn集群
ServiceLoader.load(classOf[ExternalClusterManager], loader).asScala.filter(_.canCreate(url))

在spark源码resource-manamgers模块下有这三种集群的ExternalClusterManager的实现，标准的SPI扩展方式
ExternalClusterManager用来创建集群对应的scheduler 与 backend的实现类

在resource-mamagers/yarn/src/main/resources/META-INF/services/org.apache.spark.scheduler.ExternalClusterManager文件中定义了
org.apache.spark.scheduler.cluster.YarnClusterManager

2.2  YarnClusterManager负责创建yarn的scheduler 和 backend
// yarn 的 taskScheduler创建，区分cluster 与 client 模式
override def createTaskScheduler(sc: SparkContext, masterURL: String): TaskScheduler = {
    sc.deployMode match {
      case "cluster" => new YarnClusterScheduler(sc)
      case "client" => new YarnScheduler(sc)
      case _ => throw new SparkException(s"Unknown deploy mode '${sc.deployMode}' for Yarn")
    }
  }

// yarn 的schedulerBackend的创建，区分cluter 与 client
  override def createSchedulerBackend(sc: SparkContext,
      masterURL: String,
      scheduler: TaskScheduler): SchedulerBackend = {
    sc.deployMode match {
      case "cluster" =>
        new YarnClusterSchedulerBackend(scheduler.asInstanceOf[TaskSchedulerImpl], sc)
      case "client" =>
        new YarnClientSchedulerBackend(scheduler.asInstanceOf[TaskSchedulerImpl], sc)
      case  _ =>
        throw new SparkException(s"Unknown deploy mode '${sc.deployMode}' for Yarn")
    }
  }

```

#####   client模式初始化  YarnScheduler &  YarnClientSchedulerBackend

```
##  YarnClientSchedulerBackend  <<  YarnSchedulerBackend  << CoarseGrainedSchedulerBackend
1.  在CoarseGrainedSchedulerBackend中定义了driver 的通信对象driverEndpoint
val driverEndpoint = rpcEnv.setupEndpoint(ENDPOINT_NAME, createDriverEndpoint())

2.  YarnSchedulerBackend中实现了该方法，实现类为YarnDriverEndpoint
override def createDriverEndpoint(properties: Seq[(String, String)]): DriverEndpoint = {
    new YarnDriverEndpoint(rpcEnv, properties)
  }



```









####  参考

* https://blog.csdn.net/wjl7813/article/details/79968423