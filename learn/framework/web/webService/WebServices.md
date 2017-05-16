#[WebServices](http://blog.csdn.net/zhuizhuziwo/article/details/8153327)
##WebServices 是什么：
*   基于 XML 和 HTTP 的
*   提供夸平台的远程服务：
*   WebServices三种基本元素
    -   SOAP  = XML + HTTP
        +   传什么东西，XML
        +   怎么传送东西，HTTP
    -   WSDL , 服务的规格，名称，地址，请求参数，响应参数
        *   网络服务描述语言，Web Services Description Language，是一门基于 XML 的语言，用于描述 Web Services 以及如何对它们进行访问。   
        *   WSDL 描述了 Web服务的三个基本属性：
            -   服务所提供的操作
            -   如何访问服务
            -   服务位于何处（通过 URL 来确定就 OK 了）  
    -   UDDI,Universal Description，Discovery and Integration，也就是通用的描述，发现以及整合。
        +   UDDI 就是一个目录，只不过在这个目录中存放的是一些关于 Web 服务的信息而已


##JAX-RS and JAX-WS
*   JAX-WS：全称是JavaTM API forXML-Based Web Services 
*   JAX-RS :全称是  JavaTM API forRESTful Web Services
*   关于JAX-WS与JAX-RS两者是不同风格的SOA架构。前者以动词为中心，指定的是每次执行函数。而后者以名词为中心，每次执行的时候指的是资源。


##[JAX-RS](https://abhirockzz.gitbooks.io/rest-assured-with-jaxrs/content/) 
*   一个基于POJO，注解驱动的构建Web服务符合REST原则框架 
*   JAX-RS Core
    -   Resources,RESTful应用基础
        +   @javax.ws.rs.Path，映射http请求到RESTful资源（类的方法）
    -   Basic annotations，注解，映射http请求到方法
    -   子Resources，
    -   Deployment specific APIs
*   Providers
    -    two providers,  
        +    Message Body Reader: HTTP payload to Java object transformers
            *    raw -> JAXB annotated model class  -> java Bean 
        +    Message Body Writer: convert Java object into HTTP payloads before sending them to the caller
            +    java Bean  -> JAXB annotated model class -> raw
    +   Filters,request processing chain
        *   Filters provide AOP (Aspect Oriented Programming) 
        *   应该处理：headers, request URIs, the invoked HTTP method
        *   不应该处理业务相关的，授权，认证，验证等操作
        *   四类filter
            *   Server side Request filter,服务端filter，实现ContainerRequestFilter接口
                -   Pre-matching filters，分发给java 方法之前调用
                -   Post-matching filters，完成方法匹配之后
            *   Server side Response filter，after a response (or an exception)
            *   Client side Request filter，after the HTTP request creation before it is dispatched to the server.
            *   Client side Response filters
        *   ContainerRequestContext共享属性
    +   Interceptors，manipulate HTTP message payloads
        *   two categories 
            -   Reader Interceptor，客户端的发送消息处理，服务端的接受消息处理
            -   Writer Interceptor，客户端的接受消除处理，服务端的发送消息处理
    +   Exception Mapper，maps Java exceptions to Response objects.


##RESTful services 
*   指的是一组架构约束条件和原则。满足这些约束条件和原则的应用程序或设计就是 RESTful。
    -   客户端和服务器之间的交互在请求之间是无状态的