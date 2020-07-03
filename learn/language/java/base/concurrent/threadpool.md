####  ThreadPoolExecutor

#####  核心参数

| corePoolSize    | 核心线程数，连接池维持的最低线程数量                         |
| --------------- | ------------------------------------------------------------ |
| maximumPoolSize | 允许的最大的线程数                                           |
| keepAliveTime   | 线程最大空闲时间，大于核心线程数的线程都会被销毁             |
| unit            | keepAliveTime的时间单位                                      |
| workQueue       | 提交任务的等待队列，主要实现：LinkedBlockingQueue，SynchronousQueue |
| threadFactory   | 用户创建线程的工厂方法，可以统一其名称前缀后缀，方便调试追踪问题 |
| handler         | 队列容量上限时拒绝策略                                       |
|                 |                                                              |



* 推荐自定义实现threadFactory，指定线程启动是的名称，方便调试追踪
* BlockingQueue<Runnable> workQueue ：工作队列
  * ArrayBlockingQueue：FIFO有界阻塞队列，初始化容量后，不能更改。
    * 在队列全满时执行入队将会阻塞，在队列为空时出队同样将会阻塞。
    * 并发阻塞是通过ReentrantLock和Condition来实现的，ArrayBlockingQueue内部只有一把锁，意味着同一时刻只有一个线程能进行入队或者出队的操作。
  * LinkedBlockingQueue：
    * 两把锁，一把锁用于入队，一把锁用于出队
* RejectedExecutionHandler -- 当执行队列满等无法执行新任务时
  * AbortPolicy ： 直接抛出异常终止
  * CallerRunsPolicy：直接在调用线程执行run方法
  * DiscardPolicy: 直接忽略丢弃



```
// 执行任务
public void execute(Runnable command) {
        if (command == null)
            throw new NullPointerException();
        /*
         * Proceed in 3 steps:
         *
         * 1. If fewer than corePoolSize threads are running, try to
         * start a new thread with the given command as its first
         * task.  The call to addWorker atomically checks runState and
         * workerCount, and so prevents false alarms that would add
         * threads when it shouldn't, by returning false.
         *
         * 2. If a task can be successfully queued, then we still need
         * to double-check whether we should have added a thread
         * (because existing ones died since last checking) or that
         * the pool shut down since entry into this method. So we
         * recheck state and if necessary roll back the enqueuing if
         * stopped, or start a new thread if there are none.
         *
         * 3. If we cannot queue task, then we try to add a new
         * thread.  If it fails, we know we are shut down or saturated
         * and so reject the task.
         */
        int c = ctl.get();
        // 如果工作线程小于核心线程数
        if (workerCountOf(c) < corePoolSize) {
            // 添加工作线程，任务交由新建工作线程执行
            if (addWorker(command, true))
                return;
            c = ctl.get();
        }
        // 当前工作线程数大于核心线程数，添加到工作队列
        if (isRunning(c) && workQueue.offer(command)) {
            int recheck = ctl.get();
            if (! isRunning(recheck) && remove(command))
                reject(command);
            else if (workerCountOf(recheck) == 0)
                addWorker(null, false);
        }
        else if (!addWorker(command, false))
            reject(command);
    }
```



