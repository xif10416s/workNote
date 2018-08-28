#   spark 基本处理流程--RDD

##  org.apache.spark.examples.SparkPi处理分析

###  pi计算原理
*   http://blog.csdn.net/ranchlai/article/details/11019155
*   利用圆与其外接正方形面积之比为pi/4的关系（圆面积：pi*r*r ，正方形面积：2r*2r=4*r*r），通过产生大量均匀分布的二维点，计算落在单位圆和单位正方形的数量之比再乘以4便得到pi的近似值。样本点越多，计算出的数据将会越接近真识的pi


### 基本处理流程
####  基本流程
![](../images/1.jpg)
![](../images/2.png)

#### 详细流程
![](../images/baseprocess.jpg)


###  一、[初始化sparkContext](./1-SparkContext-初始化.md)
```
val spark = SparkSession
      .builder.master("spark://xifeideMacBook-Pro.local:7077")
      .config("spark.jars", "/Users/seki/git/learn/spark/examples/target/original-spark-examples_2.11-2.2.1-SNAPSHOT.jar")
      .appName("Spark Pi")
      .getOrCreate()
```

### 二、准备rdd [ParallelCollectionRDD] => `spark.sparkContext.parallelize(1 until 200000, 2)`
*   sparkContext#parallelize
    -   new ParallelCollectionRDD[T]，新建了一个rdd,继承自RDD
    -   数据源就是本地内存的 data: Seq[T] = 1 until 200000
    -   重写了最基本的RDD的三个方法
        +   数据逻辑分区 ==> getPartitions，获取所有的分区ParallelCollectionPartition
            -   numSlices = 2 ，所以 data序列切分成2个分区([1-100000][100001-200000])
        +   单个分区数据处理 ==> compute, 返回指定ParallelCollectionPartition数据的Iterator
            *   `new InterruptibleIterator(context, s.asInstanceOf[ParallelCollectionPartition[T]].iterator)`
            *   ParallelCollectionPartition 的iterator 就是`values.iterator`,也就是[1-100000]或者[100001-200000]list的Iterator
        *   getPreferredLocations
            -   空实现，不起作用

### 三、rdd的map操作 , 属于Transformations,不提交任务 ==> [ParallelCollectionRDD --> MapPartitionsRDD]
     map { i =>
      val x = random * 2 - 1
      val y = random * 2 - 1
      if (x*x + y*y <= 1) 1 else 0
    }
*   RDD#map =>
    *    val cleanF = sc.clean(f) 
        -    map函数f需要被序列化发送到worker节点执行
        -    检查外部变量使用是否正确，是否可序列化，保证能被序列化并发送到executor正确执行
    *   `new MapPartitionsRDD[U, T](this, (context, pid, iter) => iter.map(cleanF))`
        +   包装成MapPartitionsRDD
        +   构造函数接受一个函数 f: (TaskContext, Int, Iterator[T]) => Iterator[U],实现就是在对应partition上数据Iterator应用cleanF函数，也就是随机2个值然后计算是否落在园的函数，一个Partition执行100000次
        +   当实际出发计算的时候才真正执行计算

### 四、rdd的reduce操作，属于action，触发执行操作 `reduce(_ + _)`
*   RDD#reduce
*    val cleanF = sc.clean(f) ， 同上
*    定义partition的reducePartition函数,ResultStage会保存这个函数，提交任务时被序列化广播出去，map端先合并，比group by好</br>
```
        val reducePartition: Iterator[T] => Option[T] = iter => {
          if (iter.hasNext) {
            Some(iter.reduceLeft(cleanF))//Partition中元素执行reduce函数
          } else {
            None
          }
        }
```
*   定义如何各个partion并行处理结果函数mergeResult,task完成时调用</br>
```
val mergeResult = (index: Int, taskResult: Option[T]) => {
      if (taskResult.isDefined) {
        jobResult = jobResult match {
          case Some(value) => Some(f(value, taskResult.get))
          case None => taskResult
        }
      }
    }
```
*    运行job ``` sc.runJob(this, reducePartition, mergeResult)```

