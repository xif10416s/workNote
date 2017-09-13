#   ELK 日志分析系统 
##  ELK由Elasticsearch、Logstash和Kibana三部分组件组成

### download
*   [https://www.elastic.co/downloads/logstash](https://www.elastic.co/downloads/logstash)
*   [https://www.elastic.co/downloads/elasticsearch](https://www.elastic.co/downloads/elasticsearch)
*   [https://www.elastic.co/downloads/kibana](https://www.elastic.co/downloads/kibana)

### start
*   elasticSearch
    -  /opt/elk/elasticsearch/bin/elasticsearch &
    -  curl -X GET http://localhost:9200/
*   logstash
    -   创建配置文件 logstash_start.conf
    -   /opt/elk/logstash/bin/logstash -f /opt/elk/logstash/logstash_start.conf &
*   kibana
    -   /opt/elk/kibana/bin/kibana &
    -   通过浏览器访问http://[you_address]:5601就可以看到kibana界面了。
*   test 
    -   echo "hello" > /var/log/messages

logstash_start.conf
```
input {
file {
  path => "/var/log/messages"
  start_position => "beginning"
}
}
output {
  elasticsearch { hosts => ["localhost:9200"] }
}
``` 

#   test evn start
ES_JAVA_OPTS="-Xmx512m -Xms512m" /opt/elk/elasticsearch/bin/elasticsearch &
/opt/elk/logstash/bin/logstash -f /opt/elk/logstash/logstash_start.conf &
/opt/elk/kibana/bin/kibana &


4、ELK的帮助手册

ELK官网：https://www.elastic.co/
ELK官网文档：https://www.elastic.co/guide/index.html
ELK中文手册：http://kibana.logstash.es/content/elasticsearch/monitor/logging.html

https://wenku.baidu.com/view/7ea490d0f7ec4afe04a1dff8.html

##  ELK在大数据运维系统中的应用
*   分布式日志数据集中式查询和管理
*   系统监控，包含系统硬件和应用各个组件的监控
*   故障排查
*   安全信息和事件管理
*   报表功能
*   线上业务的准实时监控、业务异常时及时定位原因、排除故障、程序研发时跟踪分析Bug、业务趋势分析、安全与合规审计，深度挖掘日志的大数据价值
*   日志查询，问题排查，上线检查
*   服务器监控，应用监控，错误报警，Bug管理
*   性能分析，用户行为分析，安全漏洞分析，时间管理

##  ELK实战举例
*   通过ELK组件对Spark作业运行状态监控，搜集Spark环境下运行的日志。经过筛选、过滤并存储可用信息，从而完成对Spark作业运行和完成状态进行监控，实时掌握集群状态，了解作业完成情况，并生成报表，方便运维人员监控和查看。
*   对系统资源状态监控
    -   供用户实时监控系统资源、节点状态、磁盘、CPU、MEM，以及错误、警告信息等。
*   通过ELK组件对系统日志管理和故障排查 
