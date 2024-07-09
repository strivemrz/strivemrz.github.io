---
title: springMvc
categories: spring
tags:
  - spring
excerpt: ''
cover: https://desk-fd.zol-img.com.cn/t_s960x600c5/g5/M00/02/03/ChMkJlbKxtqICzWtABTs9EQHkIUAALHrQLAW5UAFO0M151.jpg
---

maven项目创建过慢

1. 添加键值对 archetypeCatalog internal
2. 设置阿里镜像

springMvc中Dispatcher作用

​	*org.springframework.web.servlet.DispatcherServlet*

![image-20220109170751482](C:\Users\19651\AppData\Roaming\Typora\typora-user-images\image-20220109170751482.png)

```xml
<!DOCTYPE web-app PUBLIC
 "-//Sun Microsystems, Inc.//DTD Web Application 2.3//EN"
 "http://java.sun.com/dtd/web-app_2_3.dtd" >

<web-app>
  <display-name>Archetype Created Web Application</display-name>
  <servlet>
    <servlet-name>DispatcherServlet</servlet-name>
    <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
    <init-param>
      <param-name>contextConfigLocation</param-name>
      <param-value>classpath:springMvc.xml</param-value>
    </init-param>
    <load-on-startup>1</load-on-startup>
  </servlet>
  <servlet-mapping>
    <servlet-name>DispatcherServlet</servlet-name>
    <url-pattern>/</url-pattern>
  </servlet-mapping>
</web-app>
```

DispatcherServlet：前端控制器

​									用户请求到达前端控制器，它就相当于mvc模式中的c，dispatcherServlet是整个流程控制的中心，由其他调用其他组件处理用户的请求，dispatcherServlet的存在降低了组件之间的耦合性。

HandlerMapping：处理器映射器

​									HandlerMapping负责根据用户请求找到Handler即处理器，SpringMvc提供了不同的映射器实现不同的映射方式，例如：配置文件方式，实现接口方式，注解方式等。

HandlerAdapter：处理器适配器

​									通过HandlerAdapter对处理器进行执行，这是适配器模式的应用，通过扩展适配器可以对更多类型的处理器进行执行。

View：视图

