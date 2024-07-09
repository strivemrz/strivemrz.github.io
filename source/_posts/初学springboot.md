---
title: springBoot
categories: springBoot
tags:
  - springBoot
excerpt: ''
cover: https://www.yulumi.cn/gl/uploads/allimg/201128/161H4HI-2.jpg
---

初学springboot(shiro/redis/dubbo)

配置文件:applicationContext.xml或者application.yaml

其中application.yaml对空格要求非常严格

```yaml
#对象操作
person:
  username: 张三${random.uuid}
  date: 2022/04/30
  dog:
    name: ${person.world:旺旺碎冰冰}
    date: 2013/01/12
  list:
    - list
    - hello
  map: {k1: v1,k2: v2}
```

@ConfigurationProperties(prefix = "person")

将配置文件中的信息映射到实体类中

@PropertySource(value = "classpath:ahui.properties")指定文件类似于ssm中<context:property-placeholder location="classpath:demo.properties"/>

properties文件中文乱码

settings->file-encodings->使用utf-8

松散绑定

比如yaml中写的字段名是lst-name这个和lstName是一样的

jsr303校验

加入依赖

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-validation</artifactId>
    <version>2.3.5.RELEASE</version>
</dependency>
```

在类上面加@Validated注解

在要校验的字段上面加注解校验@Email

多环境配置

```yaml
spring:
  profiles:
    active: dev(此时采用的是application-dev.yaml文件)
```

或

```yaml
#application.yaml对空格要求非常严格
spring:
  profiles:
    active: dev
---
spring:
  profiles: dev
server:
  port: 8083
---
spring:
  profiles: test
server:
  port: 8084
```

配置项目根路径

```yaml
server:
  servlet:
    context-path: /myPro
