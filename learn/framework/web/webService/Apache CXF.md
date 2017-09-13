#   Apache CXF
##  [Apache CXF是什么](https://www.ibm.com/developerworks/cn/education/java/j-cxf/)
*   Apache CXF = Celtix + XFire,开源的 Services 框架
*   CXF 是一种基于 Servlet 技术的 SOA 应用开发框架，需要 Servlet 容器的支持。
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

##  spring web 集成
### web.xml
*   `org.apache.cxf.transport.servlet.CXFServlet`
    -   loadBus,加载bus ,SpringBus
        +    `<bean id="cxf" class="org.apache.cxf.bus.spring.SpringBus" destroy-method="shutdown"/>` 配置在META-INF/cxf/cxf.xml

### org.apache.cxf.bus.spring.SpringBus
*   extensions
*   in
*   out


### Interceptors and Phases
*   拦截器
*   分为输入拦截器，输出拦截器
*   分为不同的阶段，也就是执行的顺序
*   定义两个方法，一个处理消息 handleMessage， 一个是处理错误 handleFault。别看Interceptor这么简单，这里需要提醒注意的是，在实行具体的Interceptor的这两个方法中，千万别调用Interceptor内部的成员变量。这是由于Interceptor是面向消息来进行处理的，每个Interceptor都有可能运行在不同的线程中，如果调用了Interceptor中的内部成员变量，就有在Interceptor中造成临界资源的访问的情况，而这时的Interceptor也就不是线程安全的Interceptor了。

### Features in CXF
*   adding capabilities to a Server, Client or Bus ， 添加功能


org.apache.cxf.continuations





