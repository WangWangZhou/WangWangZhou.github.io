# Sentinel服务熔断Ribbon系列

## 服务提供者

建Module

写POM

改YML

主启动

业务类



cloudalibaba-provider-payment9003

cloudalibaba-provider-payment9004

POM

```xml
        <dependency>
            <groupId>com.alibaba.cloud</groupId>
            <artifactId>spring-cloud-starter-alibaba-nacos-discovery</artifactId>
        </dependency>
```

YML

```yaml
server:
  port: 9003
spring:
  application:
    name: nacos-payment-provider
  cloud:
    nacos:
      discovery:
        server-addr: localhost:8848
management:
  endpoints:
    web:
      exposure:
        include: "*"
```

主启动

```java
@SpringBootApplication
@EnableDiscoveryClient
public class PaymentMain9003 {
    public static void main(String[] args) {
        SpringApplication.run(PaymentMain9003.class,args);
    }
}
```

业务类

```java
@RestController
public class PaymentController {

    @Value("${server.port}")
    private String serverPort;

    public static HashMap<Long, Payment> hashMap = new HashMap<>();

    static {//模拟数据库
        hashMap.put(1L, new Payment(1L, "28a8c1e3bc2742d8848569891fb42181"));
        hashMap.put(2L, new Payment(2L, "28a4c1e3bc2742d8848569891fb42181"));
        hashMap.put(3L, new Payment(3L, "28a5c1e3bc2742d8848569891fb42181"));
    }

    @GetMapping(value = "/paymentSQL/{id}")
    public CommonResult<Payment> paymentSQL(@PathVariable("id") Long id){
        Payment payment = hashMap.get(id);
        CommonResult<Payment> result = new CommonResult<>(200, "from mysql,serverPort: " + serverPort, payment);
        return result;
    }

}
```

## 服务消费者

 YML

```yaml
server:
  port: 84
spring:
  application:
    name: nacos-order-consumer
  cloud:
    nacos:
      discovery:
        server-addr: localhost:8848
    sentinel:
      transport:
        dashboard: localhost:8080
        port: 8719

# 消费者将要去访问的微服务名称(注册成功进nacos的微服务提供者)
service-url:
  nacos-user-service: http://nacos-payment-provider
```

由于要用Ribbon

```java
@Configuration
public class ApplicationContextConfig {

    @Bean
    @LoadBalanced
    public RestTemplate getRestTemplate(){
        return new RestTemplate();
    }
}
```

业务类

```java
    @RequestMapping("/consumer/fallback/{id}")
//    @SentinelResource(value = "fallback") //没有配置
//    @SentinelResource(value = "fallback",fallback = "handlerFallback") //fallback只负责业务异常
//    @SentinelResource(value = "fallback",blockHandler = "blockHandler") //blockHandler只负责sentinel控制台配置违规
    @SentinelResource(value = "fallback",fallback = "handlerFallback",
            blockHandler = "blockHandler",
            exceptionsToIgnore = {IllegalArgumentException.class})
    public CommonResult<Payment> fallback(@PathVariable("id") Long id){

        CommonResult<Payment> result = restTemplate.getForObject(SERVICE_URL + "/paymentSQL/" + id, CommonResult.class, id);

        if (id == 4) {
            throw new IllegalArgumentException("IllegalArgumentException,非法参数异常");
        } else if (result.getData() == null) {
            throw new NullPointerException("NullPointerException,该ID没有对应记录，空指针异常");
        }
        return result;
    }

    public CommonResult handlerFallback(@PathVariable("id") Long id, Throwable e) {
        Payment payment = new Payment(id, "null");
        return new CommonResult(444, "兜底异常handlerFallback，exception内容" + e.getMessage(), payment);
    }

    public CommonResult blockHandler(@PathVariable("id") Long id, BlockException blockException) {
        Payment payment = new Payment(id, "null");
        return new CommonResult(445, "blockHandler-sentinel限流，无此流水：blockException" + blockException.getClass().getCanonicalName(), payment);
    }
```



# 测试

没有任何配置

只配置fallback

只配置blockHandler

fallback和blockHandler都配置

忽略属性



## 测试

http://localhost:84/consumer/fallback/1

```json
{"code":200,"message":"from mysql,serverPort: 9003","data":{"id":1,"serial":"28a8c1e3bc2742d8848569891fb42181"}}
```

http://localhost:84/consumer/fallback/4

没有配置

```
Whitelabel Error Page
This application has no explicit mapping for /error, so you are seeing this as a fallback.

Tue Apr 14 23:21:08 CST 2020
There was an unexpected error (type=Internal Server Error, status=500).
IllegalArgumentException,非法参数异常
java.lang.IllegalArgumentException: IllegalArgumentException,非法参数异常
	at com.atguigu.springcloud.controller.CircleBreakerController.fallback(CircleBreakerController.java:36)
```

http://localhost:84/consumer/fallback/4

有配置

```json
{"code":444,"message":"兜底异常handlerFallback，exception内容IllegalArgumentException,非法参数异常","data":{"id":4,"serial":"null"}}
```

http://localhost:84/consumer/fallback/5

```json
{"code":444,"message":"兜底异常handlerFallback，exception内容NullPointerException,该ID没有对应记录，空指针异常","data":{"id":5,"serial":"null"}}
```

fallback和blockHandler都配置

`若blockHandler和fallback都进行了配置，则被限流降级而抛出 BlockException 只会进入 blockHandler处理逻辑`

忽略属性

![](https://xinqianpingtaib2btest.oss-cn-shenzhen.aliyuncs.com/xinqianpingtaib2btest/blogimg/2020/微信截图_20200414235114.jpg)

http://localhost:84/consumer/fallback/4

![](https://xinqianpingtaib2btest.oss-cn-shenzhen.aliyuncs.com/xinqianpingtaib2btest/blogimg/2020/微信截图_20200414235424.jpg)