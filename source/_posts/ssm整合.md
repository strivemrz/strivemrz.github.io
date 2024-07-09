---
title: SSM整合
categories: ssm整合
tags:
  - ssm整合
excerpt: ''
cover: https://tse3-mm.cn.bing.net/th/id/OIP-C.Tl2kY94KgT6Bj5zc3R1zMQHaJQ?pid=ImgDet&rs=1
---

ssm整合

*目录结构*

![image-20220115101619458](C:\Users\19651\AppData\Roaming\Typora\typora-user-images\image-20220115101619458.png)

实现pojo实体类

~~~java
package com.mySSM.pojo;

import java.io.Serializable;

public class Admin implements Serializable {
    private Integer id;
    private String username;
    private String password;
    private Double money;
    public Integer getId() {
        return id;
    }
    public void setId(Integer id) {
        this.id = id;
    }
    public String getUsername() {
        return username;
    }
    public void setUsername(String username) {
        this.username = username;
    }
    public String getPassword() {
        return password;
    }
    public void setPassword(String password) {
        this.password = password;
    }
    public Double getMoney() {
        return money;
    }
    public void setMoney(Double money) {
        this.money = money;
    }
    @Override
    public String toString() {
        return "Admin{" +
                "id=" + id +
                ", username='" + username + '\'' +
                ", password='" + password + '\'' +
                ", money=" + money +
                '}';
    }
}

~~~

dao层

dao接口

~~~java
package com.mySSM.dao;

import com.mySSM.pojo.Admin;
import org.apache.ibatis.annotations.Param;
import org.springframework.stereotype.Repository;
import org.springframework.transaction.annotation.Transactional;

import java.util.List;
@Repository
public interface AdminDao {
    /**
     * 查询所有信息
     * @return
     */
    public List<Admin> findAll();

    /**
     * 减钱数
     * @param username
     * @param money
     * @return
     */
    public int desMoney(@Param("username") String username, @Param("money") Double money);

    /**
     * 增加钱数
     * @param username
     * @param money
     * @return
     */
    public int addMoney(@Param("username") String username,@Param("money") Double money);
}

~~~

dao的xml

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN" "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="com.mySSM.dao.AdminDao">
    <select id="findAll" resultType="admin">
        select * from ssmtest.admin
    </select>
    <update id="desMoney" >
        update ssmtest.admin set money=money-#{money} where username=#{username}
    </update>
    <update id="addMoney" >
        update ssmtest.admin set money=money+#{money} where username=#{username}
    </update>
</mapper>
```

servlet层

```java
package com.mySSM.service;

import com.mySSM.pojo.Admin;

import java.util.List;

public interface AdminService {
    public List<Admin> findAll();
    public int updMoney(String username,Double money,String username2,Double money2);
}
```

实现接口

```java
package com.mySSM.service.impl;

import com.mySSM.dao.AdminDao;
import com.mySSM.pojo.Admin;
import com.mySSM.service.AdminService;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Service;
import org.springframework.transaction.annotation.Transactional;

import javax.servlet.http.HttpServletRequest;
import java.util.List;

@Service
@Transactional
public class AdminServiceImpl implements AdminService {
    @Autowired
    private AdminDao adminDao;
    @Override
    public List<Admin> findAll() {
        System.out.println("执行了查询所有信息语句......");
        List<Admin> all = adminDao.findAll();
        return all;
    }

    @Override
    public int updMoney(String username, Double money, String username2, Double money2) {
        int i=adminDao.addMoney(username,money);
        int p=1/0;
        int j=adminDao.desMoney(username2,money2);
        if(i>0&&j>0){
            return 1;
        }
        return 0;
    }


}
```

controller层

```java
package com.mySSM.controller;

import com.mySSM.pojo.Admin;
import com.mySSM.service.AdminService;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.beans.factory.annotation.Qualifier;
import org.springframework.stereotype.Controller;
import org.springframework.test.annotation.Rollback;
import org.springframework.transaction.annotation.Transactional;
import org.springframework.web.bind.annotation.RequestMapping;

import javax.servlet.http.HttpServletRequest;
import java.util.List;

@Controller
public class Controllers {
    @Autowired
    @Qualifier("adminServiceImpl")
    private AdminService adminService;

    public AdminService getAdminService() {
        return adminService;
    }

    public void setAdminService(AdminService adminService) {
        this.adminService = adminService;
    }

