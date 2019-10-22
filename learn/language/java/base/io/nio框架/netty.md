#  Netty4.x 学习
##   Netty特点
*   设计优雅--适用于各种传输类型的统一 API 阻塞和非阻塞 Socket；基于灵活且可扩展的事件模型，可以清晰地分离关注点；高度可定制的线程模型 - 单线程，一个或多个线程池；真正的无连接数据报套接字支持
*   高性能，高吞吐，低延迟，零拷贝
*   支持SSL/TSL 和 StartTLS，安全协议
*   使用场景
    *   分布式系统中，远程服务相互通信如：分布式服务框架Dubbo, spark,hadoop

##   基础组件 -- [测试代码](https://github.com/xif10416s/java_test)
*   ChannelInboundHandlerAdapter ：  各种事件处理接口实现
    *   channelRead --  从客户端接收到消息的时候调用，处理消息
    *   channelActive  -- 连接建立完成，可以发送消息
*   NioEventLoopGroup : 是用来处理I/O操作的多线程事件循环器，Netty 提供了许多不同的 EventLoopGroup 的实现用来处理不同的传输
    *   第一个经常被叫做‘boss’，用来接收进来的连接。
    *   第二个经常被叫做‘worker’，用来处理已经被接收的连接，一旦‘boss’接收到连接，就会把连接信息注册到‘worker’上。
*   ServerBootstrap :  是一个启动 NIO 服务的辅助启动类
*   NioServerSocketChannel:  用来实例化新的channel，接收新来的连接
*    ChannelInitializer： 是一个特殊的处理类，他的目的是帮助使用者配置一个新的 Channel。

##   基本流程
####   Client流程
*   1,初始化b = Bootstrap，服务辅助启动类
*   2,初始化工作组，workGroup = NioEventLoopGroup，并设定到Bootstrap，b.group(workGruop)
    -   是用来处理I/O操作的多线程事件循环器
    -   每次connect的时候开启一个线程，NioEventLoop
    -   默认16个NioEventLoop，需要的时候启动线程
*   3,设定channel类型，NioSocketChannel.class
*   4,channel的参数设置
*   5，指定处理器
    -   解码处理器TimeDecoder
    -   消息处理器TimeClientHandler
*   6，通过Bootstrap，连接指定的服务器，b.connect(host,port)
    -   6.1, 新建NioSocketChannel : channel
    -   6.2, 初始化channel,
        +  默认handler添加
        +  设置channel的一些参数
    -   6.3, 从workGroup获取一个NioEventLoop,并启动线程，run循环运行，处理task
    -   6.4, 给eventloop添加一个registTask
    -   6.5, eventloop线程异步执行registTask
        +   在eventloop的Selector上注册，不关注任何事件，返回selectionKey给channel,channel就可以通过selectionKey注册自己感兴趣的事件
        +   更新bootStartp的注册状态，标记已经注册完成
        +   给eventLoop添加一个连接任务connectTask
        +   channel的pipeline上添加的handler
链式调用handler的channelRegistered
            *   TimeDecoder.channelRegistered，没有处理
            *   TimeClientHandler.channelRegistered
    -   6.6, eventloop线程异步执行connectTask  
        +   Channel 通过socket连接服务器
        +   在eventloop上注册selectKey
添加连接connect事件
    -   6.7 connect 就绪事件,eventloop线程异步执行
        +   SocketChannelImpl调用finishConnect
    _   6.8, read 就绪事件,eventloop线程异步执行
        +   TimeDecode handler的channelRead,将输入字节转换时间对象
        +   TimeClientHandler的channelRead,读取信息消费，打印服务器发送过来的内容


####    channel 读写处理流程--服务端
*   服务端，serverSocketChannel accept客户端SocketChannel
*   给NioServerSocketChannel的pipeline触发一个读取事件，fireChannelRead，把客户端连接传输给pipeline的handler处理链一个个处理
*   首先进入ServerBootstrap的channelRead
    -   分配一个workGroup的eventloop 
    -   把accept的客户单channel注册到workGroup的eventloop
    -   启动eventloop线程，准备接受和处理客户单channel的读写请求
*   socketChannel的pipeline触发fireChannelActive
*   链式调用handler的channelActive
    -   TimeEncoder.channelActive,默认实现
    -   TimeServerHandler.channelActive，写一个时间对象，ctx.writeAndFlush(new UnixTime())
*   ctx.writeAndFlush触发invokeWrite0，链式调用handler的write
    -   TimeServerHandler context的write,默认实现
    -   TimeEncoder context的write, 最终调用TimeEncoder的重写的encode方法实现编码
*   channel将信息返回

####   DefaultChannelPromise 异步处理协调
*   应用场景
    -   1，promise添加listener,ChannelFutureListener，观察者模式
    -   2，主线程promise.sync，线程等待wait()
    -   3，eventloop异步处理，channel 注册到eventloop的selector完成
    -   4，eventloop异步处理，safeSetSuccess，调用所有注册的listener的operationComplete，
    -   5，eventloop异步处理，建立连接完成后，promise.trySuccess，主线程唤醒
    -   6，主线程从promise获取连接完成的channel


## 参考资料
*   [Netty 4.x学习笔记 - 线程模型](http://yihongwei.com/2014/01/netty-4-x-thread-model/)
*   [Netty 4.x 用户指南](http://wiki.jikexueyuan.com/project/netty-4-user-guide/writing-discard-server.html)
*   Netty
*   https://www.jianshu.com/p/f16698aa8be2
*   [源码学习](https://github.com/code4craft/netty-learning)
*   [官网](https://netty.io/wiki/user-guide-for-4.x.html)