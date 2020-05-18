####   什么是索引

* 提高数据查询效率的数据结构
* where 条件 与 order by时会用到



####  索引的分类

* 从存储结构上来划分：
  * BTree索引（B-Tree或B+Tree索引）
  * Hash索引
  * full-index全文索引
  * R-Tree索引
*  从应用层次来分
  * 普通索引  -- 一个索引只包含单个列，一个表可以有多个单列索引
  * 唯一索引 -- 索引列的值必须唯一，但允许有空值
  * 复合索引 -- 即一个索引包含多个列
* 根据中数据的物理顺序与键值的逻辑（索引）顺序关系
  * 聚集索引 -- 并不是一种单独的索引类型，而是一种数据存储方式。
  * 非聚集索引



#### 索引的底层实现

##### hash索引

* 哈希表，只有精确匹配所有的列查询才生效
* 哈希表的key为 该表所有hash索引列的计算一个hash码，值为某一行记录对应的数据的行指针
* 不能范围查找



















#### 参考

* https://mp.weixin.qq.com/s?__biz=MzI3ODcxMzQzMw==&mid=2247494310&idx=2&sn=2a5632199bbd99e6313b19b9d24b74d0&chksm=eb506f90dc27e68621df63556b8096109dd1feec2c5201be9ea9fe98cab181426e1737fe3b93&scene=126&sessionid=1589618484&key=a7d1b6a6a8237840911f847071d8af2790685f2456a858df06b56765b9935906dba300b854f7a45344cbaef35f591173969cbf4d96a1afd89ffd31217ae3cdc0fa049fbe736df0957503958ed8bf434a&ascene=1&uin=Mjk1NTAwNzcwMg%3D%3D&devicetype=Windows+10+x64&version=62090070&lang=zh_CN&exportkey=AaeXgpnnkcvvPrGIqQ0%2BVXA%3D&pass_ticket=YTUVElEEK0KAGoYmxKhwAjnTVzcmdCMrE9AGqeU0qacdBk2qb2tb9IXdcPw7ZtvT