    @RequestMapping("/findAll")
    public String findAll(HttpServletRequest request,String username){
        List<Admin> all = adminService.findAll();
        request.setAttribute("userInfos",all);
        return "success";
    }
    @RequestMapping("/changeMoney")
    public String changeMoney(String username,Double money,String username2,Double money2){
        int i=adminService.updMoney(username,money,username2,money2);
        return "success";
    }
}
```

spring.xml配置

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context" xmlns:tx="http://www.springframework.org/schema/tx"
       xmlns:aop="http://www.springframework.org/schema/aop"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
       http://www.springframework.org/schema/beans/spring-beans.xsd http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context.xsd http://www.springframework.org/schema/tx http://www.springframework.org/schema/tx/spring-tx.xsd http://www.springframework.org/schema/aop http://www.springframework.org/schema/aop/spring-aop.xsd">

    <context:component-scan base-package="com.mySSM">
        <context:exclude-filter type="annotation" expression="org.springframework.web.bind.annotation.ControllerAdvice"/>
        <context:exclude-filter type="annotation" expression="org.springframework.stereotype.Controller"/>
    </context:component-scan>
    <bean id="dataSource" class="org.springframework.jdbc.datasource.DriverManagerDataSource">
        <property name="driverClassName" value="com.mysql.jdbc.Driver"></property>
        <property name="url" value="jdbc:mysql://localhost:3306/ssmtest"></property>
        <property name="username" value="root"></property>
        <property name="password" value="123456"></property>
    </bean>
    <bean id="sqlSessionFactoryBean" class="org.mybatis.spring.SqlSessionFactoryBean">
        <property name="dataSource" ref="dataSource"></property>
        <property name="typeAliasesPackage" value="com.mySSM.pojo"></property>
    </bean>
    <bean class="org.mybatis.spring.mapper.MapperScannerConfigurer" id="mapperScannerConfigurer">
        <property name="basePackage" value="com.mySSM.dao"></property>
    </bean>
<!--    配置事务管理-->
    <bean id="dataSourceTransactionManager" class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
        <property name="dataSource" ref="dataSource"></property>
    </bean>
    <tx:advice id="txManager" transaction-manager="dataSourceTransactionManager">
        <tx:attributes>
            <tx:method name="find*" read-only="true" propagation="SUPPORTS"/>
            <tx:method name="*" read-only="false" propagation="REQUIRED"></tx:method>
        </tx:attributes>
    </tx:advice>
    <aop:config>
        <aop:pointcut id="pot1" expression="execution(* com.mySSM.service.*..*(..))"/>
        <aop:advisor advice-ref="txManager" pointcut-ref="pot1"></aop:advisor>
    </aop:config>
<!--    <tx:annotation-driven transaction-manager="dataSourceTransactionManager"/>-->
</beans>
```

springmvc层配置

```xml
<?xml version = "1.0" encoding = "utf-8"?>
<beans xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns="http://www.springframework.org/schema/beans" xmlns:context="http://www.springframework.org/schema/context"
       xmlns:tx="http://www.springframework.org/schema/tx" xmlns:aop="http://www.springframework.org/schema/aop"
       xmlns:mvc="http://www.springframework.org/schema/mvc"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
       http://www.springframework.org/schema/beans/spring-beans.xsd
       http://www.springframework.org/schema/context
       http://www.springframework.org/schema/context/spring-context.xsd
       http://www.springframework.org/schema/tx
       http://www.springframework.org/schema/tx/spring-tx.xsd
       http://www.springframework.org/schema/aop
       http://www.springframework.org/schema/aop/spring-aop.xsd
       http://www.springframework.org/schema/mvc
       http://www.springframework.org/schema/mvc/spring-mvc.xsd ">

    <context:component-scan base-package="com.mySSM.controller" use-default-filters="false">
        <context:include-filter type="annotation" expression="org.springframework.stereotype.Controller"/>
        <context:exclude-filter type="annotation" expression="org.springframework.stereotype.Service"/>
    </context:component-scan>
    <bean class="org.springframework.web.servlet.view.InternalResourceViewResolver" id="internalResourceViewResolver">
        <property name="prefix" value="/WEB-INF/pages/"></property>
        <property name="suffix" value=".jsp"></property>
    </bean>
    <mvc:annotation-driven />
</beans>
```

jsp界面

```jsp
<%--
  Created by IntelliJ IDEA.
  User: 19651
  Date: 2022/1/13
  Time: 19:46
  To change this template use File | Settings | File Templates.
--%>
<%@ page contentType="text/html;charset=UTF-8" language="java" %>
<html>
<head>
    <title>Title</title>
</head>
<body>
<a href="findAll?username=aaa">查询</a>
<a href="changeMoney?username=aaa&money=10&username2=bbb&money2=10">修改钱数</a>
</body>
</html>
```

web.xml配置

