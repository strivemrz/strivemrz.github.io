---
title: spring
categories: spring
tags:
  - spring
excerpt: ''
cover: https://img.tt98.com/d/2020/2020062817049349/5ef856830ffc3.jpg
---

springIOC

* ![image-20220103164542950](C:\Users\19651\AppData\Roaming\Typora\typora-user-images\image-20220103164542950.png)

 ```java
  ApplicationContext application=new ClassPathXmlApplicationContext("applicationContext.xml");
  SpringS prin = application.getBean("prin", SpringS.class);
  prin.prin();
 ```

核心容器的两个接口引发出的问题

ApplicationContext

他在构建核心容器时，创建对象采取的策略是采用立即加载的方式。也就是说，只要一读取完配置文件马上就创建配置文件中配置的对象

```java
//单例模式适用
ApplicationContext application=new ClassPathXmlApplicationContext("applicationContext.xml");//直接把配置文件里的所有bean全部创建
SpringS prin = application.getBean("prin", SpringS.class);
prin.prin();
```

BeanFactory

他在构建核心容器时，创建对象采取的策略是采用延迟加载的方式。也就是说，什么时候根据id获取对象了，什么时候才真正的创建对象

```java
//多例模式适用
Resource resource=new ClassPathResource("applicationContext.xml");//仅在使用时创建对象
BeanFactory beanFactory=new XmlBeanFactory(resource);
SpringS prin = beanFactory.getBean("prin", SpringS.class);
prin.prin();
```

spring创建对象的三种方式

 一、使用默认构造函数创建，在spring的配置文件中使用bean标签，配以id和class属性之后，且没有其他属性和标签时，采用的是默认构造函数创建bean对象，如果类中没有默认构造函数，则对象无法创建

  ```java
  <!--    采用默认构造函数创建对象-->
      <bean id="prin" class="com.springTest.SpringS"></bean>
  ```

 二、使用普通工厂中的方法创建对象(使用某个类中的方法创建对象并存入spring容器)

  ```java
  <!--    采用某个类中的方法创建对象-->
      <bean id="prinsk" class="com.springTest.SpringS"></bean>
      <bean id="sec" factory-bean="prinsk" factory-method="retu"></bean>
  ```

 三、使用工厂中的静态方法创建对象(使用某个类中的静态方法创建对象，并存入spring容器)

  ```java
  <!--    使用某个类中的静态方法创建对象，并存入spring容器-->
      <bean id="sk" class="com.springTest.SpringS" factory-method="retu"></bean>
  ```

![image-20220103193515488](C:\Users\19651\AppData\Roaming\Typora\typora-user-images\image-20220103193515488.png)

bean的作用范围调整

bean标签的scope属性

​		作用：用于指定bean的作用范围

​		取值：常用的就是单例和多例的

​				singleton:单例的(默认值)

​				prototype:多例的

​				request:作用于web应用的请求范围

​				session:作用于web应用的会话范围

​				global-session:作用于集群环境的会话范围(全局会话范围)，当不是集群时，它就是session

注入

构造函数注入：

涉及的标签：constructor-arg

标签出现的位置：bean标签的内部

标签中的属性

​	type：用于指定要注入的数据的数据类型，该数据类型也是构造函数中某个或某些参数的类型

​	index：用于指定要注入的数据给构造函数中指定索引位置的参数赋值。索引的位置是从0开始

​	name：用于指定给构造函数中指定名称的参数赋值

-------------------------------------------------------------------------------------

​	value

​	ref

```xml
<bean id="prin" class="com.springTest.SpringS">
    <constructor-arg name="username" value="aaa"></constructor-arg>
    <constructor-arg name="password" value="111"></constructor-arg>
</bean>
```

set注入:

涉及的标签：property

标签出现的位置：bean标签的内部

标签中的属性

​	name：用于指定注入时所调用的set**方法名称**

​	value：用于提供基本类型和String类型的数据

​	ref：用于指定其他的bean类型数据。他指的就是在spring的Ioc核心容器中出现过的bean对象

优势：创建对象时没有明确的限制，可以直接使用默认构造函数

弊端：如果有某个成员必须有值，则获取对象是有可能set方法没有执行

```xml
<bean id="prin" class="com.springTest.SpringS">
    <property name="username" value="aaa"></property>
    <property name="password" value="222"></property>
</bean>
```

