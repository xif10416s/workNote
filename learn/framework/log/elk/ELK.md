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