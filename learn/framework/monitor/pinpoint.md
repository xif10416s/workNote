#   pinpoint -- https://github.com/naver/pinpoint/tree/master/quickstart

## http://www.cnblogs.com/yyhh/p/6106472.html
##  hbase 
*   1.2.6 download
*   修改hbase-env.sh的JAVA_HOME环境变量位置
    -   cd /data/service/hbase/conf/ & vi hbase-env.sh
    -   export JAVA_HOME=/usr/java/jdk17/
*   修改Hbase的配置信息
    -   vi hbase-site.xml
        +   `<configuration>
  <property>
    <name>hbase.rootdir</name>
    <value>file:///data/hbase</value>
 </property>
</configuration>`
*   启动hbase
    -   cd /opt/hbase/hbase/ & ./bin/start-hbase.sh
*   初始化Hbase的pinpoint库
    -   ./hbase shell /opt/pinpoint/quickstart/conf/hbase/init-hbase.txt    
*   查看HBase的数据是否初始化成功
    -   http://localhost:16010/master-status
    -   http://192.168.70.176:16010/master-status#storeStats


##  pinpoint
*   git download
*   JAVA_HOME environment variable set to JDK 7+ home directory.
*   JAVA_6_HOME environment variable set to JDK 6 home directory.
*   JAVA_7_HOME environment variable set to JDK 7 home directory.
*   JAVA_8_HOME environment variable set to JDK 8 home directory.
*   ./mvnw clean  -Dmaven.test.skip=true
*   ./mvnw  install -Dmaven.test.skip=true

### Collector 
*   quickstart/bin/start-collector.sh &

### TestApp
*   quickstart/bin/start-testapp.sh &
    -   http://192.168.70.176:28081/

### Web UI 
*   quickstart/bin/start-web.sh &
    *   http://192.168.70.176:28080/

### 配置app
*   配置pp-agent采集器
    -   解压pp-agent
        +   `cd /home/pp_test
tar -zxvf pinpoint-agent-1.5.2.tar.gz
mv pinpoint-agent-1.5.2 /data/pp-agent`
    -   编辑配置文件 pinpoint.config
    -   只需要指定到安装pp-col的IP就行了，安装pp-col启动后，自动就开启了9994，9995，9996的端口了。这里就不需要操心了，如果有端口需求，要去pp-col的配置文件("pp-col/webapps/ROOT/WEB-INF/classes/pinpoint-collector.properties")中
    -   `profiler.collector.ip=192.168.245.136`
    -   udp端口 
        +   profiler.collector.span.port=9996 // udp端口
        +   profiler.collector.stat.port=9995

# placeHolder support "${key}"
profiler.collector.stat.ip=${profiler.collector.ip}
profiler.collector.stat.port=9995
*   修改测试项目下的tomcat启动文件"catalina.sh"
    -   vi catalina.sh
```    
    第一行是pp-agent的jar包位置
    第二行是agent的ID，这个ID是唯一的，我是用pp + 今天的日期命名的，只要与其他的项目的ID不重复就好了
    第三行是采集项目的名字，这个名字可以随便取，只要各个项目不重复就好了 

    CATALINA_OPTS="$CATALINA_OPTS -javaagent:/data/pp-agent/pinpoint-bootstrap-1.5.2.jar"
    CATALINA_OPTS="$CATALINA_OPTS -Dpinpoint.agentId=pp20161122"
    CATALINA_OPTS="$CATALINA_OPTS -Dpinpoint.applicationName=MyTestPP

```

## start
*   /opt/hbase/hbase/bin/start-hbase.sh
*   /opt/pinpoint/pinpoint-master/quickstart/bin/start-collector.sh &
*   /opt/pinpoint/pinpoint-master/quickstart/bin/start-testapp.sh &
*   /opt/pinpoint/pinpoint-master/quickstart/bin/start-web.sh &


