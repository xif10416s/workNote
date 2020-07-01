#   zookeeper
*   https://www.cnblogs.com/felixzh/p/5869212.html

## 实现功能
*   Zookeeper通知机制
    *   客户端注册监听它关心的目录节点，当目录节点发生变化（数据改变、被删除、子目录节点增加删除）时，zookeeper会通知客户端。
*   Zookeeper命名服务
    -   典型应用dubbo,注册中心，服务发现
*   Zookeeper的配置管理
    -   现在把这些配置全部放到zookeeper上去，保存在 Zookeeper 的某个目录节点中，然后所有相关应用程序对这个目录节点进行监听，一旦配置信息发生变化，每个应用程序就会收到 Zookeeper 的通知，然后从 Zookeeper 获取新的配置信息应用到系统中就好
*   Zookeeper集群管理
    -   分布式服务协调,主从选举
    -   临时顺序编号目录节点，每次选取编号最小的机器作为master就好。
*   Zookeeper分布式锁
    -   在指定目录创建节点，创建成功获取锁，用完删除节点释放锁





####  参考

* [问题](https://mp.weixin.qq.com/s?__biz=MzU3NTE2NzAxNQ==&mid=2247485984&idx=1&sn=0d43762daaaad896817ff4dfaeec8ffe&chksm=fd260568ca518c7e1df050e4d9a49938857e0c7524e064458e50bb6255ac588fdaf539345820&scene=126&sessionid=1592180847&key=6f737cad8e87f27e2e138fcc00c13b1658059a9e59bac083e3239164c906d68d76e9db98c295b633493e663f661e3fa7edef9fda33dabf40d3f10193222921a1877c3628c23ce741dd853f1d282342c8&ascene=1&uin=Mjk1NTAwNzcwMg%3D%3D&devicetype=Windows+10+x64&version=62090070&lang=zh_CN&exportkey=Abmlph0U3SqzcroShXm9r8Y%3D&pass_ticket=nduiLnnAugpuAKn5KpxvtrA1fEs5uu1JHiSG7daQkpVksaCPstZP%2FOpxqJ1lqxL2)