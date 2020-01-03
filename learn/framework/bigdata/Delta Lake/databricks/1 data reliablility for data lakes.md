#  data reliablility for data lakes

##   现状
*	企业拥有大量各种各样的数据：客户数据，视频数据，点击流，邮件，传感器数据等等 --- 企业的数据湖（data lake）
*	在数据湖中的数据太脏，无法直接用来做数据分析

##   案例
*	数据存在kafka中，需要同时实现两个目标，一个是实时分析，一个是数据挖掘报表
*	实时分析可以通过spark 实现
*	但是kafka只能存一周左右数据，不适合做历史数据分析如数据挖掘
*	实现步骤
	*	第一步 -- 采用lambda架构：数据流（kafka) , batch layer (data lake) , speed layer(spark streaming), server layer
	*	第二步 -- validation ,数据清洗过滤验证
	*   第三步 -- reprocessing , 失败可重复执行
	*	第四步 -- 更新与合并 ，

## data lakes 在数据可靠性上的挑战
*	任务执行失败，导致数据在错误的状态，需要全部回滚
*	缺乏执行质量，容易创建不一致和无用的数据
*	缺少事务，无法混合添加和读取，批处理 -- TODO

##  解决方案
*	数据源（kafka，kinesis,spark)  ---> raw ingestion --> filtered,cleaned,augmented -->bussiness-level aggregates-->两种处理（实时的流分析，离线的AI和报告）
	*	原始数据-->清洗过滤-->业务聚合 所有步骤实现ACID事务
	*	原始数据基本很少处理，与原始一致
	*	支持年级别的数据存储
	*	当商业逻辑发送变化时，容易重新计算

##  dalta lake中的table =  一组结果行为集合
*	更新元数据 -- 名称，schema, 分区等
*	添加文件 
*	删除文件
*	设置事务 -- 记录一组独立的事务id
*	修改协议 -- 升级版本
*	结果产物：
	*	元数据，一组文件，一组事务id,版本号

##  日志文件结构 000000.json
*	所有表的变化（添加，修改，删除）都被有序的，原子的保存，称为commits

##  总结
*	强调使用管道处理，直接使用数据源，可以适应业务变化，调整部分或全部处理步骤