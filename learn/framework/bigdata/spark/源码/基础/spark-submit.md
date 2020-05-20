####  org.apache.spark.deploy.SparkSubmit执行流程

```
SparkSubmit#doSubmit
// 解析参数并做参数验证，
val appArgs = parseArguments(args)
submit(appArgs, uninitLog)
```

```
SparkSubmit#submit
// 环境准备 
// childMainClass: org.apache.spark.deploy.yarn.YarnClusterApplication
val (childArgs, childClasspath, sparkConf, childMainClass) = prepareSubmitEnvironment(args)


```

