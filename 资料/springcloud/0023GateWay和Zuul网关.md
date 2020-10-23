# GateWay和Zuul网关



# Zuul网关

概述简介

路由基本配置

路由访问映射规则

查看路由信息

过滤器



# Gateway新一代网关

概述简介

三大核心概念

Gateway工作流程

入门配置

通过微服务名实现动态路由

Predicate的使用

Filter的使用



# 微服务架构中网关在哪里

![](https://xinqianpingtaib2btest.oss-cn-shenzhen.aliyuncs.com/xinqianpingtaib2btest/blogimg/2020/微信截图_20200409150541.jpg)

Spring Cloud Gateway特性

![](https://xinqianpingtaib2btest.oss-cn-shenzhen.aliyuncs.com/xinqianpingtaib2btest/blogimg/2020/微信截图_20200409150934.jpg)

Spring Cloud Gateway与Zuul区别

![](https://xinqianpingtaib2btest.oss-cn-shenzhen.aliyuncs.com/xinqianpingtaib2btest/blogimg/2020/微信截图_20200409151203.jpg)



带cookie访问

`curl http://localhost:9527/payment/lb  --cookie "username=zzyy"`

带请求头访问

`curl http://localhost:9527/payment/lb  -H "X-Request-Id:123"`



Gateway的Filter

```jva
@Component
@Slf4j
public class MyLogGateWayFilter implements GlobalFilter, Ordered {


    @Override
    public Mono<Void> filter(ServerWebExchange exchange, 
    GatewayFilterChain chain) {

        log.info("*****come in My LogGateWayFilter: "+ new Date());
        String uname = exchange.getRequest()
        .getQueryParams().getFirst("uname");

        if(uname==null){
            log.info("****  用户名为null 非法用户");
            exchange.getResponse()
            .setStatusCode(HttpStatus.NOT_ACCEPTABLE);
            return exchange.getResponse().setComplete();
        }
        return chain.filter(exchange);
    }

    @Override
    public int getOrder() {//加载过滤器的顺序
        return 0;
    }
}

```

访问`http://localhost:9527/payment/lb?uname=z3`