​			3.复杂类型的注入\集合类型注入

​				用于给List结构注入的标签：

​						list array set

​				用于给Map结构集合注入的标签：

​						map props

​				结构相同，标签可以互换

```xml
<bean id="coll" class="com.springTest.SpringS">
    <property name="list">
        <list>
            <value>aaa</value>
            <value>bbb</value>
            <value>ccc</value>
        </list>
    </property>
    <property name="set">
        <set>
            <value>kkk</value>
            <value>ppp</value>
        </set>
    </property>
    <property name="strings">
        <array>
            <value>qqq</value>
            <value>www</value>
        </array>
    </property>

    <property name="map">
        <map>
            <entry key="sss" value="ooo"></entry>
            <entry key="rrr"><value>hhh</value></entry>
        </map>
    </property>
    <property name="properties">
        <props>
            <prop key="lll">pppppp</prop>
        </props>
    </property>
</bean>
```

spring注解开发

* 用于创建对象的

  *他们的作用就和在XML配置文件中编写一个<bean>标签实现的功能是一样的*

  Component:

  ​					作用：用于把当前类对象存入spring容器中

  ​					属性：value：用于指定bean的id当我们不写时，他的默认值是当前类名，且首字母改小写

```java
@Component
public class SpringS {}
```

```xml
  <?xml version="1.0" encoding="UTF-8"?>
  <beans xmlns="http://www.springframework.org/schema/beans"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xmlns:context="http://www.springframework.org/schema/context"
         xsi:schemaLocation="http://www.springframework.org/schema/beans
          https://www.springframework.org/schema/beans/spring-beans.xsd 
          http://www.springframework.org/schema/context 
          https://www.springframework.org/schema/context/spring-context.xsd">
  <!--    spring组件扫描-->
      <context:component-scan base-package="com.springTest"></context:component-scan>
  </beans>
```

  Controller：一般用在表现层

  Service：一般用在业务层

  Repository：一般用在持久层

  以上三个注解他们的作用和属性与Component是一模一样。

  他们三个是spring框架为我们提供明确的三层使用的注解，使我们三层对象更加清晰

* 用于注入数据的

  他们的作用就和在xml配置文件中的bean标签中写一个<property>标签的作用是一样的

  Autowired：

  ​				作用：自动按照类型注入。只要容器中有**唯一**的一个bean对象类型和要注入的变量类型匹配，就可以注入成功

  ​							如果Ioc容器中没有任何bean的类型和要注入的变量类型匹配，则报错

  ​							如果Ioc容器中有多个类型匹配时，则根据变量名匹配Map中的key，一样则注入成功，不一样则报错

  ![image-20220104141715692](C:\Users\19651\AppData\Roaming\Typora\typora-user-images\image-20220104141715692.png)

  出现位置：可以是变量上，也可以是方法上

  细节：在使用注解注入式，set方法就不是必须的了。

  Qualifier：作用：再按照类中注入的基础之上再按照名称注入。他在给类成员注入时不能单独使用，但是在给方法参数注入时可以

  ​					属性：value：用于指定注入bean的id

  Resource：直接按照bean的id注入，它可以独立使用

  ​					属性：name：用于指定bean的id。

  以上三个注入都只能注入其他bean类型的数据，而基本类型和String类型无法使用上述注解实现。另外，集合类型的注入只能通过xml来实现

  Value：

  ​					作用：用于注入基本类型和String类型的数据

  ​					属性：value用于指定数据的值。它可以使用spring中的SpEL(也就是spring的el表达式) SpEL的写法。${表达式}

  用于改变作用范围的：

  ​					他们的作用就和在bean标签中使用scope属性实现的功能是一样的

  ​					Scope(作用域是一级一级往上查的，套娃)：作用：用于指定bean的作用范围

  ​					属性：value：指定范围的取值。常用取值：singleton  prototype		

  和生命周期相关 了解

  ​						他们的作用就和在bean标签中使用init-method和destroy-method的作用是一样的

  ​						PreDestroy：作用：用于指定销毁方法

  ​						PostConstruce：作用：用于指定初始化方法	

```java
  @Component
  @Scope("singleton")
  public class SpringS {
      
      //初始化方法 在创建完对象之后调用
      @PostConstruct
      public void init(){
          System.out.println("用户调用初始化方法");
      }
      //销毁方法 只在单例模式下有用
      @PreDestroy
      public void destroy(){
          System.out.println("用户调用销毁方法");
      }
  }
```

  ​		