####    SparkContext#runJob 处理
    def runJob[T, U: ClassTag](
    rdd: RDD[T],//执行的任务的rdd[ MapPartitionsRDD[ParallelCollectionRDD]]
    processPartition: (TaskContext, Iterator[T]) => U,//定义如何处理Partition的函数
    resultHandler: (Int, U) => Unit)//定义如何处理返回结果函数

*   转交给DagScheduler处理任务```dagScheduler.runJob(rdd, cleanedFunc, partitions, callSite, resultHandler, localProperties.get)```

####   dagScheduler#runJob处理
*   生成jobid,`jobId = nextJobId.getAndIncrement()`
*   创建JobWaiter, `val waiter = new JobWaiter(this, jobId, partitions.size, resultHandler)`
    -   实现了JobListener，监听任务处理状态，等待所有任务执行完毕，调用resultHandler函数，处理返回结果
        +   `override def taskSucceeded(index: Int, result: Any): Unit = {}`
        -   在driver端执行
*   往dagScheduler的eventProcessLoop发送JobSubmitted事件
*   driver 的主线程进入等待状态，等待所有task执行完毕

####  [dag-scheduler-event-loop 线程] 异步处理 JobSubmitted事件
*   DAGSchedulerEventProcessLoop#doOnReceive  调用dagScheduler#handleJobSubmitted方法

####  首先划分stage, `finalStage = createResultStage(finalRDD, func, partitions, jobId, callSite)` //TODO
*   getOrCreateParentStages,根据依赖关系，划分stage ，当前例子中没有shuffle依赖所以为空
    -   getShuffleDependencies，计算依赖关系，查找当前rdd的距离最近的ShuffleDependency
        +   A <-- B <-- C，c和b是Shuffle依赖，b和a是Shuffle依赖，从C开始计算时，当期方法只返回，B <-- C 的依赖
        +   当前计算中RDD为MapPartitionsRDD，只有一个OneToOneDependency的ParallelCollectionRDD
        +   利用获取的ShuffleDependency调用getOrCreateShuffleMapStage，创建ShuffleMapStage
            *   执行过程中会递归调用getOrCreateParentStages，计算是否需要再划分stage
            *   当执行C的getOrCreateParentStages时，C Shuffle依赖 B，就返回C与B的依赖，为这个依赖创建stage的时候getOrCreateShuffleMapStage，计算了B 的依赖关系，发现了B与A的Shuffle依赖
*   创建new ResultStage，划分的stage list 作为parents 传入
*   updateJobIdStageIdMaps(jobId, stage)，更新当前jobid 和 所有stageid的映射
    -   jobIdToStageIds = new HashMap[Int, HashSet[Int]]

#####  提交stage,submitStage(stage: Stage)
*   输入是finalStage，因为依赖关系首先要计算父的stage,所以递归找到根stage，getMissingParentStages，先计算根state

#####  提交当前需要计算的stage,submitMissingTasks(stage: Stage, jobId: Int)
*   找到当前stage中需要计算的partition
    -   val partitionsToCompute: Seq[Int] = stage.findMissingPartitions()
*   给每个partition找到合适的worker计算节点，getPreferredLocs（rdd: RDD[_], partition: Int）: Seq[TaskLocation] ，就是什么partition在什么host上执行的map
    -   先检查是否缓存过，cached = getCacheLocs(rdd)(partition)，直接返回缓存的地址
    -   检查rdd是否有优选的地址preferredLocations(split: Partition)，不同rdd有特定的实现，主要目的为了数据本地化，让计算节点尽量读取本机的数据，减少网络读取数据，如：
        +   KafkaRDD#getPreferredLocations
            *   优选策略为：需要读取的kafka数据所在的主机的spark的worker的主机上执行
            *   val prefExecs = if (null == prefHost) allExecs else allExecs.filter(_.host == prefHost)
    -   检查当前rdd是否有窄依赖，如果有，就与父rdd的地址一致
