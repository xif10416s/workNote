#TOMCAT
## shutdown process
*   ./$CATALINA_HOME/bin/catalina.sh stop 30
*   org.apache.catalina.startup.Bootstrap stop
*   org.apache.catalina.startup.Catalina stop

## 术语
###JNDI
*   (Java Naming and Directory Interface,Java命名和目录接口)
*   查找和访问各种命名和目录服务的通用、统一的接口，类似JDBC都是构建在抽象层上
*   建立起逻辑关联，允许把名称同Java对象或资源关联起来，而不必知道对象或资源的物理ID。
*   用途
    -   命名或目录服务使用户可以集中存储共有信息，这一点在网络应用中是重要的，因为这使得这样的应用更协调、更容易管理

###APR/native
*   除了提供正常的web server服务（HTTP，File I/O, etc.）， Tomcat还提供了一种称之为 Apache Portable Runtime的组件为web应用提供高可扩展性，高性能以及更好的与本地服务技术整合的功能。Tomcat的文档中提到了几种典型的功能，比如sendfile，epoll和OpenSSL等高级IO功能。OS级别的功能如随机数的产生，系统状态等等。

## 代码
---
###LifecycleListeners
####NamingContextListener
*   根据不同生命周期，处理命名服务相关资源

####AprLifecycleListener
*   1. eventType == BEFORE_INIT
    -   a. 初始化并加载org.apache.tomcat.jni.Library, 即APR native Library的jni wrapper。
    -   b. 初始化并加载org.apache.tomcat.jni.SSL, 即SSL的jni wrapper。
*   2. eventType == AFTER_DESTROY
    -   a. 终止org.apache.tomcat.jni.Library
    -   b. 终止org.apache.tomcat.jni.SSL

####JreMemoryLeakPreventionListener
*   各种资源的保护，防止文件锁定，防止内存泄漏

####GlobalResourcesLifecycleListener
*   管理global JNDI resources 

####ThreadLocalLeakPreventionListener
*   避免thread-local相关的内存泄漏

----
###StandardServer
*   这个类继承了Server接口，并实现了其中的管理并维护Servcie及全局的resource的方法。


----
###Catalina
*   作为Tomcat的最重要的引擎类，Catalina还负责对Server进行启动停止等操作。
*   Digester是一个用于处理XML输入数据的处理器，它可以根据一系列的匹配规则来执行相应的操作。在Catalina类中，我们用Digester来读入conf/server.xml
    -   1.为配置文件中的Server标签创建“org.apache.catalina.core.StandardServer” 实例，并用其作为参数调用Catalina中的setServer方法。
    -   2.为Server标签中的GlobalNamingResources标签创建"org.apache.catalina.deploy.NamingResources"实例，并用其作为参数调用StandardServer类中的setGlobalNamingResources方法。
    -   3.为Server标签中的Listener创建对应其className属性（必须是LifecycleLinster的子类）的实例，并用其调用StandardServer类中的addLifecycleListener方法。
    -   4.为Server标签中的Service创建StandardService实例，并用其调用StandardServer类中的addService方法。
    -   6.为Service标签中的Executor创建StandardThreadExecutor实例，并用其调用StandardService类中的addExecutor方法。
    -   7.为Service标签中的Connector创建Connector实例，并用其调用StandardService类中的addConnector方法。同时将之与第六步创建的Executor关联。

```
Server
    -Service
        -Executor
        -Connector
            -CoyoteAdapter
            -Http11NioProtocol
                -NioEndpoint
        -Engine
            -Context
```

###StandardServer
*   servlet 容器
*   包含一个或者多个Services
*   顶级命名资源

###Service
*   包含一个或者多个Connectors，共享一容器

###Connector
*   CoyoteAdapter


----

###StandardThreadExecutor
*   该线程池类通过代理设计模式对Java Concurrent包中的线程池ThreadPoolExecutor进行简单的封装。并实现了Lifecycle接口，以及增加了发送消息的功能。

