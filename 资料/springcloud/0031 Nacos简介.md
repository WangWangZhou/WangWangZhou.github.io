# Nacos简介

![](https://xinqianpingtaib2btest.oss-cn-shenzhen.aliyuncs.com/xinqianpingtaib2btest/blogimg/2020/微信截图_20200411192220.jpg)



注册中心比较

![](https://xinqianpingtaib2btest.oss-cn-shenzhen.aliyuncs.com/xinqianpingtaib2btest/blogimg/2020/微信截图_20200411192924.jpg)

# 服务提供者

 http://192.168.184.1:8848/nacos/index.html

## 新建Module

cloudalibaba-provider-payment9001

## POM



## YML

## 主启动



vm options:  `-DServer.port=9001`

编辑启动项，

![](https://xinqianpingtaib2btest.oss-cn-shenzhen.aliyuncs.com/xinqianpingtaib2btest/blogimg/2020/微信截图_20200411213207.jpg)

点击Copy Configuration，

![](https://xinqianpingtaib2btest.oss-cn-shenzhen.aliyuncs.com/xinqianpingtaib2btest/blogimg/2020/微信截图_20200411213108.jpg)







## 业务类

## 测试

[服务列表](http://192.168.184.1:8848/nacos/index.html#/serviceManagement?dataId=&group=&appName=&serverId=&namespace=)

![](https://xinqianpingtaib2btest.oss-cn-shenzhen.aliyuncs.com/xinqianpingtaib2btest/blogimg/2020/微信截图_20200411221254.jpg)





http://localhost:83/consumer/payment/nacos/13

nacos registry, serverPort: 9002	id13



# Nacos和CAP

Nacos与其他注册中心特性对比

![](https://xinqianpingtaib2btest.oss-cn-shenzhen.aliyuncs.com/xinqianpingtaib2btest/blogimg/2020/微信截图_20200411221748.jpg)



# Nacos配置

https://nacos.io/zh-cn/docs/what-is-nacos.html

http://localhost:3377/config/info



nacos-config-client-dev.yml

```yaml
config:
    info: nacos config center,version = 1
```

配置中心小总结

![](https://xinqianpingtaib2btest.oss-cn-shenzhen.aliyuncs.com/xinqianpingtaib2btest/blogimg/2020/微信截图_20200411223849.jpg)



## Group方案

bootstrap+application方案

![](https://xinqianpingtaib2btest.oss-cn-shenzhen.aliyuncs.com/xinqianpingtaib2btest/blogimg/2020/微信截图_20200411230251.jpg)

## Namespace方案

命名空间方案

![](https://xinqianpingtaib2btest.oss-cn-shenzhen.aliyuncs.com/xinqianpingtaib2btest/blogimg/2020/微信截图_20200411231356.jpg)



![](https://xinqianpingtaib2btest.oss-cn-shenzhen.aliyuncs.com/xinqianpingtaib2btest/blogimg/2020/微信截图_20200412174158.jpg)