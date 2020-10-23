# Eureka集群环境构建



## 改映射

C:\Windows\System32\drivers\etc\hosts

```
127.0.0.1 eureka7001.com
127.0.0.1 eureka7002.com
127.0.0.1 eureka7003.com
```



改yml

server7001

```yaml
server:
  port: 7001

eureka:
  instance:
    hostname: eureka7001.com # eureka服务端的实例名称
  client:
    #false表示不向注册中心注册自己
    register-with-eureka: false
    #false表示自己端就是注册中心，我的职责就是维护服务实例，并不需要去检索服务
    fetch-registry: false
    service-url:
        #设置与Eureka Server交互的地址查询服务和注册服务都需要依赖这个地址
        defaultZone: http://eureka7002.com:7002/eureka/
```

server7002

```yaml
server:
  port: 7002

eureka:
  instance:
    hostname: eureka7002.com # eureka服务端的实例名称
  client:
    #false表示不向注册中心注册自己
    register-with-eureka: false
    #false表示自己端就是注册中心，我的职责就是维护服务实例，并不需要去检索服务
    fetch-registry: false
    service-url:
        #设置与Eureka Server交互的地址查询服务和注册服务都需要依赖这个地址
        defaultZone: http://eureka7001.com:7001/eureka/
```

注意了，server1 注册在server2上，server2注册在server1上。你也可以部署3台。

访问

http://localhost:7001

http://localhost:7002

以前的也可以访问。我们试试新的。

http://eureka7001.com:7001

http://eureka7002.com:7002

# 改服务

cloud-provider-payment8001

cloud-consumer-order80

改yml

```yaml
eureka:
  client:
    #false表示不向注册中心注册自己
    register-with-eureka: true
    #false表示自己端就是注册中心，我的职责就是维护服务实例，并不需要去检索服务
    fetch-registry: true
    service-url:
      #设置与Eureka Server交互的地址查询服务和注册服务都需要依赖这个地址
      #defaultZone: http://localhost:7001/eureka
      defaultZone: http://eureka7001.com:7001/eureka,http://eureka7002.com:7002/eureka  # 集群版
```

## 测试

先启动EurekaServer,7001,7002 服务

再启动服务提供者provider 8001

再启动消费者80

![](https://xinqianpingtaib2btest.oss-cn-shenzhen.aliyuncs.com/xinqianpingtaib2btest/blogimg/2020/微信截图_20200404001410.jpg)

