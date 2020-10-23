# 支付服务注册进zookeeper



## 服务提供者

新建cloud-provider-payment8004

POM

YML

主启动类

Controller

启动8004注册进zookeeper

验证测试

验证测试

思考



`C:\Windows\System32\drivers\etc\hosts` ,`zkServer.cmd`,`zookeeperStarted.html`







![](https://xinqianpingtaib2btest.oss-cn-shenzhen.aliyuncs.com/xinqianpingtaib2btest/blogimg/2020/微信截图_20200405195325.jpg)



```xml
<!--SpringBoot整合zookeeper客户端-->
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-zookeeper-discovery</artifactId>
    <!--先排除自带的zookeeper3.5.3-->
    <exclusions>
        <exclusion>
            <groupId>org.apache.zookeeper</groupId>
            <artifactId>zookeeper</artifactId>
        </exclusion>
    </exclusions>
</dependency>
<!--添加zookeeper3.4.9 版本-->
<dependency>
    <groupId>org.apache.zookeeper</groupId>
    <artifactId>zookeeper</artifactId>
    <version>3.4.9</version>
</dependency>
```

启动8804主启动类

访问 http://localhost:8004/payment/zk

![](https://xinqianpingtaib2btest.oss-cn-shenzhen.aliyuncs.com/xinqianpingtaib2btest/blogimg/2020/微信截图_20200405212819.jpg)



再看一下zookeeper注册中心

```bash
[zk: localhost:2181(CONNECTED) 4] ls /
[dubbo, services, zookeeper]
[zk: localhost:2181(CONNECTED) 5] ls /services
[cloud-payment-service]
[zk: localhost:2181(CONNECTED) 6] ls /services/cloud-payment-service
[7194be4f-b004-4161-a0de-b7407a964d73]
[zk: localhost:2181(CONNECTED) 7] ls /services/cloud-payment-service/7194be4f-b004-4161-a0de-b7407a964d73
[]
[zk: localhost:2181(CONNECTED) 8]
```

get看一下内容

```bash
[zk: localhost:2181(CONNECTED) 8] get /services/cloud-payment-service/7194be4f-b004-4161-a0de-b7407a964d73
{"name":"cloud-payment-service","id":"7194be4f-b004-4161-a0de-b7407a964d73","address":"localhost","port":8004,"sslP
ort":null,"payload":{"@class":"org.springframework.cloud.zookeeper.discovery.ZookeeperInstance","id":"application-1
","name":"cloud-payment-service","metadata":{}},"registrationTimeUTC":1586092477007,"serviceType":"DYNAMIC","uriSpe
c":{"parts":[{"value":"scheme","variable":true},{"value":"://","variable":false},{"value":"address","variable":true
},{"value":":","variable":false},{"value":"port","variable":true}]}}
```

可以看到是json，格式化一下。

```json
{
    "name":"cloud-payment-service",
    "id":"7194be4f-b004-4161-a0de-b7407a964d73",
    "address":"localhost",
    "port":8004,
    "sslPort":null,
    "payload":{
        "@class":"org.springframework.cloud.zookeeper.discovery.ZookeeperInstance",
        "id":"application-1",
        "name":"cloud-payment-service",
        "metadata":{

        }
    },
    "registrationTimeUTC":1586092477007,
    "serviceType":"DYNAMIC",
    "uriSpec":{
        "parts":[
            {
                "value":"scheme",
                "variable":true
            },
            {
                "value":"://",
                "variable":false
            },
            {
                "value":"address",
                "variable":true
            },
            {
                "value":":",
                "variable":false
            },
            {
                "value":"port",
                "variable":true
            }
        ]
    }
}
```



## 服务消费者





# 服务注册中心，Eureka与Zookeeper比较

## **1. 前言**

  服务注册中心，给客户端提供可供调用的服务列表，客户端在进行远程服务调用时，根据服务列表然后选择服务提供方的服务地址进行服务调用。服务注册中心在分布式系统中大量应用，是分布式系统中不可或缺的组件，例如rocketmq的name server，hdfs中的namenode，dubbo中的zk注册中心，spring cloud中的服务注册中心eureka。

  在spring cloud中，除了可以使用eureka作为注册中心外，还可以通过配置的方式使用zookeeper作为注册中心。既然这样，我们该如何选择注册中心的实现呢？

  著名的CAP理论指出，一个分布式系统不可能同时满足C(一致性)、A(可用性)和P(分区容错性)。由于分区容错性在是分布式系统中必须要保证的，因此我们只能在A和C之间进行权衡。在此Zookeeper保证的是CP, 而Eureka则是AP。



## **2. Zookeeper保证CP**

  当向注册中心查询服务列表时，我们可以容忍注册中心返回的是几分钟以前的注册信息，但不能接受服务直接down掉不可用。也就是说，服务注册功能对可用性的要求要高于一致性。但是zk会出现这样一种情况，当master节点因为网络故障与其他节点失去联系时，剩余节点会重新进行leader选举。问题在于，选举leader的时间太长，30 ~ 120s, 且选举期间整个zk集群都是不可用的，这就导致在选举期间注册服务瘫痪。在云部署的环境下，因网络问题使得zk集群失去master节点是较大概率会发生的事，虽然服务能够最终恢复，但是漫长的选举时间导致的注册长期不可用是不能容忍的。



## **3. Eureka保证AP**

  Eureka看明白了这一点，因此在设计时就优先保证可用性。Eureka各个节点都是平等的，几个节点挂掉不会影响正常节点的工作，剩余的节点依然可以提供注册和查询服务。而Eureka的客户端在向某个Eureka注册或如果发现连接失败，则会自动切换至其它节点，只要有一台Eureka还在，就能保证注册服务可用(保证可用性)，只不过查到的信息可能不是最新的(不保证强一致性)。除此之外，Eureka还有一种自我保护机制，如果在15分钟内超过85%的节点都没有正常的心跳，那么Eureka就认为客户端与注册中心出现了网络故障，此时会出现以下几种情况： 
  \1. Eureka不再从注册列表中移除因为长时间没收到心跳而应该过期的服务 
  \2. Eureka仍然能够接受新服务的注册和查询请求，但是不会被同步到其它节点上(即保证当前节点依然可用) 
  \3. 当网络稳定时，当前实例新的注册信息会被同步到其它节点中

 因此， Eureka可以很好的应对因网络故障导致部分节点失去联系的情况，而不会像zookeeper那样使整个注册服务瘫痪。

# Zookeeper数据查看工具ZooInspector

下载[地址](https://issues.apache.org/jira/secure/attachment/12436620/)

进入目录ZooInspector\build，运行zookeeper-dev-ZooInspector.jar；

\build> java -jar zookeeper-dev-ZooInspector.jar