```

页面国际化

![image-20220430212355544](C:\Users\19651\AppData\Roaming\Typora\typora-user-images\image-20220430212355544.png)

![image-20220430212430066](C:\Users\19651\AppData\Roaming\Typora\typora-user-images\image-20220430212430066.png)

![image-20220430212513491](C:\Users\19651\AppData\Roaming\Typora\typora-user-images\image-20220430212513491.png)

![image-20220430212600087](C:\Users\19651\AppData\Roaming\Typora\typora-user-images\image-20220430212600087.png)

界面：

```html
<!DOCTYPE html>
<html lang="en" xmlns:th="http://www.thymeleaf.org">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
</head>
<body>
<div>
    <input th:placeholder="#{login.username}">
    <button>[[#{login.but}]]</button>
    <div>
        <a th:href="@{/index.html(l='zh_CN',zd='hello')}">中国</a>
        <a th:href="@{/index.html(l=en_US)}">美国</a>
    </div>
</div>
</body>
</html>
```

```java
package com.mywrok.config;

import org.hibernate.validator.spi.messageinterpolation.LocaleResolverContext;
import org.springframework.web.servlet.LocaleResolver;
import org.thymeleaf.util.StringUtils;

import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import java.util.Locale;

public class MyLocalResolver implements LocaleResolver {

//    解析请求
    @Override
    public Locale resolveLocale(HttpServletRequest request) {
        String l = request.getParameter("l");
        Locale aDefault = Locale.getDefault();//获取默认的地区配置login.properties
        if(!StringUtils.isEmpty(l)){
            String[] s = l.split("_");
            aDefault=new Locale(s[0],s[1]);
        }
        return aDefault;
    }

    @Override
    public void setLocale(HttpServletRequest request, HttpServletResponse response, Locale locale) {

    }
}
```

```java
package com.mywrok.config;


import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.web.servlet.LocaleResolver;
import org.springframework.web.servlet.config.annotation.ViewControllerRegistry;
import org.springframework.web.servlet.config.annotation.WebMvcConfigurer;

@Configuration
public class MyMvcConfig implements WebMvcConfigurer {
    @Override
    public void addViewControllers(ViewControllerRegistry registry) {
        registry.addViewController("/index.html").setViewName("index");
        registry.addViewController("/").setViewName("index");
    }

    @Bean
    public LocaleResolver localeResolver(){
        return new MyLocalResolver();
    }
}
```

拦截器

```java
package com.mywrok.config;

import org.springframework.web.servlet.HandlerInterceptor;
import org.springframework.web.servlet.ModelAndView;

import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;

public class MyInterceptor implements HandlerInterceptor {
    @Override
    public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler) throws Exception {
        Object username = request.getSession().getAttribute("username");
        if(username==null){
            request.getRequestDispatcher("/login.html").forward(request,response);
            return false;
        }
        return true;
    }

}
```

```java
package com.mywrok.config;


import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.web.servlet.LocaleResolver;
import org.springframework.web.servlet.config.annotation.InterceptorRegistry;
import org.springframework.web.servlet.config.annotation.ViewControllerRegistry;
import org.springframework.web.servlet.config.annotation.WebMvcConfigurer;

@Configuration
public class MyMvcConfig implements WebMvcConfigurer {
    @Override
    public void addInterceptors(InterceptorRegistry registry) {
        registry.addInterceptor(new MyInterceptor()).addPathPatterns("/**").excludePathPatterns("/login.html","/","/css/**","/js/**","/images/**");
    }
}
```

thymeleaf组件化使用

th:fragment="slideFragment"

使用组件:1<div th:insert="~{页面名::slideFragment}"></div>

2<div th:replace="~{commons/commons::slideFragment}"></div>

组件化传参

```html
<div th:insert="~{commons/commons::hell(color='green')}">

</div>
<div th:replace="~{commons/commons::hell(color='red')}">

</div>
```

```html
<!DOCTYPE html>
<html lang="en">

<h1 th:fragment="hell" th:style="${color=='red'}?'color:red':'color:green'">HELLO WORLD</h1>

</html>
```

日期格式问题

```yaml
spring:
	mvc:
  		date-format: yyyy-MM-dd
```

前端传过来的日期格式必须是yyyy-MM-dd格式,默认是(yyyy/MM/dd形式)



springboot整合mybatis

```xml
		<dependency>
            <groupId>mysql</groupId>
            <artifactId>mysql-connector-java</artifactId>
            <version>5.1.2</version>
            <scope>runtime</scope>
        </dependency>
        <dependency>
            <groupId>org.mybatis.spring.boot</groupId>
            <artifactId>mybatis-spring-boot-starter</artifactId>
            <version>1.3.2</version>
        </dependency>
        <dependency>
            <groupId>com.alibaba</groupId>
            <artifactId>druid</artifactId>
            <version>1.1.9</version>
        </dependency>
```

```xml
spring:
#页面国际化
  messages:
    basename: i18n.login
  datasource:
    username: root
    password: 123456
    driver-class-name: com.mysql.jdbc.Driver
    url: jdbc:mysql://localhost:3306/springboottest
    type: com.alibaba.druid.pool.DruidDataSource

#mybatis
mybatis:
  mapper-locations: classpath:mybatis/mapper/*.xml
  type-aliases-package: com.mywrok.pojo
```

在启动类Application加注解@MapperScan("com.mywrok.dao")

安全=》认证&授权

SpringSecurity，shiro针对spring项目于的安全框架

authorize：授权

authentication：认证

WebSecurityConfigurerAdapter：自定义security策略

AuthenticationManagerBuilder：自定义认证策略

@EnableWebSecurity：开启WebSecurity策略

SpringSecurity的两个主要目标是"认证(authentication)"和"授权(authorize)"(访问控制)

```java
package com.mywrok.config;


import org.springframework.security.authentication.AuthenticationTrustResolver;
import org.springframework.security.config.annotation.authentication.builders.AuthenticationManagerBuilder;
import org.springframework.security.config.annotation.web.builders.HttpSecurity;
import org.springframework.security.config.annotation.web.configuration.EnableWebSecurity;
import org.springframework.security.config.annotation.web.configuration.WebSecurityConfiguration;
import org.springframework.security.config.annotation.web.configuration.WebSecurityConfigurerAdapter;
import org.springframework.security.crypto.bcrypt.BCryptPasswordEncoder;
import org.springframework.security.crypto.password.PasswordEncoder;

@EnableWebSecurity
public class SecurityConfig extends WebSecurityConfigurerAdapter {
//    链式编程
    @Override
    protected void configure(HttpSecurity http) throws Exception {
        http.authorizeRequests()
                .antMatchers("/").permitAll()
                .antMatchers("/leve1/**").hasRole("vip1")
                .antMatchers("/leve2/**").hasRole("vip2")
                .antMatchers("/leve3/**").hasRole("vip3");
//        跳到登录界面
//        没有权限默认会跳到登录界面，需要开启登录的界面
        http.formLogin();

//        注销
        在thymeleaf中<a th:href="@{/logout}"></a>
        http.logout().logoutSuccessUrl("/");//注销成功界面
    }

//    认证
//PasswordEncoder是数据加密
    @Override
    protected void configure(AuthenticationManagerBuilder auth) throws Exception {
//        这些数据正常应该从数据库中读
        auth.inMemoryAuthentication().passwordEncoder(new BCryptPasswordEncoder())
                .withUser("root").password(new BCryptPasswordEncoder().encode("123456")).roles("vip1","vip2","vip3").and()
                .withUser("genter").password(new BCryptPasswordEncoder().encode("123456")).roles("vip1").and()
                .withUser("rule").password(new BCryptPasswordEncoder().encode("123456")).roles("vip2","vip3");
    }
}

```

访问数据库：

![](C:\Users\19651\AppData\Roaming\Typora\typora-user-images\image-20220501231420354.png)

springSecurity整合thymeleaf所用依赖

~~~xml
<!-- https://mvnrepository.com/artifact/org.thymeleaf.extras/thymeleaf-extras-springsecurity5 -->
        <dependency>
            <groupId>org.thymeleaf.extras</groupId>
            <artifactId>thymeleaf-extras-springsecurity5</artifactId>
            <version>3.0.4.RELEASE</version>
        </dependency>
~~~

thymeleaf头部加xmlns:sec="http://www.thymeleaf.org/thymeleaf-extras-springsecurity5"

```html
<!DOCTYPE html>
<html lang="en" xmlns:th="http://www.thymeleaf.org"
      xmlns:sec="http://www.thymeleaf.org/thymeleaf-extras-springsecurity5">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
</head>
<body>
<h1 style="color: green">leve1.html</h1>
<a th:href="@{/logout}">退出</a>使用springSecurity自带的logout界面
<!--如果未认证-->
<div sec:authorize="!isAuthenticated()">
<h1 style="color: coral">未认证消息</h1>
</div>
<!--如果认证-->
<div sec:authorize="isAuthenticated()">
    <h1 style="color: coral">认证消息</h1>
    <span sec:authentication="name"></span>
</div>
</body>
</html>
```

记住我及首页定制

```java
package com.mywrok.config;


import org.springframework.security.authentication.AuthenticationTrustResolver;
import org.springframework.security.config.annotation.authentication.builders.AuthenticationManagerBuilder;
import org.springframework.security.config.annotation.web.builders.HttpSecurity;
import org.springframework.security.config.annotation.web.configuration.EnableWebSecurity;
import org.springframework.security.config.annotation.web.configuration.WebSecurityConfiguration;
import org.springframework.security.config.annotation.web.configuration.WebSecurityConfigurerAdapter;
import org.springframework.security.crypto.bcrypt.BCryptPasswordEncoder;
import org.springframework.security.crypto.password.PasswordEncoder;

@EnableWebSecurity
public class SecurityConfig extends WebSecurityConfigurerAdapter {
//    链式编程
    @Override
    protected void configure(HttpSecurity http) throws Exception {
        http.authorizeRequests()
                .antMatchers("/").permitAll()
                .antMatchers("/leve1/**").hasRole("vip1")
                .antMatchers("/leve2/**").hasRole("vip2")
                .antMatchers("/leve3/**").hasRole("vip3");
//        跳到登录界面
//        没有权限默认会跳到登录界面，需要开启登录的界面
        http.formLogin().usernameParameter("username").passwordParameter("password").loginPage("/userLogin");
        或者
            //http.formLogin().usernameParameter("username").passwordParameter("password").loginPage("/userLogin").loginProcessingUrl("/login");调用默认的login此时userLogin.HTML中action中写的是@{/login}

//        注销
        http.csrf().disable();//关闭跨站域请求伪造,不关闭csrf可能注销不了
        http.logout().logoutSuccessUrl("/");//注销成功界面

//        记住我
        http.rememberMe().rememberMeParameter("remember");
    }

//    认证
//PasswordEncoder是数据加密
    @Override
    protected void configure(AuthenticationManagerBuilder auth) throws Exception {
//        这些数据正常应该从数据库中读
        auth.inMemoryAuthentication().passwordEncoder(new BCryptPasswordEncoder())
                .withUser("root").password(new BCryptPasswordEncoder().encode("123456")).roles("vip1","vip2","vip3").and()
                .withUser("genter").password(new BCryptPasswordEncoder().encode("123456")).roles("vip1").and()
                .withUser("rule").password(new BCryptPasswordEncoder().encode("123456")).roles("vip2","vip3");
    }
}
```

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
</head>
<body>
<form th:action="@{/userLogin}" method="post">
    <div>
        <span>用户名</span>
        <input type="text" name="username"/>
    </div>
    <div>
        <span>密码</span>
        <input type="password" name="password">
    </div>
    <div>
        <input type="checkbox" name="remember">
    </div>
    <input type="submit" value="提交">
</form>
</body>
</html>
```

Shiro

Apache Shiro是一个Java的安全(权限)框架

Shiro可以完成，认证，授权，加密，会话管理

快速入门：

![在这里插入图片描述](https://img-blog.csdnimg.cn/20201202215335331.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2JieHlscWYxMjZjb20=,size_16,color_FFFFFF,t_70#)

##### Subject

`Subject`即主体，外部应用与subject进行交互，subject记录了当前的操作用户，将用户的概念理解为当前操作的主体。外部程序通过subject进行认证授权，而subject是通过SecurityManager安全管理器进行认证授权

##### **SecurityManager**

SecurityManager即安全管理器，对全部的subject进行安全管理，它是shiro的核心，负责对所有的subject进行安全管理。通过SecurityManager可以完成subject的认证、授权等，实质上SecurityManager是通过Authenticator进行认证，通过Authorizer进行授权，通过SessionManager进行会话管理等

SecurityManager是一个接口，继承了Authenticator, Authorizer, SessionManager这三个接口

##### Authenticator

Authenticator即认证器，对用户身份进行认证，Authenticator是一个接口，shiro提供ModularRealmAuthenticator实现类，通过ModularRealmAuthenticator基本上可以满足大多数需求，也可以自定义认证器

##### Authorizer

`Authorizer`即授权器，用户通过认证器认证通过，在访问功能时需要通过授权器判断用户是否有此功能的操作权限

##### Realm

Realm即领域，相当于datasource数据源，securityManager进行安全认证需要通过Realm获取用户权限数据，比如：如果用户身份数据在数据库那么realm就需要从数据库获取用户身份信息

不要把realm理解成只是从数据源取数据，在realm中还有认证授权校验的相关的代码

##### SessionManager

`sessionManager`即会话管理，shiro框架定义了一套会话管理，它不依赖web容器的[session](https://so.csdn.net/so/search?q=session&spm=1001.2101.3001.7020)，所以shiro可以使用在非web应用上，也可以将分布式应用的会话集中在一点管理，此特性可使它实现单点登录

##### SessionDAO

SessionDAO即会话dao，是对session会话操作的一套接口，比如要将session存储到数据库，可以通过jdbc将会话存储到数据库

##### CacheManager

CacheManager即缓存管理，将用户权限数据存储在缓存，这样可以提高性能

##### Cryptography

Cryptography即密码管理，shiro提供了一套加密/解密的组件，方便开发。比如提供常用的散列、加/解密等功能。

#### shiro认证

##### 什么是认证

身份认证，就是判断一个用户是否为合法用户的处理过程。最常用的简单身份认证方式是系统通过核对用户输入的用户名和口令，看其是否与系统中存储的该用户的用户名和口令一致，来判断用户身份是否正确

##### Subject

访问系统的用户，主体可以是用户、程序等，进行认证的都称为主体

##### Principal

身份信息，是主体（subject）进行身份认证的标识，标识必须具有唯一性，如用户名、手机号、邮箱地址等，一个主体可以有多个身份，但是必须有一个主身份（Primary Principal）

##### credential

凭证信息，是只有主体自己知道的安全信息，如密码、证书等

~~~java
import org.apache.shiro.authc.AuthenticationException;
import org.apache.shiro.authc.AuthenticationInfo;
import org.apache.shiro.authc.AuthenticationToken;
import org.apache.shiro.authz.AuthorizationInfo;
import org.apache.shiro.realm.AuthorizingRealm;
import org.apache.shiro.subject.PrincipalCollection;

/**
 * 自定义realm
 */
public class CustomerRealm extends AuthorizingRealm {
    // 授权
    @Override
    protected AuthorizationInfo doGetAuthorizationInfo(PrincipalCollection principals) {
        return null;
    }

    // 认证
    @Override
    protected AuthenticationInfo doGetAuthenticationInfo(AuthenticationToken token) throws AuthenticationException {
        return null;
    }
}

~~~

~~~java
import com.christy.shiro.realm.CustomerRealm;
import org.apache.shiro.realm.Realm;
import org.apache.shiro.spring.web.ShiroFilterFactoryBean;
import org.apache.shiro.web.mgt.DefaultWebSecurityManager;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

import java.util.HashMap;
import java.util.Map;

/**
 * Shiro的核心配置类，用来整合shiro框架
 */
@Configuration
public class ShiroConfiguration {

    //1.创建shiroFilter  //负责拦截所有请求
    @Bean
    public ShiroFilterFactoryBean getShiroFilterFactoryBean(DefaultWebSecurityManager defaultWebSecurityManager){
        ShiroFilterFactoryBean shiroFilterFactoryBean = new ShiroFilterFactoryBean();

        //给filter设置安全管理器
        shiroFilterFactoryBean.setSecurityManager(defaultWebSecurityManager);

        //配置系统受限资源
        //配置系统公共资源
        Map<String,String> map = new HashMap<String,String>();
        map.put("/index.jsp","authc");//authc 请求这个资源需要认证和授权

        //默认认证界面路径
        shiroFilterFactoryBean.setLoginUrl("/login.jsp");
        shiroFilterFactoryBean.setFilterChainDefinitionMap(map);

        return shiroFilterFactoryBean;
    }

    //2.创建安全管理器
    @Bean
    public DefaultWebSecurityManager getDefaultWebSecurityManager(Realm realm){
        DefaultWebSecurityManager defaultWebSecurityManager = new DefaultWebSecurityManager();
        //给安全管理器设置
        defaultWebSecurityManager.setRealm(realm);

        return defaultWebSecurityManager;
    }

    //3.创建自定义realm
    @Bean
    public Realm getRealm(){
        CustomerRealm customerRealm = new CustomerRealm();
        return customerRealm;
    }
}
~~~



##### swagger

最流行的Api框架

Restful Api文档在线自动生成工具=》Api文档与API定义同步更新

直接运行，可以在线测试API接口

支持多种语言：(java，php...)

~~~java
Failed to start bean &#39;documentationPluginsBootstrapper&#39;; nested exception is java.lang.NullPointerException----springboot版本太高c
~~~

springboot使用2.5.6版本整合swagger3.0

```xml
<parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
<!--        <version>2.6.7</version>-->
        <version>2.5.6</version>
        <relativePath/> <!-- lookup parent from repository -->
    </parent>
```

swagge依赖

```xml
<dependency>
    <groupId>io.springfox</groupId>
    <artifactId>springfox-boot-starter</artifactId>
    <version>3.0.0</version>
</dependency>
```

```java
package com.mywrok.config;

import org.springframework.context.annotation.Configuration;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.ResponseBody;
import springfox.documentation.oas.annotations.EnableOpenApi;
import springfox.documentation.swagger2.annotations.EnableSwagger2;

@Configuration
@EnableSwagger2
public class SwaggerTest {

  @Bean
   public Docket docket(Environment environment){
       Profiles dev = Profiles.of("dev");
//       监听环境是否是测试环境,如果是返回true 可以返回接口
       boolean boo = environment.acceptsProfiles(dev);
       return new Docket(DocumentationType.SWAGGER_2).groupName("d阿辉")
                .apiInfo(apiInfo()).enable(boo).select()
//       RequestHandlerSelectors配置要扫描接口的方式
//       basePackage指定要扫描接口的包
//                any()全部
//                none不扫描
//       withClassAnnotation扫描类上的注解->参数是类的反射
//       withMethodAnnotation扫描方法上的注解
                .apis(RequestHandlerSelectors.basePackage("com.mywrok.controller"))
                .build();
   }
   public ApiInfo apiInfo(){
       Contact contact = new Contact("阿辉", "", "1965165252@qq.com");
       return new ApiInfo("阿辉的swagger",
               "即使再小的帆也能远航",
               "1.0v",
               "http://baidu.com",
               contact, "Apache 2.0",
               "http://www.apache.org/licenses/LICENSE-2.0",
               new ArrayList());
   }

   @Bean
   public Docket docket1(){
       return new Docket(DocumentationType.SWAGGER_2).groupName("小红").select().apis(RequestHandlerSelectors.none()).build();
   }
}
```

访问接口

~~~http
http://localhost:8080/{base-path}/swagger-ui/#/
~~~

```java
//给生成的swagger的model加注释
@Data
@ApiModel
public class Admin {
    @ApiModelProperty("id")
    private Integer id;
    @ApiModelProperty("用户名")
    private String username;
    @ApiModelProperty("密码")
    private String password;
}
```

异步任务（之前都是等待方法完成在执行下一步，现在可以跳过等待时间直接执行下一步）

在主运行类上开启异步@EnableAsync

在service上加异步注解@Async=》表示这个一个异步操作可以先执行下面的

邮件任务

加入依赖

```xml
<!--        邮件任务-->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-mail</artifactId>
        </dependency>
```

```yaml
spring:
 mail:
  username: 1965165252@qq.com
  password: 通过qq邮件获取设置-》
  host: smtp.qq.com
  properties:
    mail:
      smtp:
        ssl:
          enable: true
```

```java
package com.mywrok;

import org.junit.jupiter.api.Test;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.mail.SimpleMailMessage;
import org.springframework.mail.javamail.JavaMailSenderImpl;

import javax.sql.DataSource;
import java.sql.Connection;


@SpringBootTest

class SpringbootLearnApplicationTests {
    @Autowired
    private JavaMailSenderImpl javaMailSender;

    @Test
    public void sendMail(){
        //简单邮件发送
        SimpleMailMessage simpleMailMessage=new SimpleMailMessage();
        simpleMailMessage.setSubject("hello 阿辉");//主题
        simpleMailMessage.setText("许久未见");//内容
        simpleMailMessage.setFrom("1965165252@qq.com");//自己的qq邮箱
        simpleMailMessage.setTo("1965165252@qq.com");//发送地址
        javaMailSender.send(simpleMailMessage);
    }
    @Test
    public void sendMain2() throws MessagingException {
        //复杂邮件发送
        MimeMessage mimeMessage = javaMailSender.createMimeMessage();
        MimeMessageHelper mimeMessageHelper = new MimeMessageHelper(mimeMessage,true);

        mimeMessageHelper.setSubject("hello 阿辉");
        mimeMessageHelper.setText("<p style='color:red'>hello world</p>",true);

        mimeMessageHelper.addAttachment("1.jpg",new File("A:\\壁纸\\img\\cover12.4.jpg"));

        mimeMessageHelper.setFrom("1965165252@qq.com");
        mimeMessageHelper.setTo("1965165252@qq.com");
        javaMailSender.send(mimeMessage);
    }
}
```

定时任务

cron表达式

cron表达式，六个值 （秒，分，时，日，月，星期)