```xml
<!DOCTYPE web-app PUBLIC
 "-//Sun Microsystems, Inc.//DTD Web Application 2.3//EN"
 "http://java.sun.com/dtd/web-app_2_3.dtd" >

<web-app>
  <display-name>Archetype Created Web Application</display-name>
  <context-param>
    <param-name>contextConfigLocation</param-name>
    <param-value>classpath:applicationContext.xml</param-value>
  </context-param>
  <listener>
    <listener-class>org.springframework.web.context.ContextLoaderListener</listener-class>
  </listener>

  <filter>
    <filter-name>characterEncodingFilter</filter-name>
    <filter-class>org.springframework.web.filter.CharacterEncodingFilter</filter-class>
    <init-param>
      <param-name>encoding</param-name>
      <param-value>utf-8</param-value>
    </init-param>
  </filter>
  <filter-mapping>
    <filter-name>characterEncodingFilter</filter-name>
    <url-pattern>/*</url-pattern>
  </filter-mapping>
  <servlet>
    <servlet-name>dispatcherServlet</servlet-name>
    <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
    <init-param>
      <param-name>contextConfigLocation</param-name>
      <param-value>classpath:springMvc.xml</param-value>
    </init-param>
    <load-on-startup>1</load-on-startup>
  </servlet>
  <servlet-mapping>
    <servlet-name>dispatcherServlet</servlet-name>
    <url-pattern>/</url-pattern>
  </servlet-mapping>
</web-app>
```

事务管理(在service实现类上边标注)

xml形式

*在spring配置文件中加上*

```xml
<!--    配置事务管理-->
    <bean id="dataSourceTransactionManager" class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
        <property name="dataSource" ref="dataSource"></property>
    </bean>
<!--配置事务管理的通知-->
    <tx:advice id="txManager" transaction-manager="dataSourceTransactionManager">
        <tx:attributes>
            <tx:method name="find*" read-only="true" propagation="SUPPORTS"/>
            <tx:method name="*" read-only="false" propagation="REQUIRED"></tx:method>
        </tx:attributes>
    </tx:advice>
    <aop:config>
        <aop:pointcut id="pot1" expression="execution(* com.mySSM.service.*..*(..))"/>
        <aop:advisor advice-ref="txManager" pointcut-ref="pot1"></aop:advisor>
    </aop:config>
```

注解形式(在service实现类上面注解@Transactional)

```xml
<!--    配置事务管理-->
    <bean id="dataSourceTransactionManager" class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
        <property name="dataSource" ref="dataSource"></property>
    </bean>
<tx:annotation-driven transaction-manager="dataSourceTransactionManager"/>
```



分页功能实现

导入依赖

```xml
<!--    分页功能插件-->
    <dependency>
      <groupId>com.github.pagehelper</groupId>
      <artifactId>pageHelper</artifactId>
      <version>5.1.10</version>
    </dependency>
```

在spring配置文件中设置插件

```xml
<bean id="sqlSessionFactoryBean" class="org.mybatis.spring.SqlSessionFactoryBean">
    <property name="dataSource" ref="dataSource"></property>
    <property name="typeAliasesPackage" value="com.mySSM.pojo"></property>
    <property name="plugins">
        <array>
            <bean class="com.github.pagehelper.PageInterceptor"></bean>
        </array>
    </property>
</bean>
```

在service层添加功能

```java
@Override
public PageInfo<Admin> findAlls(Integer start, Integer counts) {
    PageHelper.startPage(start,counts);
    List<Admin> all = adminDao.findAll();
    PageInfo<Admin> adminPageInfo=new PageInfo<>(all);
    return adminPageInfo;
}
```

controller添加

```java
@RequestMapping("/findAllAdmins")
    public String findAllAdmins(HttpServletRequest request, HttpServletResponse response, Integer start, Integer end) throws ServletException, IOException {
        PageInfo<Admin> alls = adminService.findAlls(start, end);
        request.setAttribute("userInfos",alls);
        return "userInfos";
    }
```

jsp：

```jsp
<table>
    <tr>
        <th>id</th>
        <th>用户名</th>
        <th>密码</th>
        <th>钱数</th>
    </tr>
    <c:forEach items="${userInfos.list}" var="item">
        <tr>
            <td>${item.id}</td>
            <td>${item.username}</td>
            <td>${item.password}</td>
            <td>${item.money}</td>
        </tr>
    </c:forEach>
    当前页码:${userInfos.pageNum}
</table>
```

pageNum	当前页码
pageSize	每页显示多少条数据
size	当前页显示多少条数据
startRow	第几条数据开始显示
endRow	显示到第几条数据
total	总共多少条数据
pages	总共多少页
startRow	第几条数据开始显示
list	查询的数据封装成列表
startRow	第几条数据开始显示
prePage	上一页
nextPage	下一页
isFirstPage	是否第一页
isLastPage	是否最后一页
hasPreviousPage	是否有上一页
hasNextPage	是否有下一页
navigatePages	显示多少个页码
navigateFirstPage	显示的第一个页码
navigateLagePage	显示的最后一个页码
navigatepageNums	将显示的页码封装成列表