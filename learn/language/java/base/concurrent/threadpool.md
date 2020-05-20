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

