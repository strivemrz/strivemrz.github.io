---
title: Feign远程服务调用
categories: springcloud
tags:
  - springcloud

excerpt: ''

cover: https://img.zcool.cn/community/01c8f15aeac135a801207fa16836ae.jpg@1280w_1l_2o_100sh.jpg

---
> > 在开发Spring Cloud微服务的时候，服务之间都是以HTTP接口的形式对外提供服务的，因此消费者在进行调用的时候，底层就是通过HTTP Client的这种方式进行访问。我们也可以使用JDK原生的 URLConnection，Apache的HTTP Client，Netty异步Http Client，Spring的RestTemplate去实现服务间的调用。但是最方便，最优雅的方式是通过Spring Cloud Open Feign进行服务间的调用，Spring Cloud对Feign进行了增强，使Feign支持SpringMvc的注解，并整合了Ribbon等，从而让Feign的使用更加方便。

**通过Feign实现远程服务调用，通过Ribbon实现负载均衡，通过Eureka进行服务注册**

#### 什么是Feign

Feign是Netflix开发的声明式，模板化的Http客户端，Feign可以帮助我们更快捷，优雅的调用HTTP API

Feign是一种声明式，模板化的HTTP客户端。在SpringCloud中使用Feign，可以做到使用HTTP请求访问远程服务，就像调用本地方法一样，开发者完全感知不到这是在调用远程方法，更感知不到在访问HTTP请求。

特性：

- 可拔插的注解支持，包括Feign注解和AX-RS注解。
- 支持可拔插的HTTP编码器和解码器。
- 支持Hystrix和他的Fallback
- 支持Ribbon的负载均衡

#### Feign工作原理

- 在开发微服务应用时，会在主程序入口添加@EnableFeignClients注解，开启对Feign Client扫描加载处理。根据Feign Client的开发规范，定义接口并加@FeignClient注解。
- 当程序启动时，会进行包扫描，扫描所有@FeignClient的注解得类，并将这些信息注入Spring IOC容器中。当定义的Feign接口中的方法被调用时，通过JDK的代理方式，来生成具体的RequestTemplate。当生成代理时，Feign会为每个接口方法创建一个RequestTemplate对象，该对象封装了HTTP请求需要的全部信息，如请求参数名，请求方法等信息都是在这个过程中确定的。
- 然后由RequestTemplate生成Request，然后把Request交给Client去处理，这里的Client可以是JDK原生的URLConnection，Apache的Http Client等。最后Client被封装到LoadBalanceclient类，这个类结合Ribbon负载均衡发起服务之间的调用。

#### @FeignClient注解

- name：指定Feign Client的名称，如果项目使用了Ribbon，name属性会作为微服务的名称，用于服务发现。
- url：url一般用于调试，可以手动指定@FeignClient调用的地址。
- decode404：当发生404错误时，如果该字段为true，会调用decoder进行编码，否则抛出FeignException。
- configuration：Feign配置类，可以自定义Feign的Encoder，Decoder，LogLevel，Contract。
- fallback：定义容错的处理类，当调用远程接口失败或超时时，会调用对应接口的容错逻辑，fallback指定的类必须实现@FeignClient标记的接口。
- fallbackFactory：工厂类，用于生成fallback类示例，通过这个属性我们可以实现每个接口的容错逻辑，减少重复的代码。
- path：定义当前FeignClient的统一前缀。

#### 样例

##### 导入依赖

```xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-openfeign</artifactId> <!-- 使用父module版本管理 -->
</dependency>
```

##### 开启Feign

```java
@SpringBootApplication
@EnableEurekaClient //开启eurken客户端
@EnableFeignClients(basePackages = {"com.springcloud.consumer"})//basePackages 是FeignClient接口类地址
public class SpringApplicationConsumerFeign {
    public static void main(String[] args) {
        SpringApplication.run(SpringApplicationConsumerFeign.class,args);
    }
}
```

##### Feign接口编写

```java
@FeignClient(name = "SPRINGCLOUD-PROVIDE")//服务提供者name
@Repository
public interface AdminClientService {

    @GetMapping("/Admin/getAdminById/{id}")
    public Admin findAdminById(@PathVariable("id") Long id);
}
```

##### Controller层调用Feign

```java
@RestController
public class IndexController {
//  RestTemplate远程接口调用
//    @Autowired
//    private RestTemplate restTemplate;
//
////    private static final String Url_Prefix="http://localhost:8081/";
//    private static final String Url_Prefix="http://SPRINGCLOUD-PROVIDE/";
//
//    @GetMapping("/api/Admin/getAdminById/{id}")
//    public Admin getAdminById(@PathVariable("id") Long id){
//        Admin admin = restTemplate.getForObject(Url_Prefix + "Admin/getAdminById/" + id, Admin.class);
//        return admin;

//  feign远程接口调用
        @Autowired
        private AdminClientService adminClientService=null;

        @GetMapping("/api/Admin/getAdminById/{id}")//这里要和服务提供者地址一样，不然会404
        public Admin getAdminById(@PathVariable("id") Long id) {
            Admin adminById = adminClientService.findAdminById(id);
            return adminById;
        }
}
```