- \*： 表示匹配该域的任意值。比如Minutes域使用*，就表示每分钟都会触发。

- -：表示范围。比如Minutes域使用 10-20，就表示从10分钟到20分钟每分钟都会触发一次。
- ,：表示列出枚举值。比如Minutes域使用1,3，就表示1分钟和3分钟都会触发一次。
- / : 表示间隔时间触发(开始时间/时间间隔)。例如在Minutes域使用 5/10，就表示从第5分钟开始，每隔10分钟触发一次。
- ? : 表示不指定值。简单理解就是忽略该字段的值，直接根据另一个字段的值触发执行。
- 表示该月第n个星期x(x#n)，仅用星期域。如：星期：6#3，表示该月的第三个星期五。
- L : 表示最后，是单词"last"的缩写（最后一天或最后一个星期几）；仅出现在日和星期的域中。用在日则表示该月的最后一天，用在星期则表示该月的最后一个星期。如：星期域上的值为5L，则表示该月最后一个星期的星期四。在使用'L'时，不要指定列表','或范围'-'，否则易导致出现意料之外的结果。
- W: 仅用在日的域中，表示距离当月给定日期最近的工作日（周一到周五），是单词"weekday"的缩写。

```java
@EnableScheduling//在运行类开启定时功能注解
@Scheduled//什么时候执行,在service类中使用
TaskScheduler//任务调度者
TaskExecutor//任务执行者
```

远程服务调用(RPC)

rpc两个核心模块:通讯，序列化(数据传输需要转换)

什么时dubbo？

dubbo是一款高性能，轻量级的开源java rpc框架，它提供了三大核心能力，面向接口的远程方法调用，智能容错和负载均衡，以及服务自动注册和发现

zookeeper相当于Registry

**dubbo暴露端口之后向Registry注册，消费者向Registry请求，有的话返回接口地址，消费者通过地址向提供者请求端口**

![image-20220503140425666](C:\Users\19651\AppData\Roaming\Typora\typora-user-images\image-20220503140425666.png)

安装zookeeper

[Apache Downloads](https://www.apache.org/dyn/closer.lua/zookeeper/zookeeper-3.8.0/apache-zookeeper-3.8.0-bin.tar.gz)

下载后解压，将config目录下的zoo_sample.cfg 重命名为 zoo.cfg (Zookeeper配置文件，默认端口为2181，可根据实际进行修改)。然后双击bin目录下的zkServer.cmd启动即可。

管理员身份运行zkServer.sh报错(可在内部加pause，暂停查看报错信息),也有可能是8080端口冲突

dubbo下载地址：https://github.com/dangdangdotcom/dubbox

~~~cmd
G:\dubbo\dubbox-master>mvn clean package -Dmaven.test.skip=true
~~~