---
##tomcat 工作线程
###JIoEndpoint
*   处理到来的tcp连接
*   实现一个简单的服务模型
    -   一个监听线程，accept 来自 一个 socket
    -   每次到来一个连接，创建一个新的worker 线程
*   名称是 prefix-address-port，如http-0.0.0.0-8080-1

###JIoEndpoint.Acceptor
*   获得HTTP请求socket。并从工作线程池中获得一个线程，将socket分配给一个工作线程。
*   名称是 -Acceptor-

###RMI TCP CONNECTION-127.0.0.1

###RMI TCP ACCEPT-0

###RMI SCHEDULER

###NioBlocking.BlockPoller

###http-nio-8080-acceptor-0
*   NioEndpoint.Acceptor
*   获得HTTP请求socket。并从工作线程池中获得一个线程，将socket分配给一个工作线程。
*   选择poller ，异步register socket


###http-nio-8080-ClientPoller-0
*   NioEndpoint
*   注册到一个poller，添加到poller 的SynchronizedQueue中，循环处理，注册到socket,selectorKey
*   异步监听事件，最终启动新线程调用Http11ConnectionHandler处理

###http-nio-8080-exec-1
*   SocketProcessor -->Http11ConnectionHandler,处理请求事件

###Catalina-exec-0

###catalina-startstop-0
*   ContainerBase

###ajp-nio-8009-acceptor-0

###ajp-nio-8009-ClientPoller-0


##工作流程
###关闭流程
*   Catalina stop
*   run and remove shutdownHook
*   StandardServer stop
*   StandardService stop
*   Connector pause
*   Http11NioProtocol pause
*   NioEndpoint pause 
    -   Pause the endpoint, which will stop it accepting new connections.
    -   steps
        -   paused = true;
            +   Acceptor.run  
                *   更新状态state = AcceptorState.PAUSED;
                *   进入循环等待，不在接受新的连接
            +   NioEndpoint.Poller线程，进入循环等待
        +   unlockAccept
            *   伪造一个虚假连接，连接服务器
            *   等待1秒钟，所有Acceptor进入非RUNNING状态，acceptor threads to unlock
*   NioEndpoint  stopInternal
    -   LimitLatch.releaseAll()
        +   一个FIFO queue,限制长度
        +   releaseAll =》 sync.releaseShared(0) ，长度为0
    -   running = false; 
        +   Acceptor.run 循环结束，线程执行结束
    -   Poller.destroy
        +   close = true;
            *   events();处理队列中的事件
        +   selector.wakeup();
    -   stopLatch.await，等待所有Poller结束
    -   清理资源
        +   eventCache.clear();
        +   keyCache.clear();
        +   nioChannels.clear();
        +   processorCache.clear();
    -   关闭线程池
        +   executorTerminationTimeoutMillis = 5000 ，默认等待5秒



## 问题
### 同个浏览器多个标签页轮流访问servlet ,无法并发
*   环境
    -   tomcat7，8
    -   intellj idea 
    -   jdk 1.7
    -   浏览器 firefox
*   操作
    -   1，打开浏览器4个标签页同时访问
        +   现象
            *   多个线程轮流执行，无并发
        +   *原因*
            *   同浏览器多个标签发送相同请求，顺序执行，
    -   2，打开不同浏览器，或者两台机器的浏览器访问
        +   现象
            *   可以并发

#### servlet 代码
```
public class TestServlet extends HttpServlet{
    @Override
    protected void doGet(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        System.out.println("test........"+Thread.currentThread().getName() + new Date());
        try {
            Thread.currentThread().sleep(10000l);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        resp.getWriter().write("Ok");
    }
}
```
```
// 操作1 结果，无并发结果
test........http-nio-8080-exec-5Thu Jun 08 11:27:22 CST 2017
test........http-nio-8080-exec-7Thu Jun 08 11:27:32 CST 2017
test........http-nio-8080-exec-8Thu Jun 08 11:27:42 CST 2017
test........http-nio-8080-exec-9Thu Jun 08 11:27:52 CST 2017
```





