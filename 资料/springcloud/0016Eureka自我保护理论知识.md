# Eureka自我保护理论知识

![](https://xinqianpingtaib2btest.oss-cn-shenzhen.aliyuncs.com/xinqianpingtaib2btest/blogimg/2020/微信截图_20200405172023.jpg)

## 配置



### cloud-eureka-server7001

```yaml
eureka:
  instance:
    hostname: eureka7001.com # eureka服务端的实例名称
  client:
    #false表示不向注册中心注册自己
    register-with-eureka: false
    #false表示自己端就是注册中心，我的职责就是维护服务实例，并不需要去检索服务
    fetch-registry: false
    service-url:
    #集群指向其它eureka
      #defaultZone: http://eureka7002.com:7002/eureka
    #单机就是自己
      defaultZone: http://eureka7001.com:7001/eureka/
  server:
    # 关闭自我保护机制,保证不可用服务被及时踢除
    enable-self-preservation: false
    eviction-interval-timer-in-ms: 2000
```





### cloud-provider-payment

8001和8002两个yml

```yaml
eureka:
  client:
    #false表示不向注册中心注册自己
    register-with-eureka: true
    #false表示自己端就是注册中心，我的职责就是维护服务实例，并不需要去检索服务
    fetch-registry: true
    service-url:
      #设置与Eureka Server交互的地址查询服务和注册服务都需要依赖这个地址
      defaultZone: http://localhost:7001/eureka
      #defaultZone: http://eureka7001.com:7001/eureka,http://eureka7002.com:7002/eureka  # 集群版
  instance:
    instance-id: payment8002
    prefer-ip-address: true
    #心跳检测与续约时间
    #开发时设置小些，保证服务关闭后注册中心能及时剔除服务
    #Eureka客户端向服务端发送心跳的时间间隔,单位为秒(默认是30秒)
    lease-renewal-interval-in-seconds: 2
    #Eureka服务端在收到最后一次心跳后等待时间上限，单位为秒(默认是90秒)，超时将剔除服务
    lease-expiration-duration-in-seconds: 3
```