* spring新注解

  该类是一个配置类，它的作用和bean.xml是一样的

  spring中的新注解

  Configuration

  ​					作用：指定当前类是一个配置类

  ​					细节：当配置类作为AnnotationConfigApplicationContext对象创建的参数时，该注解可不写

```java
  ApplicationContext annotationConfigApplicationContext = new AnnotationConfigApplicationContext(SpringS.class);
```
			@ComponentScan("com.springTest")

​									作用：用于通过注解指定spring在创建容器时要扫描的包

​									属性：value：它和basePackages的作用是一样的都是用于指定创建容器时要扫描的包，使用此注解就等于在xml中配置了：

```xml
<context:component-scan base-package="com.springTest"></context:component-scan>
```

​				Bean

​						作用：用于把当前方法的返回值作为bean对象存入spring的ioc容器中

​						属性：name：用于指定bean的id。当不写时，默认值是当前方法的名称

​						细节：当我们使用注解配置方法时，如果方法有参数，spring框架会去容器中查找有没有可用的bean对象。查找的方式和Autowired注解的作用是一样的

```java
@Bean("name")
public SpringS getSpr(){
    return new SpringS();
}
```

​				Import

​							用于导入其他的配置类

​				@PropertySource

​							作用：用于指定properties文件的位置

​							属性：value：指定文件的名称和路径。

​										关键字：classpath：表示类路径下

spring整合junit

应用程序的入口

main方法

junit单元测试中，没有main方法也能执行

junit集成了一个main方法

该方法就会判断当前测试类中哪些方法有@Test注解

junit就让有Test注解的方法执行

junit不会管我们是否采用spring框架

​	在执行测试方法时，junit根本不知到我们是不是使用了spring框架所以也就不会为我们读取配置文件/配置类创建spring核心容器

由以上三点可知，当测试方法执行时，没有Ioc容器，就算写了Autowired注解，也无法实现注入

--------------------------------------

使用junit单元测试，测试spring的配置

spring整合junit的配置

1. 导入spring整合junit的jar(坐标spring-test)

2. 使用junit提供的一个注解把原有的main方法替换了，替换成spring提供的@Runwith

3. 告知spring的运行器，spring和ioc创建是基于xml还是注解的，并且说明位置

   @ContextConfiguration

   ​		locations：指定xml文件的位置，加上classpath关键字，表示在类路径下

   ​		classes：指定注解类所在的位置

4. 当使用spring5.x版本的时候，要求junit的jar必须是4.12及以上

   ```java
   @RunWith(SpringJUnit4ClassRunner.class)
   @ContextConfiguration(classes = beans.class)//@ContextConfiguration(locations ="classpath:applicationContext.xml")
   public class AppTest
   {
       @Autowired
       @Qualifier("sk")
       private SpringS springS=null;
   
       @Test
       public void shouldAnswerWithTrue() throws InterruptedException {
   //        ClassPathXmlApplicationContext applicationContext = new ClassPathXmlApplicationContext("applicationContext.xml");
   //        SpringS springS = (SpringS) applicationContext.getBean("springS");
   //        ApplicationContext bean = new AnnotationConfigApplicationContext(beans.class);
   //        SpringS springS = bean.getBean("sk", SpringS.class);
           System.out.println(springS.sk.getUsername());
       }
   }
   ```

ThreadLocal

ThreadLocal是解决线程安全问题一个很好的思路，它通过为每个线程提供一个独立的变量副本解决了变量并发访问的冲突问题。在很多情况下，ThreadLocal比直接使用synchronized同步机制解决线程安全问题更简单，更方便，且结果程序拥有更高的并发性。

动态代理

JDK动态代理

```Java
public class JDKProxy implements InvocationHandler {

    private Object object;

    public void setObject(Object object) {
        this.object = object;
    }

    public Object getProxy(Object object){
        this.object=object;
        return Proxy.newProxyInstance(JDKProxy.class.getClassLoader(),object.getClass().getInterfaces(),this);
    }
    @Override
    public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
        System.out.println("前置增强");
        Object objects=method.invoke(object,args);
        System.out.println("后置增强");
        return objects;
    }
}
```

CGLIB动态代理

