# SpringBoot.md
*   https://github.com/tengj/SpringBootDemo/tree/master
*   https://springcloud.cc/spring-cloud-dalston.html

## starter
*   功能模块的依赖封装，简化依赖
*   自定义starter
    -   Spring Boot会检查你发布的jar中是否存在 META-INF/spring.factories 文件,该文件中以 EnableAutoConfiguration 为key的属性应该列出你的配置类
        +   org.springframework.boot.autoconfigure.EnableAutoConfiguration=com.yazuo.framework.core.logging.LoggingAutoConfiguration
*   http://www.importnew.com/24164.html


##  常用starter
###  微服务相关
*   org.springframework.cloud
    -   基于熟悉springboot基础上
    -   开发人员提供了快速构建分布式系统中一些常见模式的工具（例如配置管理，服务发现，断路器，智能路由，微代理，控制总线）
*   spring-cloud-starter-eureka  //TODO
    -   将服务注册到注册中心的客户端
    -   它提供基于流量、资源利用率以及出错状态的加权负载均衡
*   spring-cloud-starter-eureka-server
    -   通过 Register， Get，Renew 等 接口提供注册和发现
    -   服务的发现由Eureka客户端来完成，而服务的消费由Ribbon来完成。Ribbo是一个基于HTTP和TCP的客户端负载均衡器，当我们将Ribbon和Eureka一起使用时，Ribbon会从Eureka注册中心去获取服务端列表，然后进行轮询访问以到达负载均衡的作用，服务端是否在线这些问题则交由Eureka去维护。OK，下面我们将通过一个简单的案例，来看看如何实现服务的发现与消费。
*   spring-boot-starter-actuator
    -   actuator是监控系统健康情况的工具
    -   需要实时或定期监控服务的可用性
*   spring-cloud-starter-feign   
    -   Feign是SpringCloud中的HTTP客户端,调用Spring-Cloud远程服务的框架/工具
    -   Feign是一个声明式的Web服务客户端。方便的客户端用来调用其他服务。
    -   可以实现负载均衡
    -   为什么使用Feign
        +   Feign的关注点在于简化开发人员使用工具包的复杂度，以最少的代码编写代码从而提供java http客服端
        +   通过定制解码器和异常处理，开发人员可以任意编写文本化的HTTP API
