##  spark 通讯模块.md -- (v-2.4.0)
*	不同服务器上的不同角色（Driver,Master,worker)之间相互通信, 通过基于Netty的RPC通信框架实现
	*	性能好--无锁化的串行设计，零拷贝，内存池

###   概要
*  Netty 介绍
*  Spark rpc 主要组件
*  主要应用场景

####  [Netty基础](learn/language/java/base/io/nio框架/netty.md)

####  spark rpc 主要组件及功能介绍（在common模块下） 
*	org.apache.spark.network.TransportContext
	*	创建TransportServer 与 TransportClientFactory 所需要的上下文信息，通过TransportChannelHandler 设置 netty channel 管道。
*	TransportClientFactory ： 用户创建TransportClient的工厂类
	*	维护一个与其他主机的连接池，相同主机返回相同的TransportClient
	*	为所有TransportClient，提供一个共享的线程池
*	TransportClient： 发送请求到服务器，并处理服务器的响应消息，线程安全，可以多线程调用
	*	提供了两种通讯协议
		*	control-plane RPCs （面向控制）：  用于远程调用，信息交互，调用频繁，数据量小
			*	client.sendRPC(new OpenFile("/foo")) 
		*	data-plane（面向数据传输） ： 用于大数据块传输	
			*	client.fetchChunk(streamId = 100, chunkIndex = 0, callback)
	*	包含一个Channel : 与指定服务器的链路连接
	*	TransportResponseHandler ： 处理服务器的响应消息
*	TransportServer： 通常netty server 启动流程
	*	netty 的server  接收并处理来自客户端的请求
	*	通过传入特定的RpcHandler 实现业务逻辑处理
	*	List<TransportServerBootstrap> bootstraps： 当客户端连接时，一系列自定义客户端channel设置，如SASL权限认证等
*	TransportChannelHandler： 用于投递request给 TransportRequestHandler ， 以及 响应消息给 TransportResponseHandler
	*	TransportServer 与 TransportClientFactory 为每一个 channel 创建 TransportChannelHandler 
		*	客户端 通过 netty channel 用来发起 RequestMessage  ， 在服务端被处理该请求消息
		*	服务器端 产生一个 ResponseMessage ， 客户端会接收并处理
		*	服务器端和客户端可以相互通信，都会有发送消息和接收响应消息的处理
	*	每个TransportChannelHandler包含一个TransportClient，可以让服务端返回消息给客户端


####  主要应用场景











### 参考：
