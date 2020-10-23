# Sentinel持久化规则

# 项目配置

cloudalibaba-sentinel-service8401

YML

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

feign:
  sentinel:
    enabled: true # 激活Sentinel 对Feign的支持
```

POM

```xml
        <dependency>
            <groupId>com.alibaba.csp</groupId>
            <artifactId>sentinel-datasource-nacos</artifactId>
        </dependency>
```

Nacos

新建配置

Data-ID

```
cloudalibaba-sentinel-service
```

Group

```
DEFAULT_GROUP
```

配置格式 

json

配置内容

```json
[
    {
        "resource":"/rateLimit/byUrl",
        "limitApp":"default",
        "grade":1,
        "count":1,
        "strategy":0,
        "controlBehavior":0,
        "clusterMode":false
    }
]
```

说明

```
resource:资源名称
limitApp:来源应用
grade:阈值类型，0表示线程数，1表示QPS
count:单机阈值
strategy:流控模式，0表示直接，1表示关联，2表示链路
controlBehavior:流控效果，0表示快速失败，1表示Warm Up,2表示排除等待
clusterMode:是否集群
```

# 测试

快速访问http://localhost:8401/rateLimit/byUrl

结果

```
Blocked by Sentinel (flow limiting)
```

停止8401再看Sentinel

重新启动8401再看Sentinel

乍一看还是没有，稍等一会儿

多次调用http://localhost:8401/rateLimit/byUrl

重新配置出现了，持久化验证通过