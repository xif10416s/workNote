#Apache CXF
##[Apache CXF是什么](https://www.ibm.com/developerworks/cn/education/java/j-cxf/)
*   Apache CXF = Celtix + XFire,开源的 Services 框架
*   支持**JAX-WS**（Java API for XML Web Services），
    -   在 JAX-WS中，一个远程调用可以转换为一个基于XML的协议例如SOAP，在使用JAX-WS过程中，开发者不需要编写任何生成和处理SOAP消息的代码。JAX-WS的运行时实现会将这些API的调用转换成为对应的SOAP消息。
    -   优点：
        +   java应用更好的夸平台
            *   动态代理机制，只需要调用定义的java接口
        +   注解
            *   通过注解配置元信息，标注java的web service功能
        +   异步调用机制
        +   依赖注入，@Resource
        +   数据绑定，Java Architecture for XML Binding (JAXB) 
        +   Dynamic and static clients
            *   动态client,dispatch
                -    an XML messaging oriented 
                -    支持异步调用
            *   静态client,代理client对象
        +   支持 Message Transmission Optimized Mechanism (MTOM)
        +   Multiple data binding technologies
*   JAXB，Java Architecture for XML Binding，java 和 xml数据绑定
    *   将Java 类和 xml 相互映射
    *    通过注解配置，不用wsdl
    *    开发服务接口：
        -    the standard JavaBeans service endpoint interface 
        -    a new Provider interface to enable services to work at the XML message level.
    *   标记发布服务
        -   javax.jws.WebService annotation
        -   avax.jws.WebServiceProvider annotation 
    *   两种开发方式：
        -   Java bean -> wsdl
        -   wsdl -> java bean
*   支持RESTful services
    -   使用http 绑定



