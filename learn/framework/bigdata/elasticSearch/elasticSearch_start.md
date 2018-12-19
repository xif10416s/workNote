# elasticSearch 安装及测试
##  下载地址
*   https://www.elastic.co/downloads/elasticsearch

##  启动
*   解压
*   ./elasticsearch
*   ./kibana
    -   http://localhost:5601/app/kibana#/home?_g=()

##  插件
*   es -sql -- https://github.com/NLPchina/elasticsearch-sql
    -   http://localhost:9200/_plugin/sql/
    -   node node-server.js 

##  X-Pack -- 打包功能
*   Security
*   Alerting
*   Monitoring
*   Reporting
*   Graph
*   Machine Learning
*   Elasticsearch SQL

## 数据备份同步
*   elasticsearch.yaml 添加白名单：
    -   reindex.remote.whitelist: "otherhost:9200, another:9200, 127.0.10.*:9200, localhost:*"
*   同步语句 -- 先建表
```
POST _reindex
{
  "size": 100,
  "source": {
    "remote": {
      "host": "http://192.168.49.58:8200",
      "socket_timeout": "4m"
    },
    "index": "daily_store_summary"
  },
  "dest": {
    "index": "daily_store_summary"
  }
}

POST _reindex
{
  "source": {
    "remote": {
      "host": "http://192.168.49.58:8200",
      "socket_timeout": "4m"
    },
    "index": "daily_store_summary",
    "query":{
    "range":{
      "buss_date":{
        "gte":"2018-07-01",
        "lte":"2018-07-03"
      }
    }
  }
  },
  "dest": {
    "index": "daily_store_summary"
  }
}


``` 