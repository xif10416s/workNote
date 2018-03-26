#Spark Streaming
## before spark 2.0
## DStream
*   kafka
    -   https://my.oschina.net/u/1250040/blog/908571
*   描述
    -   相同类型的一组RDD,代表连续的数据流
    -   可以由实时数据流创建,TCP sockets , kafka ,flume ,现有DStreams转换操作,
    -   该类提供了基础的map,fliter,widow,PairDStreamFunction ==>join,groupByKeyAndWindow
    -   batchInterval＝＞　batchDuration，执行任务的周期，包含一个或者多个ｂｌｏｃｋ
    -   blockInterval　＝＞　receiver接收到数据后创建ｂｌｏｃｋ的周期
    -   一个DStream关联一个ｒｅｃｅｉｖｅｒ，需要从多个Receiver并行读取需要创建多个DStream,一个Receiver运行载一个Ｅｘｅｃｕｔｏｒ，占用１个ｃｐｕ
    -   来自数据源的数据接收到以后，ｒｅｃｅｉｖｅｒ会每隔blockInterval时间生成ｂｌｏｃｋ
    -   batchInterval　间隔的block被创建成RDD,block数就是ＲＤＤ的分区数，分区数对应task数， blockInterval== batchinterval 　意味着只有一个分区，本地执行

*   子类
    -   InputDStream
        -   描述
            -   所有输入流的基类
            -   spark stream 系统会调用 start 和stop来开始或者停止接受数据，比InputStream多这两个接口
            -   只需要在ｄｒｉｖｅｒ上生成RDD的，可以继承InputDStream通过一个线程在driver节点生成RDD.
                -   例如:FileInputDStream => 监控 HDFS目录生成的新文件
            -   需要在ｗｏｒｋｅｒ节点上接收数据的，需要继承ReceiverInputDStream
        -   子类
            -   ReceiverInputDStream
                -   描述
                    -   在ｗｏｒｋｅｒ节点启动ｒｅｃｅｉｖｅｒ接收外部数据
                    -   新增getReceiver接口，用以ｗｏｒｋｅｒ　节点获取ｒｅｃｅｉｖｅｒ
                -   子类
                    -   SocketInputDStream
            -   DirectKafkaInputDStream

*   缺点
    -   需要ｔｒａｎｓｆｅｒ

##DStreamGraph

##JobGenerator
*   batchDuration 生成一次执行任务事件

##StreamingContext
*   batchDuration
    -   任务周期间隔

##Receiver
*   在ｗｏｒｋｅｒ节点获取数据的对象
    -   ReceiverSupervisor
        -   ReceiverSupervisorImpl
            -   pushSingle　将接收到的数据添加数据到BlockGenerator的ｂｕｆｆｅｒ
            -   pushAndReportBlock
                -   通过blockManager存储ｂｌｏｃｋ
                -   通知更新blockInfo
    -   ReceiverTracker
        -   ReceiverTrackerEndpoint
    -   JobScheduler
    -   BlockGenerator
        -   RateLimiter
            -   waitToPush
                -   限制一次处理一个消息，消息处理时新来的消息被block
            -   com.google.common.util.concurrent.RateLimiter
                -   每秒处理多少任务量
                -   批处理时候每秒多少大小一批
        -   启动２个线程，一个线程负责把ｒｅｃｅｉｖｅｒ中获取的数据分批，另一个线程负责把这些分批好的块push到block manager
        -   spark.streaming.blockInterval
            -   buffer刷新block时间默认２００ms

