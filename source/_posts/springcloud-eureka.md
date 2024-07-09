---
title: eureka服务注册与发现
categories: springcloud
tags:
  - springcloud

excerpt: ''

cover: https://uploadfile.bizhizu.cn/2017/0706/20170706034625699.jpg

---

#### Eureka服务注册与发现

> Eureka是Netflix的一个子模块，也是核心模块之一，Eureka是一个基于Rest的服务，用于定位服务，以实现云端中间层服务发现和故障转移，有了服务发现与注册，只需要使用服务的标识符，就可以访问到服务，而不需要修改服务调用的配置文件了，功能类似于Dubbo的注册中心，比如Zookeeper

#### Eureka的基本架构

- springcloud封装了NetFlix公司开发的Eureka模块来实现服务注册和发现
- Eureka采用了C-S架构设计，EurekaServer作为服务注册功能的服务器，也是服务注册中心
- 而系统中的其他微服务使用Eureka的客户端连接到EurekaServer并维持心跳连接，这样系统的维护人员就可以通过EurekaServer来监控系统中各个微服务是否正常运行，springcloud的一些其他模块(比如Zuul)就可以通过EurekaServer来发现系统中的其他微服务，并执行相关逻辑
- Eureka Server提供服务注册服务，各个节点启动后，会在Eureka Server中进行注册，这样Eureka Server中的服务注册表中将会存所有可用服务节点的信息，服务节点的信息可以在界面中直观的看到。
- Eureka Client是一个Java客户端，用于简化Eureka Server的交互，客户端同时也具备一个内置的，使用轮询负载算法的负载均衡器。在应用启动后，将会向Eureka Server发送心跳(默认周期 30秒)。如果Eureka Server在多个心跳周期内没有接收到某个节点的心跳。Eureka Server将会从服务注册表中把这个服务节点移除掉(默认周期 90秒)

#### Eureka自我保护机制

> 某一时刻一个微服务不可用，eureka不会立即清理，依旧会对该微服务的信息进行保存

- 默认情况下，如果Eureka在一定时间内没有接收到某个微服务实例的心跳，EurekaServer将会注销该实例(默认90秒)。但是当网络分区故障发生时，微服务与Eureka之间无法正常正常通信，此时本不应该注销这个服务(此时这个服务是健康的)。Eureka通过自我保护机制来解决这个问题。当EurekaServer节点在短时间内丢失过多客户端时(可能发生网络分区故障)，那么这个节点就会进入自我保护模式。一旦进入该模式，EurekaServer就会保护服务注册表中的信息，不再删除服务注册表中的数据(也就是不会注销任何微服务)。当网络故障恢复后，该Eureka节点会自动退出自我保护模式。
- 在自我保护模式中，EurekaServer会保护服务注册表的信息，不再注销任何服务实例。当他的心跳数重新恢复到阈值以上时，该EurekaServer节点就会退出自我保护模式。宁可保留错误的服务注册信息，也不盲目注销任何可能健康的服务实例。
- 再SpringCloud中可以使用eureka.server.enable-self-preservation: false禁用自我保护模式。

#### CAP原则

> Zookeeper是CP，Eurken是AP

CAP原则包含 一致性(Consistency)，可用性(Availability)，分区容错性(Partition tolerance)

三选二 AP，AC，CP，不存在CAP

一致性(C)：在分布式系统中的所有数据备份，在同一时刻是否有同样的值，即写操作之后的读操作，必须返回该值。分为弱一致性，强一致性，和最终一致性

可用性(A)：在集群中一部分节点故障后，集群整体是否还能响应客户端的读写请求。(对数据更新具备高可用性)

分区容忍性(P)：以实际效果而言，分区相当于对通信的时限要求。系统如果不能在时限内达成数据一致性，就意味着发生了分区的情况，必须就当前操作在C和A之间做出选择。

#### Eureka样例

- 服务端代码

> 导入依赖

```xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-eureka-server</artifactId>
    <version>1.4.6.RELEASE</version>
</dependency>
```

> 编写eureka服务端配置

```yml
#eureka配置
eureka:
  instance:
    hostname: springcloud-eureka-7001 #eureka服务端的实例名称
  client:
    register-with-eureka: false # 表示是否向eureka注册中心注册自己
    fetch-registry: false #fetch-registry 如果为false，则表示自己为注册中心
    service-url: #监控页面
      defaultZone: http://localhost:7002/eureka/ #如果是集群 这里写其他eureka地址，中间用'，'隔开
```

> 开启erureka服务端

```java
@SpringBootApplication
@EnableEurekaServer
public class SpringcloudEurekaServer {
    public static void main(String[] args) {
        SpringApplication.run(SpringcloudEurekaServer.class,args);
    }
}
```

- 客户端代码(服务提供方)

> 导入依赖

```xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
    <version>1.4.6.RELEASE</version>
</dependency>
```

```xml
<!-- 健康监控 -->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-actuator</artifactId>
</dependency>
```

> 客户端配置

```yml
eureka:
  client:
    service-url:
      defaultZone: http://localhost:7001/eureka/,http://localhost:7002/eureka #eureka注册中心的地址,这里是集群样例
  instance:
    instance-id: springcloud-provider #服务的显示信息

#服务具体信息
info:
  app.name: springcloud-provider
  company: springcloud-actuator
```

> 开启eureka客户端

```java
@SpringBootApplication
@EnableEurekaClient
@EnableDiscoveryClient //使用DiscoveryClient 可以获取客户端信息
public class SpringApplicationStar {
    public static void main(String[] args) {
        SpringApplication.run(SpringApplicationStar.class,args);
    }
}
```