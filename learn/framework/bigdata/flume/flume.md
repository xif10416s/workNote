#   flume.md
##  主要组件 & 接口 ，http://blog.csdn.net/englishsname/article/details/53013841
*   LifecycleAware
    -   所有组件基础生命周期控制，包括sink,source,channel
    -   start() & stop() 开启或者停止一个服务或者组件
*   source
    -   消息生产者，调用ChannelProcessor持久化消息到channel
    -   ChannelProcessor
        +   将事件put到对应的channel
        +   ChannelSelector:selector
            -   每个ChannelProcessor都有一个ChannelSelector，来选择需要的channel
        +   InterceptorChain interceptorChain
*   Channel
    -   连接source(事件生产者)与sink（事件消费者），起到缓存队列的作用 
*   sink
    -   消费事件
    -   可以更具sink的行为分组SinkGroup
*   SinkRunner
    -   可能对应一个sink也可能对应一个sinkgroup
    -   SinkProcessor policy
        +   调用sink.process处理
    -   PollingRunner
        +   循环处理线程
        +   如果当前批次没有处理任何数据需要等待
            *   policy.process().equals(Sink.Status.BACKOFF)
*   Transaction -- MemoryTransaction 实现
    -   主要通过2个list
        +   putList = new LinkedBlockingDeque<Event>(transCapacity);
            *   记录新收取到的准备写入channel的事件
        +   takeList = new LinkedBlockingDeque<Event>(transCapacity);
            *   记录准备消费的事件
        +   transactionCapacity : 队列容量
    -    source的ChannelProcessor 与 sink的process 时都会从channel 获取transaction,
        +   但是ChannelProcessor 与sink 是不同的线程，transaction与当前线程绑定，所以是两个不同的transaction
        +    多个线程对应的多个 transaction 并发操作MemoryChannel的全局变量：
            -    Object queueLock
                -   队列锁
            -   LinkedBlockingDeque<Event> queue = new LinkedBlockingDeque<Event>(capacity)
                -   共享队列，代表实际操作的事件队列，doPut提交后的队列
            -    Semaphore queueRemaining = new Semaphore(capacity)
                +    队列的剩余容量的控制，虽然只有source在put,但是sink回滚时候也需要放回队列，就可能超过队列容量
            -    queueStored = new Semaphore(0)
                +    source放入队列消息和sink获取队列消息的线程信号协调，放入多少次才能获取多少次
        +   source获取的transaction只使用了putList，takeList始终为空
        +   sink的transaction只使用了takeList，putList始终为空
        +   doCommint方法为source与sink共用，一套代码实现了2个功能
    -   doBegin方法为空实现没有具体操作
    -   doPut方法source写入事件
        +   向putList添加event,当超过容量时ChannelException
    -   doTake方法sink获取事件并处理
        +   takeList放满时ChannelException
        +   从queue获取事件，event = queue.poll()，放入takeList
    -   doCommit提交事务
        +   putList有事件时，将putList内容转移到queue中，queue.offer(putList.removeFirst()
            *   然后清空putList和takeList
        +   queueStored = new Semaphore(0)，一开始无法doTake,要等待有了doPut并doCommint之后,queueStored.release(puts),才可以doTake
            *   doPut多少量，doTake才能处理多少
        -   remainingChange = takeList.size() - putList.size();
    -   doRollback回滚
        +   对于sink来说：
            -   把takeList内容放回queue中,queue.addFirst(takeList.removeLast());
            -   释放队列信号量，可以让sink重新take,queueStored.release(takes);
        +   对于source来说：
            *   清空putList就行，putList.clear();

## 命令
*   启动：
    -    bin/flume-ng agent -n flumeagent1 -c conf -f conf/post_action.properties

# hive sink
## hive sink 配置参数
*   http://blog.51niux.com/?id=197
*   hive.txnsPerBatchAsk    多少批处理为一个文件，控制文件的大小
*   batchSize  一批次处理多少数据
    -   hdfs 一个文件的大小 =   hive.txnsPerBatchAsk  * batchSize
*   round, roundValue,roundUnit三个参数是用来配置每10分钟在hdfs里生成一个文件夹
    *    a1.sinks.k1.roundValue = 10
    *    以上配置将会将时间戳降至最后10分钟。 例如，将时间戳标题设置为2012年6月12日上午11:54:34，将“country”标题设置为“india”的事件将评估为分区（continent ='asia'，country ='india'，time ='2012-06-12-11-50'。序列化程序配置为接受包含三个字段的选项卡分隔输入，并跳过第二个字段。

##  一批次数据处理流程
*   循环batchSize次，for (; txnEventCount < batchSize; ++txnEventCount)
    -   从channel获取一个事件，Event event = channel.take();
    -   创建hive通信，与hive的metaStore交互获取元数据，HiveEndPoint：endPoint
        +   设置partition变量的情况下，需要配置一下partition
            *   realPartVals.add(BucketPath.escapeString(partVal, headers, timeZone,
                  needRounding, roundUnit, roundValue, useLocalTime));
            *   所以 needRounding，roundUnit，roundValue，useLocalTime都是针对timestamp的处理，而timestamp是用来做partition路径取值的，也就是决定了按时间分割的partitin的取值，
                -   是否需要对时间做取整，按什么单位，多少取整
    -   利用endpoint创建HiveWriter:writer，`new HiveWriter(endPoint, txnsPerBatchAsk, autoCreatePartitions,
                    callTimeout, callTimeoutPool, proxyUser, serializer, sinkCounter)` ,并写入event，writer.write(event)
        -   HiveWriter初始化
            +   通过endPoint新建一个连接，StreamingConnection connection = newConnection(hiveUser);
            +   初始化RecordWriter recordWriter = serializer.createRecordWriter(endPoint) ，json的情况对应的实现是StrictJsonWriter
                *   负责写入刷新等批处理，底层是交给hive的RecordUpdater
            +   **获取下一个批次的hive事务，TransactionBatch txnBatch = nextTxnBatch(recordWriter)**
            +   **开始一个Transaction**，this.txnBatch.beginNextTransaction();
        -   HiveWriter.write
            +   HiveWriter内部有个ArrayList<Event> batch缓存，所有写入event都放入batch
            +   当batch.size() == writeBatchSz（1000）时，会将event序列化writeEventBatchToSerializer（）
                *   把batch中event写入serializer，serializer.write(txnBatch, event);
                    -   其实是通过批事务写入，txnBatch.write(e.getBody());
                        +   最终通过recordWriter.write(getCurrentTxnId(), record);
                *   清空batch
*   一批次循环完成，刷新数据，writer.flush(true);
    -   将batch的数据序列化，writeEventBatchToSerializer();
    -   提交事务，commitTxn
        +    recordWriter刷新，recordWriter.flush();写入hdfs
        +    提交当前currentTxnIndex的事务
    -   判断是否当前批次hive事务的所有transaction已经用完,txnBatch.remainingTransactions() == 0
        +   如果没有剩余transaction了，关闭批量hive事务，closeTxnBatch(),创建新的一批事务txnBatch = nextTxnBatch(recordWriter);
            *   关闭批事务，closeTxnBatch()
                -   recordWriter.closeBatch();
        +   如果还有，开启下一个transaction，txnBatch.beginNextTransaction();


### 问题
*   flume  hive sink 小文件
*   500万 = 40 m
*   分区时间为服务器时间，多批次数据在一个文件，对于同一个批次跨了分区怎么办


## 启动
### LifecycleSupervisor