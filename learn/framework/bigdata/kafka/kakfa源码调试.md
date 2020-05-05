###   kakfa源码调试

###### 一，环境准备

1. 下载最新源码https://www.apache.org/dyn/closer.cgi?path=/kafka/2.5.0/kafka-2.5.0-src.tgz
2. 下载gradle, https://gradle.org/releases/
3. 进入源码的根路径，使用gradle编译 输入命令：
   1. gradle idea 等待一段时间就可以看到编译成功
   2. gradle wrapper
   3. gitbash
      1. export http_proxy=http://127.0.0.1:1081  设置代理
      2. ./gradlew jar
4.  编译通过
5. 导入idea打开
6. 把client generated/generated-test拷贝到对于client包下，移除src   <== TODO 不专业



###### 二，kafka服务器启动

1. windows环境测试需要下载编译好的kafka_2.11-2.4.1.tgz
   1. 通过：bin/zookeeper-server-start.sh config/zookeeper.properties启动zookeeper
2. kafka服务启动
   1. 主类：kafka.Kafka
   2. 程序参数： config/server.properties
3. 通过windows的bat命令测试
   1. kafka-topics.bat --create --zookeeper localhost:2181 --replication-factor 1 --partitions 4 --topic test2
   2.  查看zookeeper信息：bin/zookeeper-shell.sh localhost:2181



​    



######	出错问题

* Error:(19, 39) java: 程序包org.apache.kafka.common.message不存在.
  * 新版kafka将request和response的格式类改成自动生成了。你可以运行在Kafka源码目录下运行./gradlew重新生成一遍，然后再导入到IDEA
    * windows -- git bash  ./ gradlew
      * 代理设置：export http_proxy=http://127.0.0.1:1081
      * 通过网站https://www.ipaddress.com/查找raw.githubusercontent.com的ip地址
      * 修改hosts  199.232.68.133  raw.githubusercontent.com
        * c:\windows\system32\drivers\etc
* 执行.gradlew   报错  找不到 org.gradle.wrapper.GradleWrapperMain
  * 在根目录执行gradle wrapper
  * https://juejin.im/post/5c9395bcf265da612c3a3def
* kafka程序启动，SLF4J: Failed to load class "org.slf4j.impl.StaticLoggerBinder".
  * core.iml修改slf4j的scope，直接删除
* log4j:WARN No appenders could be found for logger (kafka.utils.Log4jControllerRegistration$).
  * 从config中拷贝log4j.properties  到 ，core/src/scala中