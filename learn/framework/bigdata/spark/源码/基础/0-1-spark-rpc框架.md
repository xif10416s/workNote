# spark  内置rpc框架
##  主要组件
*   TransportContext 
    -   主要功能
        +   传输上下文信息 ，包含 用于创建TransportServer 和 TransportClientFactory的上下文
        +   通过TransportChannelHandler设置 Netty Channel pipelines
            *   关联requestId与回调函数
    -   组件
        +   TransportConf -- 传输上下文的配置信息
        +   RpcHandler -- 对客户端请求消息进行处理
            *   接受TransportClient发送的消息，并处理
        +   MessageEncoder -- 消息编码后放入管道
        +   MessageDecoder -- 对管道读取数据解析
    -   主要方法
        +   TransportServer createServer（）
            *   创建rpc框架的服务端，提供高效，低级别的流服务
        +   TransportClientFactory createClientFactory()
            *   TransportClient创建的工厂方法
            *   内部维持一个连接池，TransportClient[] clients，相同host返回相同client
            *   贡献同一个线程
        +   TransportChannelHandler initializePipeline(SocketChannel channel) 
            *   创建一个channelHandler，可以是服务端的handler,也是是client的，用来处理请求和响应数据
            *   初始化pipeline的encoder，decoder操作
*   TransportClient
    -   主要功能
        -   Rpc框架的客户端，用户获取预先协商好的流中的连续数据块
        -   目的是高效的传输大量数据，数据将被查分成几百kb到几mb的块
    -   主要组件：
        +   Channel
            *   netty的channel
        +   TransportResponseHandler
            *   处理来自服务端的响应数据
    -   主要方法：
        +   fetchChunk
            *   从远端请求一个数据块
            *   块索引从0开始，可以多次请求同一个块
            *   主要操作：
                -   生成StreamChunkId
                -   将StreamChunkId和callback对应关系记录到outstandingFetches的map中
                -   通过channel发送fetch请求
                    +   监听执行结果，成功的情况打印日志
        +   stream
            *   根据指定的id从远端获取stream
            *   操作基本同上,发送的请求类型不同StreamRequest
        +   sendRpc
            *   发送rpc请求到远程server端
            *   操作基本同上，发送的请求类型不同RpcRequest
        +   sendRpcSync
        +   send
            *   OneWayMessage
*   TransportServer
    -   主要成员变量：
        -   List<TransportServerBootstrap> bootstraps
    -   主要方法：
        +   init
            *   初始化netty server =>ServerBootstrap
*   ServerBootstrap
