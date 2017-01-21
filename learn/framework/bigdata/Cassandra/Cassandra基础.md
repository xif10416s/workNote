##基本数据模型
*   keyspace→database，scheme
*   column family→table，是某个特定key的column集合，是一个行结构，每个cf物理上被放在单独文件
*   column→列，最小单位，包含name和value和timestamp
*   column的排序按照column的name排序1

##cassandra的存储结构
commitlog→客户端提交首先保存数据和操作，持久化到磁盘，用以恢复，按照一定格式组成的byte数组

mentable→从commitlog刷新到内存中，回写缓存

sstable→数据持久化到磁盘，分为data，index，filter三中数据格式
1:记录数据到commitlog
2:同时保存到memtable，定期刷新
3:压缩
