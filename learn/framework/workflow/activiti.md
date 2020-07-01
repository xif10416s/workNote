####  Activiti  工作流

#####  主要功能

* 管理流程, 处理业务流程的
* 适应业务流程的变化，降低开发维护成本



#####  基础流程

* 定义工作流，activiti 以图的方式定义：有哪些步骤
  * 业务流程建模与标注（Business Process Model and Notation，BPMN) ，描述流程的基本符号，包括这些图元如何组合成一个业务流程图（Business Process Diagram）
  * BPMN这个就是我们所谓**把工作流定义出来的流程图**
* 初始化工作流引擎，配置数据库连接
  * 代码初始化 或者 配置文件初始化activiti.cfg.xml
  * `ProcessEngine processEngine = ProcessEngines.getDefaultProcessEngine();`
* 通过引擎部署工作流
* 根据名称启动对应的工作流
* 各个角色查看自己的任务列表，发现需要自己处理的任务时，执行相对应处理
* 标记完成任务，进入下一个步骤



####  框架组件

#####  底层支撑表 28张，所有的表都以ACT_开头

* ACT_RE_*: 'RE'表示repository。 这个前缀的表包含了流程定义和流程静态资源 （图片，规则，等等）。对应RepositoryService接口
* ACT_RU_*: 'RU'表示runtime。 这些运行时的表，包含流程实例，任务，变量，异步任务，等运行中的数据。 Activiti只在流程实例执行过程中保存这些数据， 在流程结束时就会删除这些记录。 这样运行时表可以一直很小速度很快。对应RuntimeService接口和TaskService接口
* ACT_ID_*: 'ID'表示identity。 这些表包含身份信息，比如用户，组等等。对应IdentityService接口
* ACT_HI_*: 'HI'表示history。 这些表包含历史数据，比如历史流程实例， 变量，任务等等。对应HistoryService接口
* ACT_GE_*: 通用数据， 用于不同场景下，如存放资源文件。



##### 7大接口

* RepositoryService：提供一系列管理流程部署和流程定义的API。
* RuntimeService：在流程运行时对流程实例进行管理与控制。
* TaskService：对流程任务进行管理，例如任务提醒、任务完成和创建任务等。
* IdentityService：提供对流程角色数据进行管理的API，这些角色数据包括用户组、用户及它们之间的关系。
* ManagementService：提供对流程引擎进行管理和维护的服务。
* HistoryService：对流程的历史数据进行操作，包括查询、删除这些历史数据。
* FormService：表单服务。



表设计原则：流程数据和业务数据相分离。Activiti相关表只负责流程的跳转、走向等。流程中产生的业务表单数据、审批意见、附件等存储在开发人员定义的业务表中。流程数据和业务数据之间通过processInstanceId(流程实例ID)和业务数据主键相互关联。



####  流程图制作

#####  定义流程图方式

* idea ,eclipse 插件
* 在线设计功能



#####   模型元素

######  Task  任务

* userTask :   需要人工干的一件事情，当流程执行到userTask任务时，会创建一个新的task在act_ru_task表中，通过指定 user 或者 group 标记处理人
  * activiti:dueDate="${dateVariable}"   ：  指定任务的截止日期，可以被**TaskService**，`TaskListener`s  修改
  * activiti:assignee="kermit"  ：  指定某个处理人,办理人只能指定一个人，不能使用逗号分隔
  * activiti:candidateUsers="kermit, gonzo" :  候选用户，可以指定多个，适合人不多的情况，设置候选组需要主动签taskService.claim(taskId, currentUserId);
  * activiti:candidateGroups="management, accountancy"    ： 办理角色 或 办理岗位， 适合办理人多的情况
    * 设置办理人优先级：Assignee>Candidate users>Candidate groups



##### 任务处理责任人指定与匹配方法

* 









####  参考文档

* [Activiti User Guide 官网](https://www.activiti.org/userguide/)
* [Activiti6简明教程](https://www.jianshu.com/p/701056e672a4)
* https://tomoya92.github.io/2019/04/24/activiti-env/
* https://juejin.im/post/5aafa3eef265da23784015b9
* 表结构：https://blog.csdn.net/hj7jay/article/details/51302829
* https://www.cnblogs.com/dengjiahai/p/7536999.html