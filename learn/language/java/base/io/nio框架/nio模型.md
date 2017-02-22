##前提
*   大多数程序 = 输入+ 处理 + 输出 ，离不开I/O操作
    -   输入/输出
        +   文件系统
        +   网络
    -   I/O操作 - Java Nio 模型（I/O复用）
        1.  应用程序请求读取数据操作，调用系统内核
            2.  系统内核准备数据，完成后返回就绪状态
        3.  应用程序调用系统内核复制数据，复制到应用程序缓冲区
            4.  系统内核复制完成，返回就绪状态
        4.  处理缓冲区数据
            
*   CPU的处理速度是要远远快于IO速度，如果CPU为了IO操作（例如从Socket读取一段数据）而阻塞显然是不划算的
*   多进程或者线程去进行处理，进程切换的开销
*   事件驱动，或者叫回调的方式 =》 
    -   应用业务向一个中间人注册一个回调（event handler），当IO就绪后，就这个中间人产生一个事件，并通知此handler进行处理。
    -   找工作
        +   联系中介（中间人）留个号码（注册一个回调）
        +   中间人发现有新职位了，打电话通知求职者
*   基本处理
    1.  读取请求
    2.  解码
    3.  处理
    4.  编码
    5.  返回
*   分而治之是通常处理扩展的方式
    -   分解处理逻辑，变成一些小的任务，每个任务执行不需要阻塞
    -   执行任何可以执行的任务


##线程模型
*   hread per Connection: 在没有nio之前，这是传统的java网络编程方案所采用的线程模型。即有一个主循环，socket.accept阻塞等待，当建立连接后，创建新的线程/从线程池中取一个，把该socket连接交由新线程全权处理。这种方案优缺点都很明显，优点即实现简单，缺点则是方案的伸缩性受到线程数的限制。
    -   ![](http://img.blog.csdn.net/20140807141316765?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvaXRfbWFu/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)
*   Reactor in Single Thread: 有了nio后，可以采用IO多路复用机制了。我们抽取出一个单线程版的reactor模型，时序图见下文，该方案只有一个线程，所有的socket连接均注册在了该reactor上，由一个线程全权负责所有的任务。它实现简单，且不受线程数的限制。这种方案受限于使用场景，仅适合于IO密集的应用，不太适合CPU密集的应用，且适合于CPU资源紧张的应用上。
*   Reactor + Thread Pool: 方案2由于受限于使用场景，但为了可以更充分的使用CPU资源，抽取出一个逻辑处理线程池。reactor仅负责IO任务，线程池负责所有其它逻辑的处理。虽然该方案可以充分利用CPU资源，但是这个方案多了进出thread pool的两次上下文切换。
*   Reactors in threads: 基于方案3缺点的考虑，将reactor分成两个部分。main reactor负责连接任务（accept、connect等），sub reactor负责IO、逻辑任务，即mina与netty的线程模型。该方案适应性十分强，可以调整sub reactor的数量适应CPU资源紧张的应用；同时CPU密集型任务时，又可以在业务处理逻辑中将任务交由线程池处理，如方案5。该方案有一个不太明显的缺点，即session没有分优先级，所有session平等对待均分到所有的线程中，这样可能会导致优先级低耗资源的session堵塞高优先级的session，但似乎netty与mina并没有针对这个做优化。


 
##Reactor模式[Netty源码解读（四）Netty与Reactor模式](http://www.blogjava.net/DLevin/archive/2015/09/02/427045.html) </br>
[reactor and nio](http://gee.cs.oswego.edu/dl/cpjslides/nio.pdf)</br>
[中文](http://blog.csdn.net/yangzishiw/article/details/53242103)
*   Reactor模式主要是提高系统的吞吐量，在有限的资源下处理更多的事情。
*   角色
    -   Reactor EventLoop及时响应相对于的读/写IO事件，派发给合适的handler处理，
        +   io分两步
            *   等待就绪->就绪状态 
            *   复制数据->复制完成
    -   Handlers 处理非阻塞读/写IO事件所对应的业务逻辑
    -   管理IO读/写事件到对应处理器的绑定
*   由一个不断等待和循环的单独进程（线程）来做这件事，它接受所有handler的注册，并负责先操作系统查询IO是否就绪，在就绪后就调用指定handler进行处理，这个角色的名字就叫做Reactor。
```
public void run() {
    try {
        while (!Thread.interrupted()) {
            selector.select();
            Set selected = selector.selectedKeys();
            Iterator it = selected.iterator();
            while (it.hasNext())
                dispatch((SelectionKey) (it.next()));
            selected.clear();
        }
    } catch (IOException ex) { /* ... */
    }
}

```
*   ![](http://img3.tbcdn.cn/L1/461/1/826b5df93974b1fc1269e92e815760e3817a2c50?spm=5176.100239.blogcont2371.8.VAXG9X)
*   Reactor模式里，操作系统只负责通知IO就绪，具体的IO操作（例如读写）仍然是要在业务进程里阻塞的去做的
*   Proactor模式，由操作系统将IO操作执行好（例如读取，会将数据直接读到内存buffer中），而handler只负责处理自己的逻辑，真正做到了IO与程序处理异步执行。
*   Reactor是同步IO，Proactor是异步IO。
*   模式：
    -   single thread version 
        +   ![](http://cejdh.img47.wal8.com/img47/533449_20151202165458/145085907164.png)
        +   单线程版Reactor模式是最基本的实现，其核心就是单线程Reactor的EventLoop在不断处理被Selector检测到的IO事件，但缺点也显而易见：
            -   随着客户端的连接数目的增加，如果业务的处理也需要消耗不小时间的话，那仅仅单次的EventLoop循环都会消耗不少时间才能进入下一次循环，导致IO事件阻塞在Selector里不能被及时轮询处理到
            -   而且随着多核CPU的爆发，当拥有多核机器时，应当适当利用多线程能力来分担本来是单线程的Rector，以去应对更多的客户端连接，否则依旧是单线程Rector的话，岂不是浪费了多核这个潮流强项了？
    -   Worker Thread Pools </br>
    ![](http://my.csdn.net/uploads/201207/22/1342924397_5725.png)
        -   reactor + thread pool（能适应密集计算）
        -   引入了Worker Threads（工人线程，消费线程，即有一群工人老早就做好准备处理即将到来任务了）——线程池；理由是Reactor的EventLoop轮询应当快速响应IO触发事件，而不应当消耗在本应该是任务处理器处理的业务上： 
        -   在单线程Reactor的基础上把非IO相关的业务处理部分（decode、computer和encode）拆出来封装成为一个单独的任务（Runnable/命令模式），如此一来在线程池中就能立即进行计算处理了
    -   Using Multiple Reactors  </br>
    ![](http://img.blog.csdn.net/20131104212207218?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvam51X3NpbWJh/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)
        -   着客户端连接越来越多，单个Reactor线程处理IO能力会达到饱和状态，在多核机器上看到的现象是只有一个核心利用率较高，其他核心是闲置的，所以应当适当利用多核优势，扩展成匹配CPU核数的多个Reactor，达到分担IO负载的目的： 
        -   多Reactor根据职责划分为1个mainReactor和多个subReactors，mainReactor主要负责接收客户端连接，因为TCP初始需要经历3次握手才能确认连接，这个连接过程的消耗在客户端较多时其开销是不小的，单独使用mainReactor处理保证了其他已经连接上的客户端在subReactors中不受其影响，从而快速响应处理业务，以此分摊负载并提高系统整体系能
        
