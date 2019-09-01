##  spark 通讯模块.md -- (v-2.4.0)
*	不同服务器上的不同角色（Driver,Master,worker)之间相互通信, 通过基于Netty的RPC通信框架实现
	*	性能好--无锁化的串行设计，零拷贝，内存池

###   概要
*  Netty 介绍
*  Spark rpc 主要组件
*  主要应用场景

####  [Netty基础](../../../../../../language/java/base/io/nio框架/netty.md)

####  spark rpc 基础组件及功能介绍（在common模块下） 
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

####  spark rpc 封装组件（core模块）
*	RpcEnv: 上层抽象的rpc环境信息，包含主要通信对象
	*	基础组件
		*	RpcEndpoint ： RPC终端对象的抽象，定义了rpc 通信终端的生命周期，以及处理接收到的消息
			*	主要方法：receive，receiveAndReply，由各种业务情况实现逻辑
			*	会有各种各样的RpcEndpoint实现
		*	RpcEndpointRef ： 一个远端RPC终端的引用，定义了一些发送消息的接口，用来给远端发送消息
			*	主要方法： send， ask
			*	NettyRpcEndpointRef实现类，主要是发送消息，基础通信实现
				*	初始化只需要url地址
			*	通过setupEndpointRef初始化并注册，或者从接受到消息中获取 //TODO
	*	setupEndpoint(name: String, endpoint: RpcEndpoint): RpcEndpointRef
		*	给定一个名称，注册该rpc终端，并返回该rpc终端的引用
			*	rpc终端（RpcEndpoint）用来处理接收到的消息，引用（RpcEndpointRef）用来给远端RPC发送消息
*	NettyRpcEnv: netty的RpcEnv实现
	*   主要成员：
		*	dispatcher: Dispatcher 负责接收消息的投递路由
		*	transportContext：TransportContext ，负责netty client ,netty server 的创建和初始化
	*	主要方法：
		*	startServer： 创建netty server 
		*	createClient:  创建 netty client
		*	setupEndpoint ： 注册终端方法
		*	发送消息方法：
			*	send ， ask
*	Inbox &  Outbox
	*	Inbox : 消息接受信箱,相当于一个接收到消息的队列，异步缓冲，提高吞吐量，及并行处理
		*	process方法为具体消息的处理
			*	Dispatcher初始化的时候专门有一个dispatcher-event-loop线程池，会多线程并发处理消息
	*   Outbox : 发送消息信箱
		*	send方法发送消息，outbox内部起了一个线程，将发送信箱的消息发送出去
*	Dispatcher ： 封装的消息路由组件，将接收到的消息传递给合适的 endpoint 处理
	*	维护了一组rpc终端以及终端的引用ref
	*	新的RpcEndpoint 终端会被注册进来
	*	主要方法：
		*	postToAll：给所有注册的终端发送消息
		*	postRemoteMessage ：发送消息给远程终端
		*	postLocalMessage ： 发送消息给本地终端
		*	postMessage ： 发送消息给指定的终端，实际上只是先将消息放入指定终端对于的Inbox接收信箱中
	*	通信方式：
		*	本地发送消息到远端
		*	本地发送消息给本地其他终端
		*	接收本地消息并处理
		*	接收远端消息并处理
*	主要消息对象
	*	InboxMessage ： 收件箱的消息，会被发送到合适的终端处理，主要在Dispatcher中使用
		*	OneWayMessage： 最终调用endpoint.receive，不需要响应消息
		*	RpcMessage ： 最终调用endpoint.receiveAndReply，消息处理后返回响应消息
	*	OutboxMessage
		*   OneWayOutboxMessage
		*	RpcOutboxMessage

####	[基础类图]
![](https://github.com/xif10416s/workNote/blob/master/learn/framework/bigdata/spark/images/rpc_endpoints_loops_class.jpg)	

####	核心流程图
![](../../../images/common_rpc.png)

####  RpcEnv的初始化
*	需要用到 RpcEnv通信的主体对象为：
	*	Master对象(org.apache.spark.deploy.master.Master)，包含一个rpcEnv对象，Master节点的入口类
	*	Worker对象（org.apache.spark.deploy.worker.Worker),包含一个rpcEnv对象，Worker节点的入口类
	*	Driver程序需要初始化SparkContext，其中包含了rpcEnv
*	RpcEndpoint 与 RpcEndpointRef 初始化


####  主要应用案例
#####  driver 初始化与 Master 节点通信，注册到master
*	Driver 启动是需要初始化SparkContext，其中rpcEnv，[SchedulerBackend](https://github.com/xif10416s/workNote/blob/master/learn/framework/bigdata/spark/%E6%BA%90%E7%A0%81/%E5%9F%BA%E7%A1%80/1-SparkContext-%E5%88%9D%E5%A7%8B%E5%8C%96.md)也会初始化
	*	以StandaloneSchedulerBackend为例，启动时会初始化StandaloneAppClient对象rpcEnv被传入该对象，负责与spark  standalone 集群通信
	*	StandaloneAppClient 主要组件
		*	ClientEndpoint 以 AppClient名字被注册到 rpcEnv
		*	ClientEndpoint：
			*	tryRegisterAllMasters：封装了向master 发送RegisterApplication 注册消息
				*	通过rpcEnv.setupEndpointRef(masterAddress, Master.ENDPOINT_NAME) 初始化master endpointRef 并注册到Dispatcher
					*	SparkContext中配置的masterUrl就可以初始化RpcEndpointRef
			*	receive：接收master的响应消息处理，不需要再响应
				*	RegisteredApplication ： 注册成功返回消息--来自master
				*	ApplicationRemoved： master 删除当前driver
				*	ExecutorAdded ， ExecutorUpdated ： executoer信息相关
				*	WorkerRemoved： worker移除消息
			*	receiveAndReply：master响应消息交互
				*	StopAppClient ： master 停止app， 并返回master UnregisterApplication取消注册 -- 来自driver
				*	RequestExecutors ： 向master请求executor -- 来自driver
	*	基础流程
		*	driver一系列初始化，StandaloneAppClient











### 参考：
