####  AQS(AbstractQueuedSynchronizer)
* `AQS`即是抽象的队列式的同步器，内部定义了很多锁相关的方法，我们熟知的`ReentrantLock`、`ReentrantReadWriteLock`、`CountDownLatch`、`Semaphore`等都是基于`AQS`来实现的
* 一个基于队列实现的阻塞锁以及相关同步操作（信号量，事件等）的基础框架
* AQS提供了一种原子式管理同步状态、阻塞和唤醒线程功能以及队列模型的简单框架。
* `AQS`中 维护了一个`volatile int state`（代表共享资源）和一个`FIFO`线程等待队列（多线程争用资源被阻塞时会进入此队列）。
  * 这里`volatile`能够保证多线程下的可见性，当`state=1`则代表当前对象锁已经被占有，其他线程来加锁时则会失败，加锁失败的线程会被放入一个`FIFO`的等待队列中，会被`UNSAFE.park()`操作挂起，等待其他获取锁的线程释放锁才能够被唤醒。
  * `state`的操作都是通过`CAS`来保证其并发修改的安全性。
* 主要组件：
  * AQS内部Node类
    * CLH队列节点
    * 在节点中用"status"字段来跟踪每个节点是否处于阻塞状态
    * 每个节点在在其前面一个节点释放同步状态后，前驱节点将会通知该节点去获取同步状态--即：发出信号通知该节点、、、
    * CLH队列中的每个节点中的线程都有机会尝试获取同步状态，但是并不能保证一定获取成功
    
  * CLH队列
    * CLH(Craig, Landin, andHagersten)根据三个人的名字命名的队列
    * 它是队列锁的一个变种(variant)，通常用于自旋锁(这个源码中会有体现),用它代替阻塞同步器
    * CLH要比MCS更适合处理取消和超时
    *  CLH是一个双向链表队列
    * 加入到CLH队列中的节点会被加入到队列的尾部，并保证加入时的原子性
    * 节点弹出队列的时候只需要将头节点head移除即可
    * 队列中的每个节点都有一个唯一的线程，用该线程来获取同步状态
    
  * LockSupport
  
    * park -- 挂起线程
  
    * ```
      unpark(Thread thread) 唤醒指定线程
      ```

#####  基本功能

* 同步器至少要有以下两种类型的方法acquire和release
  * **acquire**：至少要有一个操作能实现对调用线程的阻塞，直到同步器允许它进行操作
  * **release**：至少要有一个操作能用一种方式解锁一个或者更多个已经阻塞的线程改变同步状态
  * 非阻塞式的同步过程尝试(tryLock)
  * 可选的超时机制，可以允许程序放弃等待
  * 可以通过中断执行取消
* 而为了适应不同的同步器，同步器要支持两种模式
  * **独占式**exclusive。要保证一次只有一个线程可以经过阻塞点
  * **共享式**shared。可以允许多个线程阻塞点



####  公平锁 & 非公平锁

* **非公平锁**和**公平锁**的区别：**非公平锁**性能高于**公平锁**性能。**非公平锁**可以减少`CPU`唤醒线程的开销，整体的吞吐效率会高点，`CPU`也不必取唤醒所有线程，会减少唤起线程的数量
* **非公平锁**性能虽然优于**公平锁**，但是会存在导致**线程饥饿**的情况。在最坏的情况下，可能存在某个线程**一直获取不到锁**。不过相比性能而言，饥饿问题可以暂时忽略，这可能就是`ReentrantLock`默认创建非公平锁的原因之一了。



#####  主要实现

* ReentrantLock
  * 对比Synchronized :
  * Synchronized 无法设置超时时间，ReentrantLock可以设置获取锁的超时时间
    
    * Synchronized 无法实现公平锁，ReentrantLock 可以实现公平锁
  * ##### **Condition 简介**
    * Condition`是在`java 1.5`中才出现的，它用来替代传统的`Object`的`wait()`、`notify()`实现线程间的协作，相比使用`Object`的`wait()`、`notify()`，使用`Condition`中的`await()`、`signal()`这种方式实现线程间协作更加安全和高效。因此通常来说比较推荐使用`Condition
    * condition 增加一个condition队列
    * condition不会产生死锁，底层是park/unpark实现
  
* CountDownLatch -

  * 通过一个计数器，初始值为线程数，当线程执行完成，计数器就减1，当所有线程执行完成，计数器为0时，继续执行

* Cyclicbattier

  * 类似CountDownLatch ，协调重循环通过一个屏障
  * CountDownLatch基于AQS的共享模式，而CycliBarrier基于Condition实现









#### 参考

* https://www.jianshu.com/p/6a77f60fd236
* https://www.jianshu.com/p/d2e5965631dc
* https://www.jianshu.com/p/63588ebea397
* https://blog.csdn.net/aesop_wubo/article/details/7533186
* https://www.jianshu.com/p/0f876ead2846
* https://mp.weixin.qq.com/s?__biz=MzU0OTk3ODQ3Ng==&mid=2247487474&idx=1&sn=dbeb425948d994c6eb4743a2353bdcb0&chksm=fba6e7f1ccd16ee758c73c870fd0a8cb345b6093d711338315f83d906735f31171b09c411d3c&scene=126&sessionid=1589595386&key=56c677ebc18a7bd890239b1e209dca2fd0b8b52c1deb9afcd9295d2c43e409334c86cf277a78ca46551de0eb670fd19481dce0147c0cdf0fafb05914c039f0336fc0e274f9ce5ed4c32cfb9fa3d58ac7&ascene=1&uin=Mjk1NTAwNzcwMg%3D%3D&devicetype=Windows+10+x64&version=62090070&lang=zh_CN&exportkey=AbKYzAzhKU0yFyDJfYfG53I%3D&pass_ticket=YTUVElEEK0KAGoYmxKhwAjnTVzcmdCMrE9AGqeU0qacdBk2qb2tb9IXdcPw7ZtvT