#基本操作
##cassandra2.0 
###  查看容量

   ./nodetool cfstats  mlapp.hq_allocate_ad_log
### 删除
*   罗辑

    写入数据最终被持久化到sstables,sstables是不可编辑的

    数据不能被直接删除

    删除操作转换为更新"tombstone"状态

    在第一次compaction发生时候,数据将被删除完全和相应的磁盘空间回收

    执行nodetool repair keySpace table <== XX--compact type validation

    nodetool <options> compact <keyspace> ( <table> ... ) //执行压缩,触发回收

    nodetool compactionstats


----------------

    初始456g--SSTable count: 14---15年9月-16年5月

    删除15年9月--15年12月--没次5-15天,每天几次

    一周左右480g并没有减少

    运行 compact  需要 45小时

    运行前      ==>sstable :14 --Write Count: 14825347162--bytes: 506478668520

    运行一晚上==>SSTable count: 25-- Write Count: 14852918287-- bytes: 515938190221

    sstable增加11,write count 增加27571125 ,     容量增加:8.8098661gb

  5. 清理完成后

 Write Latency: 0.03196510995766934 ms.

        Pending Tasks: 0

                Table: ml_ad_exposure_log

                SSTable count: 18

                Space used (live), bytes: 375394649806

                Space used (total), bytes: 375470895731

                Off heap memory used (total), bytes: 247701116



-------------------

*   command
1.压缩

  ./nodetool compact mlapp ml_ad_exposure_log

2.状态查看

 ./nodetool compactionstats

./nodetool cfstats mlapp.ml_ad_exposure_log;

3.snapshot 清理

../nodetool clearsnapshot -t name mlapp

./nodetool clearsnapshot -t 3c7d3cd0-2b99-11e6-a45d-9d020f71c9d1

