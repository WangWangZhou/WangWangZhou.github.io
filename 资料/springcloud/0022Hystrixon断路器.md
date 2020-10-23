# Hystrixon断路器



# 概述

![](https://xinqianpingtaib2btest.oss-cn-shenzhen.aliyuncs.com/xinqianpingtaib2btest/blogimg/2020/微信截图_20200407213521.jpg)





# Hystrix重要概念

服务雪崩

![](https://xinqianpingtaib2btest.oss-cn-shenzhen.aliyuncs.com/xinqianpingtaib2btest/blogimg/2020/微信截图_20200407211702.jpg)



服务降级

服务熔断

服务限流





# hystrix案例

# hystrix工作流程

# 服务监控hystrixDashboard

## cloud-consumer-hystrix-dashboard9001

```java
@SpringBootApplication
@EnableHystrixDashboard
public class HystrixDashboardMain9001 {

    // 访问 http://localhost:9001/hystrix
    
    public static void main(String[] args) {
        SpringApplication.run(HystrixDashboardMain9001.class, args);
    }
}
```

YML

```yaml
server:
  port: 9001
```

POM

```xml
    <dependencies>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-netflix-hystrix-dashboard</artifactId>
        </dependency>
        <dependency>
            <groupId>com.atguigu.springcloud</groupId>
            <artifactId>cloud-api-commons</artifactId>
            <version>${project.version}</version>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-actuator</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-devtools</artifactId>
            <scope>runtime</scope>
            <optional>true</optional>
        </dependency>
    </dependencies>
```

cloud-provider-hystrix-payment8001

![](https://xinqianpingtaib2btest.oss-cn-shenzhen.aliyuncs.com/xinqianpingtaib2btest/blogimg/2020/微信截图_20200409142448.jpg)

POM

```xml
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-actuator</artifactId>
        </dependency>
```



## 9001监控8001



http://localhost:9001/hystrix

http://localhost:8001/hystrix.stream

![](https://xinqianpingtaib2btest.oss-cn-shenzhen.aliyuncs.com/xinqianpingtaib2btest/blogimg/2020/微信截图_20200409142057.jpg)

通过访问

正确的地址 http://localhost:8001/payment/circuit/1

和错误的地址http://localhost:8001/payment/circuit/-1

来测试

![](https://xinqianpingtaib2btest.oss-cn-shenzhen.aliyuncs.com/xinqianpingtaib2btest/blogimg/2020/微信截图_20200409143553.jpg)

整图说明

![](https://xinqianpingtaib2btest.oss-cn-shenzhen.aliyuncs.com/xinqianpingtaib2btest/blogimg/2020/微信截图_20200409144236.jpg)

整图说明2

![](https://xinqianpingtaib2btest.oss-cn-shenzhen.aliyuncs.com/xinqianpingtaib2btest/blogimg/2020/微信截图_20200409144445.jpg)



