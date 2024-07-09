---
title: config远程服务配置
categories: springcloud
tags:
  - springcloud

excerpt: ''

cover: https://img.tt98.com/d/2020/2020061918007905/5eec872f4f143.jpg

---

#### 什么是springcloud config分布式配置中心

> 在分布式系统中，由于微服务的数量巨多，每个微服务都有对应的配置信息文件才能运行，如果我们每一个微服务都自己带一个application.yml，那么上百个的配置文件修改起来，会让我们非常的繁琐，为了方便服务配置文件统一管理，实时更新，springcloud提供了ConfigServer来解决这个问题

springcloud config是一个解决分布式系统的配置解决方案，它包含了server和client两个部分。

server用来获取远程的配置信息(默认为git仓库)，并且以接口的形式提供出去

client根据server提供的接口读取配置文件，以便于初始化自己的应用。



springcloud config为微服务架构中各个微服务提供集中化的外部支持，配置服务器为各个不同的微服务应用的所有环节提供了一个中心化的外部配置

springcloud config支持配置服务放在配置服务的内存中(即本地)，也支持放在远程git仓库中

在springcloud config组件中，分两个角色，一是服务端config server，二是客户端config client。

- 服务端：分布式配置中心，它是一个独立的微服务应用，用来连接配置服务器并为客户端提供获取配置信息，加密，解密信息等访问接口
- 客户端：通过指定的配置中心来管理应用资源，以及与业务相关的配置内容，并在启动的时候从配置中心获取和加载配置信息。

#### 注意事项

- bootstrap.yml配置文件加载先于application.yml配置文件
- 与springcloud config相关的属性必须配置在bootstrap.yml中
- server(服务端)从git仓库中获取各个配置文件信息
- bootstrap系统级别配置，application用户级别配置，系统级别优先于用户级别加载



#### springcloud config分布式配置中心能干什么

- 集中式管理配置文件
- 不同环境，不同配置，动态化的配置更新，分环境部署，比如 /dev /test /prod /beta /release
- 运行期间动态调整配置，不再需要在每个部署的机器上编写配置文件，服务会向注册中心统一拉取配置自己的信息
- 当配置发生变动时，config服务端不需要重启，即可感知到配置的变化，并应用新的配置
- 将配置信息以REST接口的形式暴露
- 修改config服务器配置，需要重启各个服务模块，获取最新配置信息

#### 样例(git)

##### 服务端

- 导入依赖

```xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-config-server</artifactId>
</dependency>
```

- 启动类开启config

```java
@SpringBootApplication
@EnableConfigServer
public class SpringcloudConfig {
    public static void main(String[] args) {
        SpringApplication.run(SpringcloudConfig.class,args);
    }
}
```

- 配置config的git地址

```yml
server:
  port: 3344
#spring相关配置
spring:
  cloud:
    config:
      server:
        git:
          uri: https://gitee.com/north_z/test-repository.git #git仓库地址
          search-paths: GitTest #路径
          default-label: config-test #分支名称
          username: 		#用户名密码，如果设置为公有仓库，可以不用写
          password: 
  #解决获取到的配置文件乱码
  http:
    encoding:
      charset: utf-8
      enabled: true
      force: true
  application:
    name: springcloud-config-server-3344
#eureka相关配置
eureka:
  client:
    service-url:
      defaultZone: http://localhost:7001/eureka/,http://localhost:7002/eureka
  instance:
    instance-id: config服务端

```

- 浏览器访问测试

~~~dtd
http://localhost:3344/application-dev.yml

#匹配规则
label:分支
/{application}/{profile}[/{label}]
/{application}-{profile}.yml
/{label}/{application}-{profile}.yml
/{application}-{profile}.properties
/{label}/{application}-{profile}.properties
~~~

##### 客户端

- 导入依赖

```xml
<!-- config客户端配置 -->
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-config</artifactId>
</dependency>
<!-- eureka客户端相关配置 -->
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
</dependency>
```

- 在bootstrap中配置config服务端

```yml
spring:
  cloud:
    config:
      uri: http://localhost:3344 #config服务端ip
      name: config-client #config服务器文件名，不用写后缀
      profile: dev #启用dev环境
      label: config-test #config-test分支，如果服务端指定分支，这里可以不

```



#### 读取配置文件乱码问题

- 实现PropertySourceLoader接口

```java
package com.config.config;

import lombok.extern.slf4j.Slf4j;
import org.springframework.boot.env.OriginTrackedMapPropertySource;
import org.springframework.boot.env.PropertySourceLoader;
import org.springframework.core.io.Resource;

import java.io.IOException;
import java.io.InputStream;
import java.io.InputStreamReader;
import java.nio.charset.StandardCharsets;
import java.util.Collections;
import java.util.List;
import java.util.Map;
import java.util.Properties;

@Slf4j
public class PropertySource implements PropertySourceLoader {
    @Override
    public String[] getFileExtensions() {
        return new String[]{"properties","xml"};
    }

    @Override
    public List<org.springframework.core.env.PropertySource<?>> load(String name, Resource resource) throws IOException {
        Map<String, ?> properties = loadProperties(resource);
        if(properties.isEmpty()){
            return Collections.emptyList();
        }
        return Collections.singletonList(new OriginTrackedMapPropertySource(name,properties));
    }
    private Map<String,?> loadProperties(Resource resource) {
        Properties properties=new Properties();
        InputStream inputStream=null;
        try {
            inputStream=resource.getInputStream();
            properties.load(new InputStreamReader(inputStream, StandardCharsets.UTF_8));
        } catch (IOException e) {
            e.printStackTrace();
        }finally {
            if(inputStream!=null){
                try {
                    inputStream.close();
                } catch (IOException e) {
                    log.error("close IO failure ....",e);
                }
            }
        }
        return (Map)properties;
    }
}
```

- 在resource下创建META-INF，创建spring.factories

```properties
org.springframework.boot.env.PropertySourceLoader=com.config.config.PropertySource
```

- 在application.properties配置文件中添加配置

```properties
spring.cloud.config.server.git.search-paths=GitTest
spring.cloud.config.server.git.uri=https://gitee.com/north_z/test-repository.git
spring.cloud.config.server.git.default-label=config-test
#添加如下配置
spring.http.encoding.charset=utf-8
spring.http.encoding.enabled=true
spring.http.encoding.force=true
```

