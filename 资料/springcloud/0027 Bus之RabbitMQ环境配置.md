#  Bus之RabbitMQ环境配置

## RabbitMQ

安装[rabbitmq](https://www.rabbitmq.com/download.html)

`sudo docker run -it --rm --name rabbitmq -p 5672:5672 -p 15672:15672 rabbitmq:3-management`

http://IP:15672

## 安装portainer

```bash
sudo docker run -d -p 9000:9000 \
    --restart=always \
    -v /var/run/docker.sock:/var/run/docker.sock \
    --name prtainer-demo \
    docker.io/portainer/portainer
```

http://IP:9000

# 改POM

```xml
        <!--    添加消息总线RabbitMQ支持    -->
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-bus-amqp</artifactId>
        </dependency>
```

## 改bootstrap.yml

```yaml
server:
  port: 3355

spring:
  application:
    name: config-client
  cloud:
    config:
      label: master
      name: config
      profile: dev
      uri: http://localhost:3344
  rabbitmq:
      host: 192.168.43.212
      port: 15672
      username: guest
      password: guest
eureka:
  client:
    service-url:
      defaultZone: http://localhost:7001/eureka
# 暴露监控端点
management:
  endpoints:
    web:
      exposure:
        include: "*"
```

发送POST 请求

`curl -X POST "http://localhost:3344/actuator/bus-refresh"`

一次发送，处处生效。

修改hosts

```
127.0.0.1 config-3344.com
127.0.0.1 config-3355.com
127.0.0.1 config-3366.com
```

# 测试

## 打开浏览器各个页面

打开Github

https://gitee.com/wangwang2020/springcloud-config.git

总配置3344

http://config-3344.com:3344/master/config-dev.yml

客户端3355

http://config-3355.com:3355/configInfo

客户端3366

http://config-3366.com:3366/configInfo

## 测试---通知所有

### 第一步，修改Github上的配置文件，增加版本号

### 第二步，发送POST请求

发送POST 请求

`curl -X POST "http://localhost:3344/actuator/bus-refresh"`

### 第三步，配置中心页面和客户端页面查看 

客户端，

http://config-3355.com:3355/configInfo

http://config-3366.com:3366/configInfo

获取配置信息，发现都已经刷新了

总结

一次修改，广播通知，处处生效

### 只通知3355 

`curl -X POST "http://localhost:3344/actuator/bus-refresh/config-client:3355"`

# 通知总结

![](https://xinqianpingtaib2btest.oss-cn-shenzhen.aliyuncs.com/xinqianpingtaib2btest/blogimg/2020/微信截图_20200410161820.jpg)

