# OpenFeign是什么

![](https://xinqianpingtaib2btest.oss-cn-shenzhen.aliyuncs.com/xinqianpingtaib2btest/blogimg/2020/微信截图_20200406214725.jpg)



## Feign能干什么

![](https://xinqianpingtaib2btest.oss-cn-shenzhen.aliyuncs.com/xinqianpingtaib2btest/blogimg/2020/微信截图_20200406215009.jpg)





## Feign与OpenFeign区别

![](https://xinqianpingtaib2btest.oss-cn-shenzhen.aliyuncs.com/xinqianpingtaib2btest/blogimg/2020/微信截图_20200406215619.jpg)



**OpenFeign使用步骤**

```xnl
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-openfeign</artifactId>
        </dependency>
```

**接口+注解  微服务调用接口+@FeignClient**



**新建cloud-consumer-feign-order80**

**POM**

**YML**

**主启动**

添加`@EnableFeignClients`注解。

```java
@SpringBootApplication
@EnableFeignClients
public class OrderFeignMain80 {

    public static void main(String[] args) {
        SpringApplication.run(OrderFeignMain80.class,args);
    }
}
```

**业务类**

```java
@Component
@FeignClient(value = "CLOUD-PAYMENT-SERVICE")
public interface PaymentFeignService {

    @GetMapping(value = "/payment/get/{id}")
    CommonResult<Payment> getPaymentById(@PathVariable("id") Long id);

    @GetMapping(value = "/payment/feign/timeout")
    String paymentFeignTimeOut();

}

```

controller

```java
@RestController
@Slf4j
public class OrderFeignController {

    @Resource
    private PaymentFeignService paymentFeignService;

    @GetMapping(value = "/consumer/payment/get/{id}")
    public CommonResult<Payment> getPaymentById(@PathVariable("id") Long id){
        return paymentFeignService.getPaymentById(id);
    }

    @GetMapping(value = "/consumer/payment/feign/timeout")
    public String paymentFeignTimeOut(){
        //客户端默认等待1秒钟
        return paymentFeignService.paymentFeignTimeOut();
    }
}

```



**测试**

http://192.168.184.1/consumer/payment/feign/timeout

![](https://xinqianpingtaib2btest.oss-cn-shenzhen.aliyuncs.com/xinqianpingtaib2btest/blogimg/2020/微信截图_20200406235949.jpg)

超时，因为客户端默认只等待1秒钟，但是服务器3秒钟才返回。测试通过。





**小总结**



OpenFeign超时控制





OpenFeign日志打印功能

