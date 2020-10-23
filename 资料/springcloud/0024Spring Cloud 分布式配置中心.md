# Spring Cloud 分布式配置中心

![](https://xinqianpingtaib2btest.oss-cn-shenzhen.aliyuncs.com/xinqianpingtaib2btest/blogimg/2020/微信截图_20200409213040.jpg)

配置仓库

```
https://gitee.com/exclusiver/springcloud-config.git
```

POM

```xml
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-config-server</artifactId>
        </dependency>
```

YML

```yaml
server:
  port: 3344
spring:
  application:
    name: cloud-config-center
  cloud:
    config:
      server:
        git:
          # 采用ssh免密码登录
          #uri: https://gitee.com/exclusiver/springcloud-config.git
          uri: https://gitee.com/wangwang2020/springcloud-config.git
          #搜索目录
          search-paths:
            - springcoud-config
      ##读取分支
      label: master

#服务注册到eureak地址
eureka:
  client:
    service-url:
      defaultZone: http://localhost:7001/eureka
```

主启动

```java
@SpringBootApplication
@EnableConfigServer
public class ConfigCenterMain3344 {
    public static void main(String[] args) {
        SpringApplication.run(ConfigCenterMain3344.class, args);
    }
}
```

windows下修改C:\Windows\System32\drivers\etc目录下的hosts文件

`127.0.0.1 config-3344.com`



```
http://config-3344.com:3344/master/config-dev.yml
```