```java
public class CGLIBProxy implements MethodInterceptor {

    public Object getPro(Object object){
        Enhancer enhancer=new Enhancer();
        enhancer.setSuperclass(object.getClass());
        enhancer.setCallback(this);
        return enhancer.create();
    }
    @Override
    public Object intercept(Object o, Method method, Object[] objects, MethodProxy methodProxy) throws Throwable {
        System.out.println("前置增强");
        Object o1=methodProxy.invokeSuper(o,objects);
        System.out.println("后置增强");
        return o1;
    }
}
```

springAop专业术语

连接点(Joinpoint)：被代理类的所有接口方法

切点(Pointcut)：被代理类增强的接口方法

通知(Advice)：切点增强的一些功能  添加日志、管理事务

目标对象(Target)：需要对他进行通知的业务类

织入：将通知添加到目标类具体连接点上的过程

代理：一个类被Aop织入后生成一个结果类，他是融合了原类和通知逻辑的代理类

切面：切面由切点和通知组成

spring切入点写法

标准写法：public void com.springTest.SpringAopTest.test(int,int)

访问修饰符可以省略：void com.springTest.SpringAopTest.test(int,int)

返回值可以使用通配符，表示任意返回值：* com.springTest.SpringAopTest.test(int,int)

包名可以使用通配符，表示任意包，但是有几级包，就需要写几个*： * * . * . * .Account(..)

包名可以使用..表示当前包及子包(中间可以有一个或多个子包)：* * .. Account(..)

全通配写法：* *..*.*(..)

实际开发中切点的常用写法：* com.Aop.impl.* . *(..)

spring通知

```xml
<!--    springAOP配置-->
    <aop:config>
        <aop:pointcut id="pot1" expression="execution(public * com.springTest.SpringAopTest.*())"/>
<!--        配置切面-->
        <aop:aspect id="aspect1" ref="springAops">
<!--            前置通知-->
            <aop:before method="show" pointcut-ref="pot1"></aop:before>
<!--            后置通知-->
            <aop:after-returning method="retu" pointcut-ref="pot1"></aop:after-returning>
<!--            异常通知-->
            <aop:after-throwing method="throwk" pointcut-ref="pot1"></aop:after-throwing>
<!--            最终通知-->
            <aop:after method="after" pointcut-ref="pot1"></aop:after>
<!--            环绕通知  增强方法中要有ProceedingJoinpoint-->
            <aop:around method="show" pointcut-ref="pot1"></aop:around>
        </aop:aspect>
    </aop:config>
```

springAop注解实现

```Java
package com.springTest;

import org.aspectj.lang.ProceedingJoinPoint;
import org.aspectj.lang.annotation.*;
import org.springframework.stereotype.Component;

@Component("springAops")
@Aspect
public class SpringAops {
//    springAop环绕通知
    @Pointcut("execution(public * com.springTest.SpringAopTest.*())")
    public void pot1(){}
    @Around("pot1()")
    public Object show(ProceedingJoinPoint poj){
        Object ob=null;
        try {
            System.out.println("前置通知");
            ob=poj.proceed();
            System.out.println("后置通知");
            return ob;
        } catch (Throwable e) {
            e.printStackTrace();
            System.out.println("异常通知");
        }finally {
            System.out.println("最终通知");
        }
        return ob;
    }
    @AfterReturning("pot1()")
    public void after(){
        System.out.println("后置通知");
    }
    @AfterThrowing("pot1()")
    public void throwk(){
        System.out.println("异常通知");
    }
    @After("pot1()")
    public void retu(){
        System.out.println("最终通知");
    }
}
```

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context"
       xmlns:aop="http://www.springframework.org/schema/aop"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
        https://www.springframework.org/schema/beans/spring-beans.xsd
        http://www.springframework.org/schema/context
        https://www.springframework.org/schema/context/spring-context.xsd
        http://www.springframework.org/schema/aop
        https://www.springframework.org/schema/aop/spring-aop.xsd">
<!--    spring组件扫描-->
    <context:component-scan base-package="com.springTest"></context:component-scan>
<!--    开启AOP注解-->
    <aop:aspectj-autoproxy/>
