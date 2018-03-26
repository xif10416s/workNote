#Apache Airflow -- 
##工作流执行
*   有向无环图dag
*   a generic workflow scheduler with dependency management.

## 安装
*   http://site.clairvoyantsoft.com/installing-and-configuring-apache-airflow/
*   http://morefreeze.github.io/2016/12/airflow.html
*   配置时区：
*   Please note that the Web UI currently only runs in UTC
*   [core]
default_timezone = CST
```
 import pytz
 from datetime import datetime
 pytz.timezone('Asia/Shanghai').localize(datetime.now())
```

## command 
*   nohup airflow webserver -p 8080 >> ~/airflow/logs/webserver.logs &
*   nohup airflow scheduler >> ~/airflow/logs/scheduler.logs &
*   ps -eaf | grep airflow
*   ps -ef |grep airflow-webserver |awk '{print $2}'|xargs kill -9
*   ps -ef |grep 'airflow webserver' |awk '{print $2}'|xargs kill -9
*   airflow delete_dag  tutorial  -f


## 使用
*   时间计算 -- https://www.jianshu.com/p/e878bbc9ead2
    -   actual_execution_date= execution_date +schedual_interval”，或者我们换一种说法，我们在airflow上看到一个任务是6am执行的，而且interval=4hours，那么execution_date的值是2am，而不是6am，所以获取某个task的真正执行时间，需要获取execution_date的下个执行周期时间，即使用dag.following_schedule(execution_date)
    -    对于周期任务， airflow 传入的时间是上一个周期的时间（划重点），比如你的任务是每天执行， 那么今天传入的是昨天的日期，如果是周任务，那传入的是上一周今天的值。
    -    之前过期的任务不执行，开始时间设置在上一个周期之前
    -    start date 的配置将不会再生效，这个作业的调度开始时间将直接按照上次调度所对应的 execution date 来计算。


## spark
*   SparkSubmitOperator
```


```



