---
title: Ribbon负载均衡
categories: springcloud
tags:
  - springcloud

excerpt: ''

cover: https://ts1.cn.mm.bing.net/th/id/R-C.d21976011a39e03930ea634bcf9949a0?rik=J3TO%2fHI2l0DjwQ&riu=http%3a%2f%2fimage.qianye88.com%2fpic%2f50a86ca9519c5d51f894196d9d84cceb&ehk=9SJmQvzfm%2bHw5sqK2D8Jmi7fcHuHN5DBmMSWYw6%2bU%2fs%3d&risl=&pid=ImgRaw&r=0

---
#### Ribbon是什么？

Ribbon是Netflix发布的开源项目，主要功能是提供客户端的软件负载均衡算法，将NetflIx的中间层服务连接在一起。Ribbon客户端组件提供一系列完善的配置项，如连接超时，重试等。就是在配置文件中列出LB(LoadBalancer,负载均衡)后面所有的机器，Ribbon会自动的帮助你基于某种规则(如简单轮询，随机连接等) 去连接这些机器。 也很容易使用Ribbon实现自定义的负载均衡算法。

#### Ribbon能干什么？

- LB,即负载均衡，在微服务或分布式集群中经常用到的一种应用
- 负载均衡就是将用户的请求平坦的分配到多个服务器上，从而达到系统的HA(高可用)
- 常见的负载均衡有Nginx，Lvs等等
- dubbo，SpringCloud中均提供了负载均衡，SpringCloud的负载均衡算法可以自定义
- 负载均衡简单分类：
  - 集中式LB
    - 即在服务的消费方和提供方之间使用独立的LB设施，如Nginx，由该设施负责把访问的请求通过某种策略转发至服务的提供方
  - 进程式LB
    - 将LB逻辑集成到消费方，消费方从服务注册中心获知有哪些地址可用，然后自己再从这些地址中选出一个合适的服务器。
    - Ribbon就属于进程式LB，他只是一个类库，集成于消费方进程，消费方通过他来获取到服务提供方的地址

#### 自定义负载均衡策略

~~~java
@Configuration
public class ConfigBean {//@Configuration -- spring  applicationContext.xml

    /**
     * IRule:
     * RoundRobinRule 轮询策略
     * RandomRule 随机策略
     * AvailabilityFilteringRule ： 会先过滤掉，跳闸，访问故障的服务~，对剩下的进行轮询~
     * RetryRule ： 会先按照轮询获取服务~，如果服务获取失败，则会在指定的时间内进行，重试
     */
    @Bean
    public IRule myRule() {
        return new RandomRule();//使用随机策略
        //return new RoundRobinRule();//使用轮询策略
        //return new AvailabilityFilteringRule();//先过滤掉不能使用的服务再使用轮询策略
        //return new RetryRule();//使用轮询策略
    }
}

~~~

#### 样例

- 在服务消费方导入依赖(Ribbon是客户端负载均衡，需要在客户端获取到eureka注册中心信息，决定去哪个服务器请求接口)

```xml
<!-- 新版eureka-client注册了ribbon，不用再次导入 -->
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
    <version>1.4.6.RELEASE</version>
</dependency>
```

- 给RestTemplate工具类加负载均衡注解

~~~java
@Configuration
public class ConfigBean {
    @Bean
    @LoadBalanced //负载均衡
    public RestTemplate getRestTemplate(){
        return new RestTemplate();
    }
}
~~~

- 配置eureka

~~~yml
eureka:
  client:
    register-with-eureka: false #是否向eureka服务端注册
    service-url:
      defaultZone: http://localhost:7002/eureka/,http://localhost:7001/eureka #这里写eureka注册中心地址
~~~

- 启动类开启eureka客户端

~~~java
@SpringBootApplication
@EnableEurekaClient
public class SpringApplicationConsumer {
    public static void main(String[] args) {
        SpringApplication.run(SpringApplicationConsumer.class,args);
    }
}
~~~

- 配置接口请求地址

~~~java
@RestController
public class IndexController {

    @Autowired
    private RestTemplate restTemplate;

//    private static final String Url_Prefix="http://localhost:8081/";
    private static final String Url_Prefix="http://SPRINGCLOUD-PROVIDE/";//这里是服务提供方name(spring.application.name)

    @GetMapping("/api/Admin/getAdminById/{id}")
    public Admin getAdminById(@PathVariable("id") Long id){
        Admin admin = restTemplate.getForObject(Url_Prefix + "Admin/getAdminById/" + id, Admin.class);
        return admin;
    }
}
~~~

