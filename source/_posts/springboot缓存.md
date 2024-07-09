---
updated: 2022-12-05
created: 2022-12-05
title: springboot缓存
categories: springBoot
tags:
  - springBoot
excerpt: ''
cover: https://img.tt98.com/d/file/tt98/2020041317019386/5e9416c62e6be.jpg
---

## 简介

缓存是一种介于数据永久存储介质与应用程序之间的数据临时存储介质，使用缓存可以有效的减少低速数据读取过程的次数(磁盘io)，提高系统性能。此外缓存不仅可以用于提高永久性存储介质的数据读取效率，还可以提供临时数据存储空间。

## springboot内置缓存

### 导入依赖

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-cache</artifactId>
</dependency>
```

### 开启缓存

```java
@SpringBootApplication
@MapperScan("com.mybatisplus.mapper")
@EnableTransactionManagement
@EnableCaching
public class MybatisPlusTestApplication {

    public static void main(String[] args) {
//        SpringApplication.run(MybatisPlusTestApplication.class, args);
        SpringApplication application = new SpringApplication(MybatisPlusTestApplication.class);
        application.addListeners(new FirstLinster());
        application.run(args);
    }
}
```

### service层缓存注解

```java
@Service
@CacheConfig(cacheNames = "adminCache")
public class AdminServiceImpl extends ServiceImpl<AdminMapper, Admin>
    implements AdminService{

    @Autowired
    private AdminMapper adminMapper;

    @Override
    @Cacheable(key = "#id")//value 缓存名称 缓存实际上是Map存储 key:id,value:查询数据
    public Admin selectAdminByID(Long id) {
        return adminMapper.selectById(id);
    }

    @Override
    @CacheEvict(key = "#admin.uid") //修改了之后清除指定缓存(类型要和存储时候一致)
    public Integer updateAdminByID(Admin admin) {
        System.out.println(admin.getUid());
        return adminMapper.updateById(admin);
    }
}
```

## Redis缓存

springboot默认使用的是lettucs作为redis客户端连接技术

lettucs与jedis对比

- jedis连接Redis服务器是直连模式，当多线程模式下使用jedis会存在线程安全问题，解决方案可以通过配置连接池使每个连接专用
- lettcus基于Netty框架进行Redis服务器连接，底层设计采用StatefulRedisConnection。StatefulRedisConnection是线程安全的，可以保障并发访问安全问题，所以一个连接可以被多个线程复用。

### 导入依赖

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-redis</artifactId>
</dependency>
<!-- jedis客户端 如果使用lettuce可以不写 -->
<dependency>
    <groupId>redis.clients</groupId>
    <artifactId>jedis</artifactId>
</dependency>
```

### 配置Redis缓存

```yml
spring:
  datasource:
    url: jdbc:mysql://localhost:3306/mybatisplus?useSSL=false&useUnicode=true&characterEncoding=utf-8&serverTimezone=GMT%2B8
    type: com.alibaba.druid.pool.DruidDataSource
    username: root
    password: 123456
    driver-class-name: com.mysql.cj.jdbc.Driver
  redis:
    client-type: jedis #设置redis客户端类型
    host: 192.168.0.103
    port: 6379
    password: 123456
    connect-timeout: 3000ms
  cache:
    type: redis #使用redis缓存
    redis:
      use-key-prefix: false #缓存前缀
      key-prefix:
      cache-null-values: true #缓存null true
      time-to-live: 10s #缓存时间
```

### 开启缓存

```java
@SpringBootApplication
@MapperScan("com.mybatisplus.mapper")
@EnableTransactionManagement
@EnableCaching //开启缓存
public class MybatisPlusTestApplication {

    public static void main(String[] args) {
        SpringApplication application = new SpringApplication(MybatisPlusTestApplication.class);
        application.addListeners(new FirstLinster());
        application.run(args);
    }
}
```

### 编写service缓存类

```java
@Service
@CacheConfig(cacheNames = "codeMess")
public class CodeServiceImpl {

    @CachePut(key = "#id")
    public String createCode(String id){
        int code = (int)(Math.random() * 9000 + 1000);
        System.out.println(code);
        return String.valueOf(code);
    }
    @Cacheable(key = "#id")
    public String getCode(String id){ //有id缓存直接取
        return null;
    }
}
```