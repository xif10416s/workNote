#   Apache Kylin.md
*   http://lxw1234.com/archives/category/kylin
*   https://my.oschina.net/leejun2005/blog/669369
*   http://kylin.apache.org/cn/docs/install/index.html
*   https://www.jianshu.com/p/7fafa9103252
*   Apache Kylin是一个开源的分布式分析引擎，在Hadoop之上提供超大规模数据的SQL查询接口及多维分析能力，最初由eBay开发并贡献到开源社区，能为巨大Hive表提供亚秒级别的查询响应。
*   Kylin基于MOLAP实现，查询的时候利用Calcite框架，从存储在Hbase的segment表（每一个segment对应着一个htable）获取数据，其实理论上就相当于使用Calcite支持SQL解析，数据从Hbase中读取，中间Kylin主要完成如何确定从Hbase中的哪些表读数据，如何读取数据，以及解析数据的格式。
    -   OLAP系统按照其存储器的数据存储格式可以分为关系OLAP（RelationalOLAP，简称ROLAP）、多维OLAP（MultidimensionalOLAP，简称MOLAP）和混合型OLAP（HybridOLAP，简称HOLAP）三种类型。
        +   ROLAP
            *   ROLAP将分析用的多维数据存储在关系数据库中并根据应用的需要有选择的定义一批实视图作为表也存储在关系数据库中
        +   MOLAP
            *   MOLAP将OLAP分析所用到的多维数据物理上存储为多维数组的形式，形成“立方体”的结构。维的属性值被映射成多维数组的下标值或下标的范围，而总结数据作为多维数组的值存储在数组的单元中。
        +   HOLAP
            *   由于MOLAP和ROLAP有着各自的优点和缺点（如下表所示）,且它们的结构迥然不同，这给分析人员设计OLAP结构提出了难题。为此一个新的OLAP结构——混合型OLAP（HOLAP）被提出，它能把MOLAP和ROLAP两种结构的优点结合起来
    -   Apache Calcite 一个动态数据管理框架 -- 一种查询引擎，连接多种前端和后端
        +   包含了许多组成典型数据管理系统的经典模块，但省略了一些关键功能: 数据存储，数据处理算法和元数据存储库。
        +   在应用程序和一个或多个数据存储位置和数据处理引擎之间的最佳中间层选择。
        +   功能：
            *   支持标准SQL语言；
            *   独立于编程语言和数据源，可以支持不同的前端和后端；
            *   支持关系代数、可定制的逻辑规划规则和基于成本模型优化的查询引擎； 
            *   支持物化视图（materialized view）的管理（创建、丢弃、持久化和自动识别）； 
            *   于物化视图的Lattice和Tile机制，以应用于OLAP分析； 
            *   支持对流数据的查询。 
            *   动态的数据管理系统
        +   应用
            *   Lingual (Cascading项目的SQL接口)、Apache Drill、Apache Hive、Apache Kylin、Apache Phoenix、Apache Samza和Apache Flink
*   主要流程：
    -   星型拓扑结构的数据立方体  =预计算=》多个维度组合的度量  ==保存==》hbase中 ==》对外暴露JDBC、ODBC、Rest API的查询接口，即可实现实时查询。
        +   数据立方体一般由Hive中的一个事实表,多个查找表组成
*   构建（build）数据立方体 --逐层算法（By Layer Cubing）
    -   https://www.cnblogs.com/zlslch/p/7404465.html
*   流式处理分钟级别


## Apache kylin的架构及核心组件
*   数据立方体构建引擎（Cube Build Engine）
    -   当前底层数据计算引擎支持MapReduce1、MapReduce2、Spark等。
*   Rest Server
    -   当前kylin采用的rest API、JDBC、ODBC接口提供web服务。
*   查询引擎（Query Engine）
    -   Rest Server接收查询请求后，解析sql语句，生成执行计划，然后转发查询请求到Hbase中，最后将结构返回给 Rest Server。


##  单独安装 & 测试
*   安装完成后hbase 的default多了一张表：kylin_metadata，存放元数据信息

###  ${KYLIN_HOME}/bin/sample.sh 测试
*   脚本干了什么：
    -   hive 建表   --  ${KYLIN_HOME}/sample_cube/create_sample_tables.sql 
        +   KYLIN_CAL_DT  -- 维度表
            *   保存了时间的扩展信息。如单个日期所在的年始、月始、周始、年份、月份等。
        +   KYLIN_CATEGORY_GROUPINGS  -- 维度表
            *   保存了商品分类的详细介绍，例如商品分类名称等。
        +   KYLIN_COUNTRY  -- 维度表
        +   KYLIN_ACCOUNT  -- 维度表
        +   KYLIN_SALES  -- 事实表 --星型模型的结构
            *   保存了销售订单的明细信息。每一列保存了卖家、商品分类、订单金额、商品数量等信息，每一行对应着一笔交易订单。
    -   初始数据--
        +   LOAD DATA INPATH '${hdfs_tmp_dir}/sample_cube/data/DEFAULT.KYLIN_SALES.csv' OVERWRITE INTO TABLE KYLIN_SALES;
    -   创建cube
*   build cube => 选择 “kylin_sales_cube” 的样例 cube，点击 “Actions” -> “Build”，选择一个在 2014-01-01 之后的日期（覆盖所有的 10000 样例记录);
    -   触发了n个mapereduce 任务 -- 资源占用不多，--     10.43 mins
        +   INSERT OVERWRITE TABLE kylin...'2018-07-01')(Stage-14)
        +   Kylin_Fact_Distinct_Columns_kylin_sales_cube_Step
        +   Kylin_Base_Cuboid_Builder_kylin_sales_cube
        +   Kylin_ND-Cuboid_Builder_kylin_sales_cube_Step
        +   Kylin_HFile_Generator_kylin_sales_cube_Step
    -   预计算cube,结果保存在hbase中 
        +   HBase Table: KYLIN_71PT8VJZNU
*   执行业务查询
    -   select part_dt，sum(price) as total_selled，count(distinct seller_id) as sellers from kylin_sales group by part_dt order by part_dt

## 概念
*   Project
*   model
    -   模型描述了一个星型模式的数据结构，它定义了一个事实表（Fact Table： Wiki:Fact_table）和多个维度表（Lookup Table：Wiki:Lookup_table）的连接和过滤关系。
*   cube
    -   它定义了使用的模型、模型中的表的维度（dimension:Wiki:dimension）、度量（measure:Wiki:measure ,一般指聚合函数，如：sum、count、average等）、如何对段分区（ segments partition）、合并段（segments auto-merge）等的规则。
*   Cube Segment
    -   立方体构建（build）后的数据载体，一个 segment 映射hbase中的一张表，立方体实例构建（build）后，会产生一个新的segment，一旦某个已经构建的立方体的原始数据发生变化，只需刷新（fresh）变化的时间段所关联的segment即可。
*   Job
    -   对立方体实例发出构建（build）请求后，会产生一个作业。该作业记录了立方体实例build时的每一步任务信息。


## 权限管理Project Level ACL
*   QUERY: 
    -   查询表或者cubes权限
*   OPERATION：  
    -   管理cube的权限，包含query权限
*   MANAGEMENT：
    -   管理和设计model 与 cube ，包含OPERATION权限
*   ADMIN
    -   所有权限


##  client
sh ${KYLIN_HOME}/bin/kylin.sh org.apache.kylin.tool.job.CubeBuildingCLI --cube  --endTime ${END} > ${KYLIN_HOME}/logs/system_cube_${CUBE}_${END}.log 2>&1 &