*   序列化并广播算法：计算任务是要被发送到worker节点的executor中执行 --*rdd + func* 被序列化
    -   ShuffleMapStage =>closureSerializer.serialize((stage.rdd, stage.shuffleDep)
    -   ResultStage =>closureSerializer.serialize((stage.rdd, stage.func): AnyRef
        +   stage.func为reduce函数
    -  sc.broadcast(taskBinaryBytes),广播发送到集群中,返回的是广播的地址，worker节点可以根据这个地址获取序列化信息，反序列化成算法函数
*   stage转换为task
    -   ShuffleMapStage ==>  Seq[ShuffleMapTask]
    -   ResultStage ==>  Seq[ResultTask]
    -   一个partition 对应一个 task,一个task就是一个运行线程

####  提交任务集合TaskScheduler#submitTasks(new TaskSet）
*   创建TaskSetManager ， manager = createTaskSetManager(taskSet, maxTaskFailures)
    -   管理任务集合，跟踪处理
*   调用FIFOSchedulableBuilder#addTaskSetManager
    -   Pool#addSchedulable
        +   Pool#schedulableQueue.add(schedulable)，Pool代表一组计划任务集合
            *   schedulableQueue = new ConcurrentLinkedQueue[Schedulable]
        +   所有需要执行的TaskSetManager，被放入改队列，安排执行

####   申请资源（不同集群不同的SchedulerBackend），StandaloneSchedulerBackend#reviveOffers
*   通过driverEndpoint发送消息给StandaloneSchedulerBackend#makeOffers处理，
    -   找到所有可以运行的Executors，`val activeExecutors = executorDataMap.filterKeys(executorIsAlive)`
    -   统计可用Executors的空闲资源workOffers，一组WorkerOffer，`new WorkerOffer(id, executorData.executorHost, executorData.freeCores)`

####    通知集群在所有worker节点分配资源，TaskScheduler#resourceOffers，每个task分配好在哪个executor执行
*   把资源列表重新洗牌，val shuffledOffers = shuffleOffers(filteredOffers)
    -   避免相同的tasks总是分配到相同的workOffer上
*   获取排序后的任务集合ArrayBuffer[TaskSetManager]，val sortedTaskSets = rootPool.getSortedTaskSetQueue
    -   默认FIFO
*   TaskSetManager#resourceOffer,给当前任务集合中的每个task指定executorId，最终获取一份描述信息
    -   遍历所有task
    -   生成taskid,val taskId = sched.newTaskId(),
    -   序列化task，serializedTask: ByteBuffer
        +   DagScheduler#submitMissingTasks时序列化并广播了taskBinary，此时序列化的serializedTask是broadcast对象，任务被发送到executor执行时，反序列化的是serializedTask是broadcast对象，通过broadcast对象的value从远程driver端拉取block数据
            *   taskBinary内容：
               -   ShuffleMapStage = stage.rdd, stage.shuffleDep
               -   ResultStage = stage.rdd, stage.func
    -   生成task描述，new TaskDescription
        +   taskId
        +   execId
        +   serializedTask
*   TaskScheduler记录好分配情况，tid为taskid
    -   taskIdToTaskSetManager(tid) = taskSet，
    -   taskIdToExecutorId(tid) = execId
    -   executorIdToRunningTaskIds(execId).add(tid)
*   返回Seq[Seq[TaskDescription]] 
    -   第一层，taskSet
    -   第二层，tasks的描述信息
        +   某个task分配在哪个executor

####    启动task执行，StandaloneSchedulerBackend#launchTasks
*   将task信息编码为ByteBuffer，val serializedTask = TaskDescription.encode(task)
*   根据task分配的executorid获取excutor,val executorData = executorDataMap(task.executorId)
*   给executor发送启动任务消息，executorData.executorEndpoint.send(LaunchTask(new SerializableBuffer(serializedTask)))


##  Worker端处理
###    worker节点的executor接受到消息， CoarseGrainedExecutorBackend#receive  
*   解码消息内容， val taskDesc = TaskDescription.decode(data.value)

### 执行task,Executor#launchTask
*   新建TaskRunner，val tr = new TaskRunner(context, taskDescription)
    -   继承自Runnable
*   启动线程池，执行任务，threadPool.execute(tr)

### 任务执行，TaskRunner#run
*   反序列化，获取task = ser.deserialize[Task[Any]](
          taskDescription.serializedTask, Thread.currentThread.getContextClassLoader)
*   调用task#run，执行任务

###    Task#run，实际有两种子类实现runTask方法
####    ResultTask#runTask，执行并将结果返回给driver
*   ResultTask 对应的序列化task内容为 ：rdd 和 func 在deiver划分任务的时执行了变量广播，executor运行任务的时候是使用的广播变量，所以先调用taskBinary.value，拉取广播序列化数据，在反序列化成 rdd + func对象
从广播变量中反序列化出rdd和相应的运算函数func,`val (rdd, func) = ser.deserialize[(RDD[T], (TaskContext, Iterator[T]) => U)]`
*   执行计算，`func(context, rdd.iterator(partition, context))`
    1.   rdd.iterator(partition, context),也就是MapPartitionsRDD的compute中对每个元素执行 map定义的函数（随机2个数字，计算在圆内为1，否则为0）,一个partition计算10000次
    2.   func函数，是reduce函数，map执行完成，在worker的executor端做合并，减少返回，也就是_+_
*   将返回值序列化，serializedDirectResult = ser.serialize(directResult)
*   Executor#run 返回结果给driver
    -   返回结果大于maxResultSize【默认1gb】,返回结果的blockid，让driver自己拉取，` ser.serialize(new IndirectTaskResult[Any](TaskResultBlockId(taskId), resultSize))`
    -   否则直接返回序列化结果
*    给driver发送结束状态消息，execBackend.statusUpdate(taskId, TaskState.FINISHED, serializedResult)
    -    driverRef.send(msg)


##  driver端接受到完成消息，CoarseGrainedSchedulerBackend#receive
### TaskSchedulerImpl#statusUpdate
### TaskResultGetter#enqueueSuccessfulTask
*   启动线程处理返回结果  (deserializedResult, size)
    -   getTaskResultExecutor.execute(new Runnable {}）
    -   返回的是直接结果，`case directResult: DirectTaskResult[_]`
        +   直接反序列化获取结果，`directResult.value(taskResultSerializer.get())`
    -   结果太大，返回blockid,`case IndirectTaskResult(blockId, size) =>`
        +   通过blockid，远程获取结果内容，` val serializedTaskResult = sparkEnv.blockManager.getRemoteBytes(blockId)`

### 处理成功结果DagScheduler#handleTaskCompletion
*   给jobwaiter触发任务完成事件，job.listener.taskSucceeded(rt.outputId, event.result)

### JobWaiter#taskSucceeded
*   调用resultHandler(index, result.asInstanceOf[T])处理返回结果 
*   等待所有task完成
    -   if (finishedTasks.incrementAndGet() == totalTasks) {
      jobPromise.success(())
    }

##  总结
*   数据源加载创建基础RDD,在这个rdd上各种Transformations操作会包装成对应的新的RDD,如MapPartitionsRDD
*   不同的操作map,group都有对应的类型的RDD
*   reduce函数首先会在work端先执行,操作对象为一个partition中的所有数据，然后在driver的合并task结果执行，操作对象为partition
*   

##  pi计算流程mock代码，最基础的计算流程[模拟代码地址](https://github.com/xif10416s/bigData/tree/master/scalaProject/scala-test-spark2.2/src/main/scala/com/fxi/test/base/mock/simpleprocess)
*   map函数，reduce函数，调用流程
*   计算结果如何合并
*   最基础的计算流程了解

### 运行结果
```
STEP 1 : 准备map函数 , 准备reduce函数 , 准备数据,设置切片数量
STEP 2 : 根据reduce函数创建partition内数据的reduce函数reducePartition
STEP 3 : 根据reduce函数创建partition的reduce函数mergeResult
STEP 3 :  创建JobWaiter等待并监听任务执行成功结果
STEP 4 :  提交任务执行
STEP 5 : 通过jobwaiter,主线程进入等待状态 
线程1 rdd.compute 并 调用reducePartition,合并单个partition结果 
线程0 rdd.compute 并 调用reducePartition,合并单个partition结果 
线程0 触发jobwaiter任务完成 返回结果 Some(78505)
线程1 触发jobwaiter任务完成 返回结果 Some(78489)
任务0完成, JobWaiter 调用mergeResult函数合并成功task结果[Some(78505)]
任务1完成, JobWaiter 调用mergeResult函数合并成功task结果[Some(78489)]
partition:{1} 合并任务结果开始..
所有任务完成...
STEP 6 : 任务全部完成,返回结果 
Pi is roughly 3.1398956994784974
```

## SparkContext启动参数
```
val spark = SparkSession
      .builder.master("spark://xifeideMacBook-Pro.local:7077") // standalone 集群地址
      .config("spark.jars", "/Users/seki/git/learn/spark/examples/target/original-spark-examples_2.11-2.2.1-SNAPSHOT.jar") // 运行时依赖jar,需要被发送的worker
      .config("spark.executor.memory", "512m") // 每个executor 启动内存
      .config("spark.executor.cores", "1")   // 每个executor 分配cpu数【不指定的情况，每个worker只有一个executor,使用所有分配的cpu】
      .config("spark.cores.max","4")   // app任务 最多使用多少个 cpu
      .config("spark.task.cpus","1")  // 每个执行的task 使用的cpu数，默认是1  ，所以同时并发线程数 = 分配的 cpu数 ， 1个task 可能 用多个 cpu
       .config("spark.memory.fraction","0.3") // 存储内存百分比 
      .appName("Spark Pi")
```

## 其他参数
*   spark.locality.wait和spark.locality.wait.process，spark.locality.wait.node, spark.locality.wait.rack这几个参数影响了任务分配时的本地性策略的相关细节。

## Executor 创建   -- Spark standalone
##  worker | executor | tasks
*   资源分配
    -   worker1 , cores = 4
    -   worker2 , cores = 4
*   executor 个数 = 最大可用 cpu 数 / 每个 executor 指定cpu 数
    -   spark.executor.cores 不指定，一个worker  使用一个 executor , 分配当前work 所有可用 cpu 数
*   并发线程数 =  最大可用cpu 数 /  spark.task.cpus(每个task 使用的cpu 数)
    -   一个task 至少使用 1个 cpu
    -   并发线程数 <=  当前可用cpu数
*   task的执行有TaskSchedulerImpl分配
    -   为什么能够做到 1 个 task 分配一个cpu , 因为TaskSchedulerImpl这边根据可用cpu 和 spark.task.cpus数量分配，所以1个task 至少有一个可用cpu 才能 launch task
        +   worker 端的 Executor 的 处理任务的线程池 threadPool 是没有数量限制的， 在给定的cpu数量下，接受到多少个task 就启动多少个线程

## submit 参数
###   Run on a Spark standalone
*   --executor-memory 20G   
*   --total-executor-cores 100 
*   --driver-memory 512m --



###  Run on a YARN cluster
*   --executor-memory 20G  每个executor 多少内存
*   --num-executors 50    定义多少个executor
*   --executor-cores 2   每个executor 多少cpu

### spark on yarn cluster vs client
*   http://blog.cloudera.com/blog/2014/05/apache-spark-resource-management-and-yarn-app-models/

$SPARK_HOME/bin/spark-submit --master yarn --deploy-mode client --name test  --num-executors 6 --executor-cores 3   testprime.py

3worker  9contain   15 GB /  48 GB   0 B   21 / 24 

/default-rack   RUNNING   uhadoop-adarbt-core2:23333  uhadoop-adarbt-core2:23999  Wed Nov 29 11:18:51 +0800 2017    2   4 GB  12 GB   6   2   2.6.0-cdh5.4.9
  /default-rack   RUNNING   uhadoop-adarbt-core1:23333  uhadoop-adarbt-core1:23999  Wed Nov 29 11:17:54 +0800 2017    2   4 GB  12 GB   6   2   2.6.0-cdh5.4.9
  /default-rack   RUNNING   uhadoop-adarbt-core3:23333  uhadoop-adarbt-core3:23999  Wed Nov 29 11:18:35 +0800 2017    3   5 GB  11 GB   7   1   2.6.0-cdh5.4.9 