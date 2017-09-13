#   CAT
##   https://github.com/dianping/cat
##  https://tech.meituan.com/CAT_in_Depth_Java_Application_Monitoring.html

##  xml
### server
*   client.xml
```
<server ip="192.168.2.100" port="2280" http-port="2281" />
<server ip="192.168.2.110" port="2280" http-port="2281" />
<server ip="192.168.2.120" port="2280" http-port="2281" />
```
   
*   server.xml
```
<remote-servers>192.168.2.100:2281,192.168.2.110:2281,192.168.2.120:2281</remote-servers>
remote-servers : 定义HTTP服务列表，（远程监听端同步更新服务端信息即取此值）
```

### client
*   client.xml
```
<config mode="client">
     <servers>
        <server ip="192.168.2.100" port="2280" /> 
        <server ip="192.168.2.110" port="2280" /> 
        <server ip="192.168.2.120" port="2280" /> 
    </servers> 
</config>

```

scp -P 1224 -r data xif@192.168.2.110:/home/xif/data

http://192.168.2.100:2281/cat/s/config?op=routerConfigUpdate

##   启动一台服务端10.1.1.1，修改服务端路由文件，url地址 http://10.1.1.1:8080/cat/s/config?op=routerConfigUpdate ,需要用户名密码登陆，如果配置ldap即可直接登陆，或者用默认账号catadmin/catadmin登陆。job-machine=true，alert-machine=true，让这台机器进行后续job以及告警处理，这些都可能影响到consumer性能。

<?xml version="1.0" encoding="utf-8"?>
<router-config backup-server="192.168.2.100" backup-server-port="2280">
   <default-server id="192.168.2.110" weight="1.0" port="2280" enable="true"/>
   <default-server id="192.168.2.120" weight="1.0" port="2280" enable="true"/>
</router-config>


### 修改服务端路由文件


### job-machine
*   成汇总报告和统计报告的任务
*   DefaultTaskConsumer


### alert-machine


### MessageConsumer
*   TcpSocketReceiver ,io.netty.channel.
    -   MessageHandler.handle
        +   RealtimeConsumer .consume
            *   DefaultMessageQueue.offer
            -   PeriodTask.run
                +   MessageAnalyzer.analyze
*   mysql
    *   TaskManager.createTask
    *   TransactionDelegate.createHourlyTask
    *   TransactionAnalyzer.doCheckpoint
        -    StoragePolicy.FILE_AND_DB //!isLocalMode()
    *   DefaultReportManager.storeHourlyReports
        -   

## client
### TcpSocketSender




### CatHomeModule
*   成汇总报告和统计报告的任务
*   InstallMojo
*   cat-client-config-file


### CAT的报表的存储

### CAT原始logview的存储。



### 集群状态



### 消息ID的设计
CAT每个消息都有一个唯一的ID，这个ID在客户端生成，后续都通过这个ID在进行消息内容的查找。典型的RPC消息串起来的问题，比如A调用B的时候，在A这端生成一个Message-ID，在A调用B的过程中，将Message-ID作为调用传递到B端，在B执行过程中，B用context传递的Message-ID作为当前监控消息的Message-ID。


### 分布式调用链监控
每个Transaction需要三个ID：

RootId，用于标识唯一的一个调用链
ParentId，父Id是谁？谁在调用我
ChildId，我在调用谁?


### 缺点
*   一个小时精度，不能看某次请求
*   cxf集成复杂

            