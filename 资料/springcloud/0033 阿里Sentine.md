# 阿里Sentinel

## sentinel和Hystrix的区别

![](https://xinqianpingtaib2btest.oss-cn-shenzhen.aliyuncs.com/xinqianpingtaib2btest/blogimg/2020/微信截图_20200413164008.jpg)

https://sentinelguard.io/zh-cn/

## 新建Modele

cloudalibaba-sentinel-service8401

## YML

```yaml
server:
  port: 8401
spring:
  application:
    name: cloudalibaba-sentinel-service
  cloud:
    nacos:
      discovery: #Nacos注册中心地址
        server-addr: localhost:8848
    sentinel:
      transport: #dashboard地址
        dashboard: localhost:8080
        port: 8719  #默认端口，如果被占用则从8719依次+1扫描
      datasource:
        dsl:
          nacos:
            server-addr: localhost:8848
            dataId: cloudalibaba-sentinel-service
            groupId: DEFAULT_GROUP
            data_type: json
            rule-type: flow
management:
  endpoints:
    web:
      exposure:
        include: "*"
```

## POM

```xml
        <dependency>
            <groupId>com.alibaba.cloud</groupId>
            <artifactId>spring-cloud-starter-alibaba-nacos-discovery</artifactId>
        </dependency>
        <!--    后续做持久化用到    -->
        <dependency>
            <groupId>com.alibaba.csp</groupId>
            <artifactId>sentinel-datasource-nacos</artifactId>
        </dependency>
        <!--   sentinel     -->
        <dependency>
            <groupId>com.alibaba.cloud</groupId>
            <artifactId>spring-cloud-starter-alibaba-sentinel</artifactId>
        </dependency>
        <!--   openfeign     -->
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-openfeign</artifactId>
        </dependency>
```

## 主启动

```java
@SpringBootApplication
@EnableDiscoveryClient
public class MainApp8401 {
    public static void main(String[] args) {
        SpringApplication.run(MainApp8401.class, args);
    }
}
```

## FlowLimitController

```java
@RestController
@Slf4j
public class FlowLimitController {

    @GetMapping("/testA")
    public String testA(){
        try {
            TimeUnit.MILLISECONDS.sleep(800);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        return "---------testA";
    }

    @GetMapping("/testB")
    public String testB(){
        return "----------testB";
    }

    @GetMapping("/testD")
    public String testD(){
        try {
            TimeUnit.SECONDS.sleep(1);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
//        log.info("testD 测试RT");
        log.info("testD 异常比例");
        int age = 10/0;
        return "------testD";
    }

    @GetMapping("/testE")
    public String testE(){
        log.info("testD 测试异常数");
        int age = 10/0;
        return "------testD 测试异常数";
    }

    @GetMapping("/testHotKey")
    @SentinelResource(value = "testHotKey",blockHandler = "deal_testHotKey")
    public String testHotKey(@RequestParam(value = "p1",required = false) String p1,
                             @RequestParam(value = "p2", required = false) String p2) {
        return "-------testHotKey";
    }

    public String deal_testHotKey(String p1, String p2,
                             BlockException exception){
        return "----deal_testHotKey,o(╥﹏╥)o";

    }

}
```

## RateLimitController

```java
@RestController
public class RateLimitController {

    @GetMapping("/byResource")
    @SentinelResource(value = "byResource",blockHandler = "handleException")
    public CommonResult byResource(){
        return new CommonResult(200, "按资源名称限流测试ok", new Payment(2020L, "serial001"));
    }

    public CommonResult handleException(BlockException exception) {
        return new CommonResult(444, exception.getClass().getCanonicalName() + "\t 服务不可用");
    }

    @GetMapping("/rateLimit/byUrl")
    @SentinelResource(value = "byUrl",blockHandlerClass = CustomerBlockHandler.class,
    blockHandler = "handlerException")
    public CommonResult byUrl(){
        return new CommonResult(200,"按url限流测试OK", new Payment(2020L, "serial001"));
    }

    //CustomerBlockHandler
    @GetMapping("/rateLimit/CustomerBlockHandler")
    @SentinelResource(value = "CustomerBlockHandler",blockHandlerClass = CustomerBlockHandler.class,
            blockHandler = "handlerException2")
    public CommonResult CustomerBlockHandler(){
        return new CommonResult(200,"按客户自定义限流测试OK", new Payment(2020L, "serial003"));
    }

}
```

## CustomerBlockHandler

```java
public class CustomerBlockHandler {

    public static CommonResult handlerException(BlockException exception) {
        return new CommonResult(4444, "按客户自定义,global handlerException", new Payment(2020L, "serial0003"));
    }
    public static CommonResult handlerException2(BlockException exception) {
        return new CommonResult(4444, "按客户自定义2,global handlerException", new Payment(2020L, "serial0003"));
    }
}
```

##  启动MainApp8401

MainApp8401

## 启动Nacos

D:\javatools\e-nacos\nacos\bin>startup.cmd

## 启动sentinel

java -jar sentinel-dashboard-1.7.0.jar

注意了，sentinel是懒加载的，要先访问一次url才有界面详情

http://localhost:8401/testA

http://localhost:8401/testB

http://localhost:8080/



![](https://xinqianpingtaib2btest.oss-cn-shenzhen.aliyuncs.com/xinqianpingtaib2btest/blogimg/2020/微信截图_20200413160243.jpg)



# 流控规则

##  基本介绍

![](https://xinqianpingtaib2btest.oss-cn-shenzhen.aliyuncs.com/xinqianpingtaib2btest/blogimg/2020/微信截图_20200413160826.jpg)

![](https://xinqianpingtaib2btest.oss-cn-shenzhen.aliyuncs.com/xinqianpingtaib2btest/blogimg/2020/微信截图_20200413160923.jpg)





## 直接

<img src="https://xinqianpingtaib2btest.oss-cn-shenzhen.aliyuncs.com/xinqianpingtaib2btest/blogimg/2020/微信截图_20200413163313.jpg"  />

快速访问http://localhost:8401/testA

结果：

```
Blocked by Sentinel (flow limiting)
```

## 关联

![](https://xinqianpingtaib2btest.oss-cn-shenzhen.aliyuncs.com/xinqianpingtaib2btest/blogimg/2020/微信截图_20200413164954.jpg)





## 链路

调用链路入口限流

# 流控效果 

## 直接失败

Blocked by Sentinel (flow limiting)

## Warm Up

预热

## 排除等待

漏桶算法

![](https://xinqianpingtaib2btest.oss-cn-shenzhen.aliyuncs.com/xinqianpingtaib2btest/blogimg/2020/微信截图_20200414123309.jpg)

