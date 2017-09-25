#   elasticsearch
##  https://elasticsearch.cn/book/elasticsearch_definitive_guide_2.x/running-elasticsearch.html
##  https://www.elastic.co/guide/en/elasticsearch/reference/current/mapping.html
##  https://es.xiaoleilu.com/052_Mapping_Analysis/45_Mapping.html

##  分词插件https://github.com/medcl/elasticsearch-analysis-ik

```
curl -XPOST http://localhost:9200/index/_analyze?analyzer=ik_max_word  -d'
联想召回笔记本电源线
'


curl -XGET http://localhost:9200/index/_mapping

curl -XPOST http://localhost:9200/index/_search
```


文档的唯一标识

##  http://localhost:5601/app/kibana