</beans>
```

Spring事务管理

```java
<!--    配置事务管理-->
    <bean class="org.springframework.jdbc.datasource.DataSourceTransactionManager" id="dataSourceTransactionManager">
        <!-- 向事务中注入源数据 -->
        <property name="dataSource" ref="dataSource"></property>
    </bean>
        <!-- 配置事务的通知 -->
    <tx:advice id="transactionInterceptor" transaction-manager="dataSourceTransactionManager">
        /**
        *isolation:用于指定事务的隔离级别。默认值是DEFAULT，表示使用数据库的默认隔离级别
        *propagation:用于指定事务的传播行为。默认值是REQUIRED，表示一定会有事务，增删改的选择。默认方法可以选择SUPPORES
        *read-only:用于指定事务是否只读。只有查询方法才能设置为true，默认是false表示读写
        *timeout：用于指定事务的超时时间，默认值是-1，表示永不超时。如果指定了数值，以秒为单位。
        *rollback-for:用于指定一个异常，当产生该异常时，事务回滚，产生其他异常时事务不回滚，没有默认值，表示任何异常都回滚。
        *no-rollback-for：用于指定一个异常，当产生该异常时，事务不回滚，产生其他异常时，事务回滚，没有默认值，表示任何异常都回滚
        */
        <tx:attributes>
            <tx:method name="*" propagation="REQUIRED" read-only="false"/>
            <tx:method name="find*" propagation="SUPPORTS" read-only="true"/>
        </tx:attributes>
    </tx:advice>
    <!--配置事务的切面-->
    <aop:config>
        <!--设置pointcut 表达式，并且把定义好的事务属性应用到切入点-->
        <aop:pointcut id="pot1" expression="execution(* com.SpringTransaction.service.impl.*.*(..))"/>
        <aop:advisor advice-ref="transactionInterceptor" pointcut-ref="pot1"></aop:advisor>
    </aop:config>
<!--    proxy-target-class="true"设置为true则表示为cglib-->
<!--    proxy-target-class="false"则为jdk动态代理-->
    <aop:aspectj-autoproxy  proxy-target-class="true"/>
```

spring事务管理注解方式实现

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
        xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
        xmlns:aop="http://www.springframework.org/schema/aop"       
        xmlns:tx="http://www.springframework.org/schema/tx" 
        xmlns:context="http://www.springframework.org/schema/context"
        xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd
        http://www.springframework.org/schema/tx http://www.springframework.org/schema/tx/spring-tx.xsd
        http://www.springframework.org/schema/aop http://www.springframework.org/schema/aop/spring-aop.xsd
        http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context.xsd"> 
<!-- 配置数据源 -->
<bean id="dataSource" class="org.springframework.jdbc.datasource.DriverManagerDataSource">
  <property name="driverClassName" value="com.mysql.jdbc.Driver"></property>
  <property name="url" value="jdbc:mysql:///spring_demo"/>
  <property name="username" value="root"/>
  <property name="password" value="root"/>  
</bean>  
<!-- 配置JdbcTemplate -->
<bean id="jdbcTemplate" class="org.springframework.jdbc.core.JdbcTemplate">
		<property name="dataSource" ref="dataSource" />
</bean>
<!-- 配置事务管理器 -->
<bean id="transactionManager" class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
	<property name="dataSource" ref="dataSource"></property>
</bean>
<!-- 开启事务注解 -->
<tx:annotation-driven transaction-manager="transactionManager"/>
<!-- 配置实体类 -->
<bean id="accountBiz" class="com.tx.AccountBiz">
	<property name="accountDao" ref="accountDao"></property>
</bean>
<bean id="accountDao" class="com.tx.AccountDao">
	<property name="jdbcTemplate" ref="jdbcTemplate"></property>
</bean>
</beans>
```

```java
package com.tx;
import org.springframework.jdbc.core.JdbcTemplate;
import org.springframework.transaction.annotation.Transactional;
//使用@Transactional注解实现事务管理
@Transactional
public class AccountDao {
	private JdbcTemplate jdbcTemplate;
	public void setJdbcTemplate(JdbcTemplate jdbcTemplate) {
		this.jdbcTemplate = jdbcTemplate;
	}
	//银行转账
	public void transfer(){
		add();
		int i=10/0;
		reduce();		
	}
	//增加余额
	public void add(){
		String sql="update account set balance=balance+? where accountNo=?";
		jdbcTemplate.update(sql, 500,655925);
	}
	//减少余额
	public void reduce(){
		String sql="update account set balance=balance-? where accountNo=?";
		jdbcTemplate.update(sql, 500,358788);
	}
}
```
