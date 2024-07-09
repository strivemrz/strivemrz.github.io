---
title: hystrix服务熔断与服务降级
categories: springcloud
tags:
  - springcloud

excerpt: ''

cover: https://scpic.chinaz.net/files/pic/pic9/202007/apic26293.jpg

---

#### 分布式系统面临的问题

复杂分布式体系结构中的应用程序有数十个依赖关系，每个依赖关系在某些时候将不可避免的失败。



#### 什么是Hystrix

Hystrix是一个用于处理分布式系统的延迟和容错的开源库，在分布式系统里，许多依赖不可避免的会调用失败比如超时，异常等，Hystrix能够保证在一个依赖出问题的情况下，不会导致整体服务失败，避免级联故障，以提高分布式系统的弹性。

"断路器"本身是一种开关装置，当某个服务单元发生故障之后，通过断路器的故障监控(类似熔断保险丝)，向调用方返回一个服务预期的，可处理的备选响应(FallBack)，而不是长时间的等待或者抛出调用方法无法处理的异常，这样就可以保证服务调用方的线程不会被长时间，不必要的占用，从而避免了故障在分布式系统中的蔓延，乃至雪崩。

#### 服务熔断

> 熔断(**服务端**)是为了防止服务端被动出现异常。降级(**客户端**)是再当服务器遇到洪流被迫关闭，调用兜底方案(合理分配服务器资源)

##### 什么是服务熔断

熔断机制是对应雪崩效应的一种微服务链路保护机制。

当扇出链路的某个微服务不可用或者响应时间太长，会进行服务的降级，进而熔断该节点微服务的调用，快速返回错误的响应信息。当检测到该节点微服务调用响应正常后恢复调用链路。在SpringCloud框架里熔断机制通过Hystrix实现，Hystrix会监控微服务间调用的状况，当失败调用到一定阈值，却省是5秒内20次调用失败就会启动熔断机制。熔断机制的注解是@HystrixCommand。



#### 服务雪崩

一个服务失败(调用时间过长或失败)，导致整条链路的服务都失败的情形，称之为服务雪崩。

多个微服务之间调用的时候，加入微服务A调用微服务B和微服务C，微服务B和微服务C又调用其他的微服务，这就是所谓的“扇出”，如果扇出的链路上某个微服务的调用响应时间过长或者不可用，对微服务A的调用就会占用越来越多的系统资源，进而引起系统崩溃，所谓的"雪崩效应"。

对于高流量的应用来说，单一的后端依赖可能会导致所有的服务器上的所有资源都在几秒钟内饱和。比失败跟糟糕的是，这些应用程序还可能导致服务之间的延迟增加，备份队列，线程和其他系统资源紧张，导致整个系统发生更多的级联故障，这些都表示需要对故障进行隔离和管理，以便单个依赖关系的失败，不能取消整个应用程序和系统。

#### 服务雪崩的解决方案

- 应用扩容：加机器，升级硬件
- 流量控制：限流(超出限定流量，返回让用户稍后再试)
- 缓存：将用户访问量大的数据放入缓存中，减少数据库访问。
- 服务降级：
  - 当下游的服务响应过慢，下游服务主动停掉一些不太重要的业务，释放出服务器资源，增加响应速度。
  - 当下游的服务因为某种原因不可用，上游主动调用本地的一些降级逻辑，避免卡顿，迅速返回给用户。
- 服务熔断：不调用该失败的服务，直接返回，快速释放资源。如果目标服务情况好转则恢复调用。



#### 服务熔断样例(服务端)

- 导入hystrix依赖

```xml
<!-- 服务熔断 -->
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-netflix-hystrix</artifactId>
</dependency>
```

- 启动类开启hystrix

```java
@SpringBootApplication
@EnableEurekaClient
@EnableDiscoveryClient
@EnableHystrix
public class SpringApplicationStar2 {
    public static void main(String[] args) {
        SpringApplication.run(SpringApplicationStar2.class,args);
    }
}
```

- 接口处声明@HystrixCommand注解

```java
@GetMapping("/getAdminById/{id}")
@HystrixCommand(fallbackMethod = "getAdminByIdSpare") //当发生异常时调用备用方法
public Admin getAdminById(@PathVariable("id") String id) throws Exception {
    Admin adminById = adminService.findAdminById(id);
    if(adminById==null){
        throw new Exception("用户未找到");
    }
    return adminById;
}

public Admin getAdminByIdSpare(@PathVariable("id") String id){
    Admin admin=new Admin();
    admin.setId(id);
    return admin;
}
```



#### 服务降级样例(客户端)

- 导入feign依赖

```xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-openfeign</artifactId>
</dependency>
```

- 开启feign.hystrix服务降级

```yml
#开启feign.hystrix服务降级
feign:
  hystrix:
    enabled: true
```

- 指定降级配置类

```java
@FeignClient(name = "SPRINGCLOUD-PROVIDE",path = "Admin",fallbackFactory = AdminClientFailService.class)
@Repository
public interface AdminClientService {

    @GetMapping("/getAdminById/{id}")
    public Admin findAdminById(@PathVariable("id") Long id);

    @RequestMapping("/getSuccessInfo")
    public Result getResultInfo();

    @PostMapping("/getResultS")
    public Results getResultS();
}
```

- 编写降级配置类

```java
@Component
public class AdminClientFailService implements FallbackFactory {
    @Override
    public Object create(Throwable throwable) {
        return new AdminClientService() {
            @Override
            public Admin findAdminById(Long id) {
                Admin admin=new Admin();
                admin.setUsername("服务降级--->该服务暂时不可用");
                return admin;
            }
            @Override
            public Result getResultInfo() {
                return null;
            }
            @Override
            public Results getResultS() {
                return null;
            }
        };
    }
}
```