​									SpringMVC框架提供了更多的View视图类型的支持，包括：jstlView，freemarkerView，pdfView等，我们最常用的视图就是jsp。

 <[mvc](https://so.csdn.net/so/search?q=mvc):annotation-driven />是一种简写形式，完全可以手动配置替代这种简写形式，简写形式可以让初学者快速应用默认配置方案

springMvc注解

@RequestMapping("/springmvc")：映射路径

params：表示传递过来的必须有username，id这个数据否则不会匹配

headers：要求必须含有对应的请求头才能匹配

method：以什么方式请求才能匹配到

```Java
@RequestMapping(value = "/springmvc",params = {"username","id"},headers = {"Accept"},method = {RequestMethod.POST})
    public String springMvcF(){
        return "hello";
    }
```

集合类型参数

```JSP
<form action="springmvc" method="post">
    用户名：<input type="text" name="username"><br>
    密码：<input type="text" name="password"><br>
    list:<input type="test" name="adminList[0].username"><br>
    list:<input type="test" name="adminList[0].password"><br>

    list:<input type="test" name="adminMap['map'].username"><br>
    list:<input type="test" name="adminMap['map'].password"><br>
    <input type="submit" value="提交">
</form>
```

springmvc拦截静态资源问题

由于在web.xml中配置了全局过滤，所以在引用js文件时同样会被拦截，只需在springMvc.xml中配置调用该文件目录时正常调用即可

```xml
<mvc:resources location="/js/" mapping="/js/**"></mvc:resources>
```

前端向后端传入json数据后端以对象形式接收

```xml
<dependency>
  <groupId>com.fasterxml.jackson.core</groupId>
  <artifactId>jackson-core</artifactId>
  <version>2.9.2</version>
</dependency>
<dependency>
  <groupId>com.fasterxml.jackson.core</groupId>
  <artifactId>jackson-databind</artifactId>
  <version>2.9.2</version>
</dependency>
```

```js
$("#in1").click(function (){
    alert("点击效果")
    $.ajax({
        url:"springmvcAjaxs",
        type:"post",
        contentType:"application/json;charset=utf-8",
        dataType:"json",//返回数据类型
        data:'{"username":"111","password":"张三"}',
        success:function (e){
            alert(e.username+":正确")
        },error:function (e){
            alert(e.password+":错误")
        }

    })
})
```

```java
@RequestMapping("/springmvcAjaxs")
public @ResponseBody UserInfo springmvcAja(@RequestBody UserInfo userInfo){
    System.out.println("后台执行了");

    System.out.println(userInfo.toString());
    return userInfo;
}
```



```java
request.getSession().getServletContext().getRealPath("/uploads/")
```

request.get[Session](https://so.csdn.net/so/search?q=Session)().getServletContext() 获取的是Servlet容器对象，相当于tomcat容器了。getRealPath("/") 获取实际路径，“/”指代项目根目录，所以代码返回的是项目在容器中的实际发布运行的根路径

springMvc上传文件

```xml
<dependency>
  <groupId>commons-io</groupId>
  <artifactId>commons-io</artifactId>
  <version>2.5</version>
</dependency>
<dependency>
  <groupId>commons-fileupload</groupId>
  <artifactId>commons-fileupload</artifactId>
  <version>1.3.1</version>
</dependency>
<dependency>
  <groupId>org.apache.commons</groupId>
  <artifactId>commons-lang3</artifactId>获取随机数
  <version>3.1</version>
</dependency>
```

```jsp
<form action="FileUp" method="post" enctype="multipart/form-data">
    <input type="file" value="选择文件" name="upload">
    <input type="submit" value="提交">
</form>
```

```java
@RequestMapping("/FileUp")
    public String FileUp(HttpServletRequest request, MultipartFile upload) throws IOException {
        String realPath = request.getSession().getServletContext().getRealPath(File.separator+"uploads"+File.separator);
        //        获取原始文件名称
        String orignl=upload.getOriginalFilename();
//        获取图片后缀
        String extension = FilenameUtils.getExtension(orignl);
        File file=new File(realPath);
        if(!file.exists()){
//            如果文件不存在则创建该目录
            file.mkdir();
        }
        String fileName=System.currentTimeMillis()+Math.ceil(Math.random()*100000) +"_pic."+extension;
        upload.transferTo(new File(realPath,fileName));
        return "success";
    }
```

```xml
    //使用配置好的解析器  注意这里id不能改
    <bean id="multipartResolver" class="org.springframework.web.multipart.commons.CommonsMultipartResolver">
<!--        上传最大文件大小10M 这里字节表示-->
        <property name="maxUploadSize" value="10485760"></property>
    </bean>
```

自定义异常处理器(报异常直接显示错误界面)

```java
import org.springframework.web.servlet.ModelAndView;

import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import java.lang.reflect.InvocationHandler;
import java.lang.reflect.Method;

public class SysExcep implements HandlerExceptionResolver {

    @Override
    public ModelAndView resolveException(HttpServletRequest httpServletRequest, HttpServletResponse httpServletResponse, Object o, Exception e) {
        ModelAndView modelAndView=new ModelAndView();
        if(e instanceof MyException){
            MyException myException=(MyException)e;
            modelAndView.addObject("errorMsg",myException.getErrorMsg());
        }else{
            modelAndView.addObject("errorMsg",e);
        }
        modelAndView.setViewName("error");
        return modelAndView;
    }
}
```

```xml
<bean id="sysExcep" class="com.mySpringMvc.SysExcep"></bean>
```

```java
package com.mySpringMvc;

public class MyException extends Exception {
    private String errorMsg;

    public String getErrorMsg() {
        return errorMsg;
    }

    public void setErrorMsg(String errorMsg) {
        this.errorMsg = errorMsg;
    }

    public MyException(String message) {
        this.errorMsg = message;
    }

    @Override
    public String toString() {
        return "MyException{" +
                "errorMsg='" + errorMsg + '\'' +
                '}';
    }
}
```

```java
@RequestMapping("/redir")
    public String redir() throws MyException {
        try {
            int i=1/0;
        }catch (Exception e){
            e.printStackTrace();
            throw new MyException("除0异常");
        }

        return "hello";
    }
```

springMvc拦截器使用

![image-20220113172216967](C:\Users\19651\AppData\Roaming\Typora\typora-user-images\image-20220113172216967.png)

```java
package com.mySpringMvc;

import org.springframework.web.servlet.HandlerInterceptor;
import org.springframework.web.servlet.ModelAndView;

import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;

public class HandlerInt implements HandlerInterceptor {
    @Override
    public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler) throws Exception {
        System.out.println("拦截器---前");
        return true;//true 放行 ，false不放行
    }

    @Override
    public void postHandle(HttpServletRequest request, HttpServletResponse response, Object handler, ModelAndView modelAndView) throws Exception {
        System.out.println("拦截器---后");
    }

    @Override
    public void afterCompletion(HttpServletRequest request, HttpServletResponse response, Object handler, Exception ex) throws Exception {
        System.out.println("拦截器---最终");
    }
}
```

```xml
<!--    配置springMvc拦截器-->
    <mvc:interceptors>
        <mvc:interceptor>
<!--            要拦截的路径-->
            <mvc:mapping path="/Inter/*"/>
<!--            <mvc:mapping path="**"/>表示所有都会拦截-->
<!--            配置的拦截类-->
            <bean class="com.mySpringMvc.HandlerInt"></bean>
        </mvc:interceptor>
    </mvc:interceptors>
```

序列化原因

web服务器通常将那些暂时不活动但未超时的httpsession对象转移到文件系统或数据库中保存，需要序列化

序列化后对象才能通过流进行数据传输
