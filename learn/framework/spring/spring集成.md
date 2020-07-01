##  spring 集成框架
###  https://docs.spring.io/spring-boot/docs/current-SNAPSHOT/reference/htmlsingle/#boot-features-kafka



## SpringBoot+Shiro+JWT

https://springboot.plus/guide/shiro-jwt.html#springboot-shiro-jwt





http://doc.jeecg.com/1273753

http://doc.jeecg.com/1273969

https://github.com/stylefeng/Guns

https://github.com/zhangdaiscott/jeecg-boot





https://www.ipaddress.com/





#####  基础问题

*  springboot + shiro   @Transaction不生效

  * SpringTransactionAnnotationParser 确认是否解析@Transaction，生成代理类

    * ```
      ((Method) element).clazz.name.equals("org.fxi.quick.module.sys.service.impl.SysRoleServiceImpl")
      ```

    * 

