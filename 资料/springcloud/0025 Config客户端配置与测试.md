# Config客户端配置与测试





bootstrap.yml

![](https://xinqianpingtaib2btest.oss-cn-shenzhen.aliyuncs.com/xinqianpingtaib2btest/blogimg/2020/微信截图_20200409225117.jpg)



http://config-3344.com:3344/master/config-prod.yml

http://localhost:3355/configInfo



手动发送POST请求刷新

`curl -X POST "http://localhost:3355/actuator/refresh"`



```
C:\Users\MagicBook>curl -X POST "http://localhost:3355/actuator/refresh"
["config.client.version","config.info"]
```

