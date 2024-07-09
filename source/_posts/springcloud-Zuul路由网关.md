---
title: Zuul路由网关
categories: springcloud
tags:
  - springcloud

excerpt: ''

cover: https://ts1.cn.mm.bing.net/th/id/R-C.2308b2986b0b5ec1414fd9b4a0e7c5f2?rik=pjUvsxcAtPz%2bYw&riu=http%3a%2f%2fpic.bizhi360.com%2fbpic%2f16%2f6816.jpg&ehk=loueoJLNs3FskRIIShZp5S4LkmOZhbQ6nxeiQWSyEDc%3d&risl=&pid=ImgRaw&r=0

---

#### 网关作用

- 统一入口：为全部服务提供一个唯一的入口，网关起到外部和内部隔离的作用，保障了后台服务的安全性。
- 鉴权校验：识别每个请求的权限，拒绝不符合要求的请求。
- 动态路由：动态的将请求路由到不同的后端集群中
- 减少客户端与服务端的耦合：服务可以独立发展，通过网关层来做映射。

#### Zuul简介

Zuul是Netflix开源的微服务网关，他可以和Eureka，Ribbon，Hystrix等组件配合使用

Zuul的核心是一系列的过滤器，这些过滤器可以完成以下功能。

- 身份认证与安全：识别每个资源的验证要求，并拒绝哪些与要求不符的请求。
- 审查与监控：在边缘位置追踪有意义的数据和统计结果，从而带来精确的生产视图。
- 动态路由：动态的将请求路由到不同的后端集群。
- 压力测试：逐渐增加指向集群的流量，以了解性能。
- 负载分配：为每一种负载类型分配对应的容量，并弃用超出限定值的请求。
- 静态响应处理：在边缘位置直接建立部分响应，从而避免其转发到内部集群。
- 多区域弹性：跨越AWS Region进行请求路由，旨在实现ELB使用的多样化，以及让系统的边缘更贴近系统的使用者。

#### Zuul的路由转发

- 路由：是指根据请求url，将请求分配到对应的处理程序。在微服务体系中，Zuul负责接收所有的请求。根据不同的url匹配规则，将不同的请求转发到不同的微服务处理。

![网关流程图.png](/2022/08/28/springcloud-Zuul路由网关/网关流程图.png)

#### 样例

- 导入依赖

```xml
<!-- zuul网关 -->
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-netflix-zuul</artifactId>
</dependency>
```

- 在启动类上开启zuul

```java
@SpringBootApplication
@EnableZuulProxy
@EnableEurekaClient
public class SpringCloudZuul {
    public static void main(String[] args) {
        SpringApplication.run(SpringCloudZuul.class,args);
    }
}
```

- zuul相关配置

```yml
server:
  port: 9527
#服务名称
spring:
  application:
    name: springcloud-zuul
#eureka相关配置
eureka:
  instance:
    instance-id: zuul9527.com
    prefer-ip-address: true #显示ip
  client:
    service-url:
      defaultZone: http://localhost:7002/eureka,http://localhost:7001/eureka #从集群中获取接口信息
info:
  app.name: zuul实例
  company.name: http://zuul.com

zuul:
  routes: #zuul路由相关配置
	#简便写法 微服务访问路径: 映射路径
#    springcloud-provide: /dept/**  
#    springcloud-provide2: /showtxt/**
    #原始写法
    mydept.serviceId: springcloud-provide #原来的微服务访问路径
    mydept.path: /dept/** #微服务映射路径
    
    showtxt.serviceId: springcloud-provide2
    showtxt.path: /showtxt/**
  ignored-services: "*"
  #  ignored-services: springcloud-provide #原路径不能访问,set类型
  prefix: /gateway #访问前缀
```

- 浏览器访问

~~~https
http://localhost:9527/gateway/dept/Admin/getAdminById/1 #http://zuul地址:zuul端口号/前缀/相关服务路径/接口地址
~~~

