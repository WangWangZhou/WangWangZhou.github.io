# Spring-cloud-Stream

Spring Cloud Stream is a framework for building highly scalable event-driven microservices connected with shared messaging systems.

The framework provides a flexible programming model built on already established and familiar Spring idioms and best practices, including support for persistent pub/sub semantics, consumer groups, and stateful partitions.



官网

https://spring.io/projects/spring-cloud-stream



https://cloud.spring.io/spring-cloud-static/spring-cloud-stream/3.0.3.RELEASE/reference/html/

rabbitMQ

https://cloud.spring.io/spring-cloud-static/spring-cloud-stream-binder-rabbit/3.0.3.RELEASE/reference/html/spring-cloud-stream-binder-rabbit.html

https://cloud.spring.io/spring-cloud-static/spring-cloud-stream-binder-rabbit/3.0.3.RELEASE/reference/html/spring-cloud-stream-binder-rabbit.html#_partitioning_with_the_rabbitmq_binder

中文指导手册

https://m.wang1314.com/doc/webapp/topic/20971999.html



cloud-stream-rabbitmq-provider8801

cloud-stream-rabbitmq-consumer8802

cloud-stream-rabbitmq-consumer8803

##  Spring Cloud Stream标准流程套路

![](https://xinqianpingtaib2btest.oss-cn-shenzhen.aliyuncs.com/xinqianpingtaib2btest/blogimg/2020/微信截图_20200410193355.jpg)



## 编码api和常用注解

![](https://xinqianpingtaib2btest.oss-cn-shenzhen.aliyuncs.com/xinqianpingtaib2btest/blogimg/2020/微信截图_20200410193057.jpg)



# cloud-stream-rabbitmq-provider8801

pom

```xml
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-stream-rabbit</artifactId>
        </dependency>
```



yml

```yaml
server:
  port: 8801
spring:
  application:
    name: cloud-stream-provider
  cloud:
    stream:
      binders:  #需要绑定的rabbitmq的服务信息
        defaultRabbit: #定义的名称，用于binding整合
            type: rabbit #消息组件类型
            environment: #环境配置
              spring:
                rabbitmq:
                  host: 192.168.43.212
                  port: 5672
                  username: guest
                  password: guest
      bindings: # 服务的整合处理
        output: # 名字是一个通道的名称
          destination: studyExchange # 表示要使用的Exchange名称定义
          content-type: application/json # 设置消息类型，本次为json
          binder: defaultRabbit #设置要绑定的消息服务的具体设置
eureka:
  client:
    service-url:
      defaultZone: http://localhost:7001/eureka
  instance:
    lease-renewal-interval-in-seconds: 2
    lease-expiration-duration-in-seconds: 5
    instance-id: send-8801.com
    prefer-ip-address: true
```

发送消息

http://localhost:8801/sendMessage

![](https://xinqianpingtaib2btest.oss-cn-shenzhen.aliyuncs.com/xinqianpingtaib2btest/blogimg/2020/微信截图_20200410202005.jpg)

