####  动态资源配置（CDH 6.1.0)

#####  基础介绍

*  根据不同项目或不同用户，对yarn资源队列进行划分，达到资源管控，任务管控的目的
* yarn资源池可以嵌套，可以在父资源池的基础上，更细粒度资源分配池
* 默认有一个名为root.default 的资源池，所有yarn applications运行在这个资源池。
* root.users资源池，是一个父资源池,通过root.users.[username]规则生成子资源池，username为系统提交任务的用户账号
  * root用户提交为root.users.root,  hdfs 用户提交 root.users.hdfs
  * 初始不存在，一旦创建就一直存在
* 提交时指定资源池：
  * hadoop mapreduce任务通过指定mapreduce.job.queuename参数指定资源池，不存在则新建
    * hadoop jar -Dmapreduce.job.queuename=etl xxx.jar  
  * spark 通过 spark.yarn.queue指定
    * spark-shell  --conf.yarn.queue=test



#####  配置管理入口（**Clusters** > **Cluster name** > **Dynamic Resource Pool Configuration**）



#####  队列执行策略

* Dominant Resource Fairness(DRF) ： 默认推荐， DRF根据那些资源的可用性和作业要求来确定CPU和内存资源的份额。
* fair : 根据内存资源公平分配。
* FIFO: 基于顺序，先进先出



#####  资源抢占

* 允许抢占其他资源池的空闲资源，保证资源的利用率



#####  ACLs配置 

* 配置那些用户和组可以提交和终止资源池中的程序
* 逗号分隔允许的用户和用户组 配置



#####  动态资源配置

* 基础参数配置：  **Cloudera Manager -> 集群 -> Yarn -> 配置** 中直接搜索配置项

  * | 是否允许创建未定义的资源池。true: yarn会自动创建任务中指定的但是还没有定义过的资源池；false:指定的未定义的资源池无效，会被划分到default资源池中 |
    | ------------------------------------------------------------ |
    | `yarn.scheduler.fair.allow-undeclared-pools = false`         |
    | `# 禁止公平调度器抢占`，开启资源抢占                         |
    | `yarn.scheduler.fair.preemption = false`                     |
    | `# 禁止以用户名作为默认队列`                                 |
    | `yarn.scheduler.fair.user-as-default-queue = false`          |
    | `# 禁用，以使队列内部分配资源的时候，采用轮询方式公平地为每个应用分配资源。若设为true，则可以设置权重，权重越大，分配的资源越多` |
    | `yarn.scheduler.fair.sizebasedweight = false`                |
    | `# 开启Yarn的ACL`， 可以配置那些用户和组可以在任何池中提交和终止应用程序 |
    | `yarn.acl.enable=true` <br />yarn.admin.acl=*   //默认所有用户 |

    

  * 配置重启后，之前动态新建的资源池被清空

  * 执行 spark-shell  --conf.yarn.queue=test   ；  spark-shell  测试后，

    * root.test ,  root.users.hdfs 资源池还是创建了  <== TODO ??

* 新建资源池 

  * Resource Limits 资源配置
    * 资源池名称
    * 最大资源百分比
    * 权重，整体资源的百分比
  * 提交/管理访问控制
    * 在ACL 为true 且 管理 ACL 不是 * 的时候，可以指定用户或

* 创建计划规则  --   根据时间的匹配资源池规则

  *  使用场景： 白天一些实时分析任务需要更多的资源，夜里跑一些离线批量任务需要更多的资源
  * 根据不同的时间段配置给不同资源池配置不同的资源比例





####  参考

* https://www.cnblogs.com/yangshibiao/p/10870846.html
* 资源配置案例  https://blog.luojilab.com/2019/08/27/big-data/tutorial-cm-yarn-hive-sentry-hue/
* 参数配置 https://docs.informatica.com/ja_jp/big-data-management/big-data-management/10-1-1-update-2/_user-guide_big-data-management_10-1-1-update-2_ditamap/GUID-21AC7103-48E7-4307-8640-403F21909A73/GUID-7D6756B1-94E0-44C9-A6BC-B8BDFBD9697D/GUID-9B93A8BD-5439-4DC2-B122-84ECD0721446/GUID-EF872EDC-E3E0-431C-BCFD-C980086C6709.html
* http://lxw1234.com/archives/2015/10/536.htm

