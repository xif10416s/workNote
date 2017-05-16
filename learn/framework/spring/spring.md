#Spring 应用
##占位配置
*   PropertyPlaceholderConfigurer
*   PropertySourcesPlaceholderConfigurer
    -   优先使用
    -   融合了系统变量


##spring 自定义标签
*   http://sammor.iteye.com/blog/1106254



##spring + jaxrs ，restful webservice
*   定义接口，用注解描述wsdl相关的元数据
    -   如url,消息类型，参数等
*   实现接口
*   spring整合jaxrs ， jaxrs:server 标签描述服务
    +   jaxrs:serviceBeans，引用服务的列表（引用接口实现类的列表）
    +   jaxrs:providers ，插件
        *   
