####  org.apache.spark.deploy.SparkSubmit执行流程

```
SparkSubmit#doSubmit
// 解析参数并做参数验证，
val appArgs = parseArguments(args)
submit(appArgs, uninitLog)
```

```scala
//SparkSubmit#submit
// 环境准备 
// childMainClass: org.apache.spark.deploy.yarn.YarnClusterApplication
val (childArgs, childClasspath, sparkConf, childMainClass) = prepareSubmitEnvironment(args)

// 根据准备的环境 执行child class的main方法
// mainClass 指定的情况并且是SparkApplication的子类: isYarnCluster 的情况， mainClass为org.apache.spark.deploy.yarn.YarnClusterApplication
val app: SparkApplication = if (classOf[SparkApplication].isAssignableFrom(mainClass)) {
      mainClass.newInstance().asInstanceOf[SparkApplication]
    } else {
      // SPARK-4170
      if (classOf[scala.App].isAssignableFrom(mainClass)) {
        logWarning("Subclasses of scala.App may not work correctly. Use a main() method instead.")
      }
      new JavaMainApplication(mainClass)
    }

// 当yarn cluster的场景，启动的是YarnClusterApplication
// 当yarn client的场景，启动的是JavaMainApplication
```



#####   yarn cluster启动 YarnClusterApplication

```scala
// 实现了SparkApplication接口
private[spark] class YarnClusterApplication extends SparkApplication {
	override def start(args: Array[String], conf: SparkConf): Unit = {
    // SparkSubmit would use yarn cache to distribute files & jars in yarn mode,
    // so remove them from sparkConf here for yarn mode.
    conf.remove("spark.jars")
    conf.remove("spark.files")

    new Client(new ClientArguments(args), conf).run()
  }
}

// org.apache.spark.deploy.yarn.Client
// 将应用程序提交到 ResourceManager 执行 ApplicationMaster 
 def submitApplication(): ApplicationId = {
    var appId: ApplicationId = null
    try {
      launcherBackend.connect()
      yarnClient.init(hadoopConf)
      yarnClient.start()

      logInfo("Requesting a new application from cluster with %d NodeManagers"
        .format(yarnClient.getYarnClusterMetrics.getNumNodeManagers))

      // Get a new application from our RM
      val newApp = yarnClient.createApplication()
      val newAppResponse = newApp.getNewApplicationResponse()
      appId = newAppResponse.getApplicationId()

      new CallerContext("CLIENT", sparkConf.get(APP_CALLER_CONTEXT),
        Option(appId.toString)).setCurrentContext()

      // 验证集群是否有足够的资源运行AM
      verifyClusterResources(newAppResponse)

      // 设置启动am的环境参数，包括环境变量，java options等
      val containerContext = createContainerLaunchContext(newAppResponse)
      val appContext = createApplicationSubmissionContext(newApp, containerContext)

      // 提交am到ResourceManager
      yarnClient.submitApplication(appContext)
      launcherBackend.setAppId(appId.toString)
      reportLauncherState(SparkAppHandle.State.SUBMITTED)

      appId
    } catch {
      case e: Throwable =>
        if (appId != null) {
          cleanupStagingDir(appId)
        }
        throw e
    }
  }
```



#####  yarn client 启动  JavaMainApplication 

```scala
// 直接在SparkSubmit程序通过反射调用--class指定类的main方法
private[deploy] class JavaMainApplication(klass: Class[_]) extends SparkApplication {
  override def start(args: Array[String], conf: SparkConf): Unit = {
    val mainMethod = klass.getMethod("main", new Array[String](0).getClass)
    if (!Modifier.isStatic(mainMethod.getModifiers)) {
      throw new IllegalStateException("The main method in the given main class must be static")
    }
    val sysProps = conf.getAll.toMap
    sysProps.foreach { case (k, v) =>
      sys.props(k) = v
    }
    mainMethod.invoke(null, args)
  }
}
```

#####   driver 执行内存与cpu资源配置 -- 具体参照org.apache.spark.deploy.yarn.Client解析

* yarn  cluster的情况
  * 通过spark.yarn.am.memory参数配置，运行在AM里面，默认512m
  * cpu通过spark.yarn.am.cores，默认1
* yarn client的情况
  * 通过spark.driver.memory参数配置，在driver端，默认1g
  * cpu通过spark.driver.cores配置，默认1

