# Windows版Nacos单机集群配置入门

## 架构图



![](https://xinqianpingtaib2btest.oss-cn-shenzhen.aliyuncs.com/xinqianpingtaib2btest/blogimg/2020/微信截图_20200412223635.jpg)

注意了，Nginx你也可以配置两个，避免单点故障

![](https://xinqianpingtaib2btest.oss-cn-shenzhen.aliyuncs.com/xinqianpingtaib2btest/blogimg/2020/20200323154325140.jpg)



## 单机版集群——配置

参考

https://nacos.io/zh-cn/docs/deployment.html

- 1.安装数据库，版本要求：5.6.5+
- 2.初始化mysql数据库，数据库初始化文件：nacos-mysql.sql
- 3.修改conf/application.properties文件，增加支持mysql数据源配置（目前只支持mysql），添加mysql数据源的url、用户名和密码。

```properties
spring.datasource.platform=mysql

db.num=1
db.url.0=jdbc:mysql://localhost:3306/nacos_dev?useUnicode=true&characterEncoding=utf-8&useSSL=false
db.user=root
db.password=root
```

复制三份Nacos，分别修改conf目录里面的配置文件

![](https://xinqianpingtaib2btest.oss-cn-shenzhen.aliyuncs.com/xinqianpingtaib2btest/blogimg/2020/微信截图_20200413133011.jpg)

修改端口号conf/application.properties

```properties
server.port=8848
```

修改端口号为8848，8847，8846。

配置集群配置文件

在nacos的解压目录nacos/的conf目录下，有配置文件cluster.conf，请每行配置成ip:port。（请配置3个或3个以上节点）

```
#it is ip
#example
192.168.43.67:8848
192.168.43.67:8847
192.168.43.67:8846
```

分别启动 `.\startup.cmd -m cluster`

![](https://xinqianpingtaib2btest.oss-cn-shenzhen.aliyuncs.com/xinqianpingtaib2btest/blogimg/2020/微信截图_20200413135420.jpg)

出现 INFO Nacos started successfully in cluster mode.

启动成功。

![](https://xinqianpingtaib2btest.oss-cn-shenzhen.aliyuncs.com/xinqianpingtaib2btest/blogimg/2020/微信截图_20200413135624.jpg)

输入 http://192.168.184.1:8846/nacos/index.html，访问集群管理，节点列表。可以看到节点的状态，当前leader为`192.168.184.1:8848`。

配置Nginx

修改nginx.conf为

![](https://xinqianpingtaib2btest.oss-cn-shenzhen.aliyuncs.com/xinqianpingtaib2btest/blogimg/2020/微信截图_20200413142238.jpg)

```

#user  nobody;
worker_processes  1;

#error_log  logs/error.log;
#error_log  logs/error.log  notice;
#error_log  logs/error.log  info;

#pid        logs/nginx.pid;


events {
    worker_connections  1024;
}


http {
    include       mime.types;
    default_type  application/octet-stream;

	upstream cluster{
		server 127.0.0.1:8848;
		server 127.0.0.1:8847;
		server 127.0.0.1:8846;
	}

    sendfile        on;
    #tcp_nopush     on;

    #keepalive_timeout  0;
    keepalive_timeout  65;

    #gzip  on;

    server {
        listen       1111;
        server_name  localhost;

        #charset koi8-r;

        #access_log  logs/host.access.log  main;

        location / {
            #root   html;
            #index  index.html index.htm;
			proxy_pass http://cluster;
        }
    }

}
```

启动

![](https://xinqianpingtaib2btest.oss-cn-shenzhen.aliyuncs.com/xinqianpingtaib2btest/blogimg/2020/微信截图_20200413142257.jpg)

http://192.168.184.1:1111/nacos/index.html

# 测试

![](https://xinqianpingtaib2btest.oss-cn-shenzhen.aliyuncs.com/xinqianpingtaib2btest/blogimg/2020/微信截图_20200413143002.jpg)





# 参考资料

[nginx菜鸟教程](https://www.runoob.com/linux/nginx-install-setup.html)

[Nacos集群配置](https://nacos.io/zh-cn/docs/cluster-mode-quick-start.html)

[Spring Cloud - Nacos注册中心入门单机模式及集群模式](https://www.cnblogs.com/ibigboy/p/12541928.html)

