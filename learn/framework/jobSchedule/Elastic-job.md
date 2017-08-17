#Elastic-job
*   分布式调度解决方案，由两个独立子项目：
    -   Elastic-job-lite
        +   轻量级去中心化解决方案，使用jar包形式提供分布式任务协调服务
    -   elactic-job-cloud
        +   使用Mesos+docker的解决方案，额外提供资源治理，应用分发以及进程隔离服务
*   Elastic-job-lite 和 elactic-job-cloud使用同一套api,一次开发就行
