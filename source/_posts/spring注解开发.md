---
title: spring注解开发
categories: spring
tags:
  - spring
excerpt: ''
cover: https://tse1-mm.cn.bing.net/th/id/R-C.52e5622d2443fa71cfdcb9bb6087911d?rik=PCz7sh2I3PKBGA&riu=http%3a%2f%2fimage.qianye88.com%2fpic%2f0e0289806288b7faabbc147abba9f0a8&ehk=09cAU0nEZS2AWz6G3CDfmCPcuCPNiQ0NAfNFysgn248%3d&risl=&pid=ImgRaw&r=0
---

### spring注解开发

1. java注解

- @Target(value = {ElementType.METHOD,ElementType.TYPE})
  - *@Target可以声明当前注解只能声明在哪里*
  - **当注解未指定value值时(value={})，则此注解可以用于任何元素之上**
  - TYPE：类或者接口上
  - FIELD：字段域声明，如@Value
  - METHOD：方法上
  - CONSTRUCTOR：构造函数声明
  - ANNOTATION_TYPE：标明注解可以用于注解声明(应用于另一个注解上)
  
- @Deprecated

  该注解是过期注解

- @SuppressWarnings({"uncheck","deprecation"})

  - 忽视指定警告
  - deprecation：忽视使用了不赞成使用的类或方法时的警告；
  - unchecked：执行了未检查的转换时的警告，例如当使用集合时没有用泛型 (Generics) 来指定集合保存的类型; 

- @Retention(value = RetentionPolicy.RUNTIME)

  - @Retention用来约束注解的生命周期，分别有三个值，源码级别(source)，类文件级别(class),运行时级别(runtime)
  - SOURCE：注解将被编译器丢弃（该类型的注解信息只会保留在源码里，源码经过编译后，注解信息会被丢弃，不会保留在编译好的class文件里）
  - CLASS：注解在class文件中可用，但会被JVM丢弃（该类型的注解信息会保留在源码里和class文件里，在执行的时候，不会加载到虚拟机中），请注意，当注解未定义Retention值时，默认值是CLASS，如Java内置注解，@Override、@Deprecated、@SuppressWarnning等
  - RUNTIME：注解信息将在运行期(JVM)也保留，因此可以通过[反射机制](https://so.csdn.net/so/search?q=反射机制&spm=1001.2101.3001.7020)读取注解的信息（源码、class文件和执行的时候都有注解的信息），如SpringMvc中的@Controller、@Autowired、@RequestMapping等。

- @Documented

  - @Documented被修饰的注解会生成到javadoc中

- @Inherited

  - @Inherited 可以让注解被继承，但这并不是真的继承，只是通过使用@Inherited，可以让子类Class对象使用getAnnotations()获取父类被@Inherited修饰的注解，如下：

~~~java
@Inherited
@Documented
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
public @interface DocumentA {
}

@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
public @interface DocumentB {
}

@DocumentA
class A{ }

class B extends A{ }

@DocumentB
class C{ }

class D extends C{ }

//测试
public class DocumentDemo {

    public static void main(String... args){
        A instanceA=new B();
        System.out.println("已使用的@Inherited注解:"+Arrays.toString(instanceA.getClass().getAnnotations()));

        C instanceC = new D();

        System.out.println("没有使用的@Inherited注解:"+Arrays.toString(instanceC.getClass().getAnnotations()));
    }
    /**
         * 运行结果:
         已使用的@Inherited注解:[@com.zejian.annotationdemo.DocumentA()]
         没有使用的@Inherited注解:[]
         */
}
~~~

  



------

### Ioc容器

1. 声明配置类

~~~java
//相当于xml配置文件
@Configuration
//@ComponentScan(value = "com.learnAnno",includeFilters = {
//        @ComponentScan.Filter(type = FilterType.ANNOTATION,classes = {Service.class})
//},useDefaultFilters = false)
@ComponentScan(value = "com.learnAnno",excludeFilters = {
        @ComponentScan.Filter(type = FilterType.ANNOTATION,classes = {Service.class})
})
//FilterType.ANNOTATION:按照注解
//FilterType.ASSIGNABLE_TYPE:按照类型
public class SpringConfig {

    @Bean//创建一个bean 此时id就是方法名person;也可以@Bean("name")
    public Person person(){
        return new Person("kkk","111");
    }
}
~~~

```java
//xml方式
//ApplicationContext classPathXmlApplicationContext = new ClassPathXmlApplicationContext("classpath:beans.xml");
//Person person = (Person) classPathXmlApplicationContext.getBean(Person.class);
//System.out.println(person);

//注解类方式
ApplicationContext annotationConfigApplicationContext = new AnnotationConfigApplicationContext(SpringConfig.class);
//Person person = (Person)annotationConfigApplicationContext.getBean("person");//按名称
Person person = (Person)annotationConfigApplicationContext.getBean(Person.class);//类型
System.out.println(person);
```

~~~java
//查找Configuration配置类中所有Person类型的bean
String[] beanNamesForType = annotationConfigApplicationContext.getBeanNamesForType(Person.class);
~~~

### 组件注册

2. @ComponentScan(value = "com.learnAnno")注解

```xml
<context:component-scan base-package="com.learnAnno"></context:component-scan>
<!--相当于xml中的写法 把Component Controller Service Repository注解得类都装载进来-->
```

3. Spring创建的bean默认都是单例的

   ```java
   //prototype:原型模式,ioc容器启动时并不会去调用方法创建对象放在容器中，每次获取的时候才会调用方法创建对象
   //singleton:单例模式(默认值),ioc容器启动会调用方法创建对象放到ioc容器中，以后每次获取的就是直接从容器(map.get())中拿。
   @Scope(value = "prototype")
   @Bean("beans")//创建一个bean 此时id就是方法名person;也可以@Bean("name")
   public Person person(){
       return new Person("kkk","111");
   }
   ```

4. 懒加载

   相当于xml文件中

   ~~~xml
   <bean name="user" class="com.shen.User" init-method="init" lazy-init="true"/>
   ~~~

   ```java
   @Scope(value = "singleton")
   @Lazy//singleton在ioc容器启动时并不会加载，只会在使用时再加载，且只加载一次
   @Bean("beans")//创建一个bean 此时id就是方法名person;也可以@Bean("name")
   public Person person(){
       System.out.println("hello world");
       return new Person("kkk","111");
   }
   ```

5. @Conditional(LinuxCon.class)

   - 当满足条件时，能够创建bean或类

~~~java
@Bean("windows")
@Conditional({WindowsCon.class})
public Person person1(){
    System.out.println(&quot;windows环境&quot;);
    return new Person(&quot;qqq&quot;,&quot;111&quot;);
}

@Bean("linux")
@Conditional(LinuxCon.class)
public Person person2(){
    System.out.println(&quot;linux环境&quot;);
    return new Person(&quot;qqq&quot;,&quot;111&quot;);
}
~~~

~~~java
public class LinuxCon implements Condition {
    @Override
    public boolean matches(ConditionContext context, AnnotatedTypeMetadata metadata) {
        Environment environment = context.getEnvironment();
        String property = environment.getProperty("os.name");
        if(property.contains("Linux")){
            return true;
        }
        return false;
    }
}
~~~

```java
public class WindowsCon implements Condition {
    /**
          *ConditionContext:判断条件能使用的上下文
          *AnnotatedTypeMetadata:注释信息
          */
    @Override
    public boolean matches(ConditionContext context, AnnotatedTypeMetadata metadata) {
        Environment environment = context.getEnvironment();
        String property = environment.getProperty("os.name");
        if(property.contains("Windows")){
            return true;
        }
        return false;
    }
}
```

6. @Import({Person.class})

~~~java
/**
     *给容器中注册组件：
     * 1）、包扫描+组件标注(@Controller,@Repository,@Service,@Component)
     * 2）、@Bean[导入第三方包里面的组件]
     * 3）、@Import[快速给容器中导入一个组件]，
     			1）@Import(要导入到容器中的组件)；容器就会自动注册这个组件，id默认是全类名
     			2）ImportSelector:返回需要导入的组件的全类名数组
     			3）ImportBeanDefinitionRegistrar：手动注册bean到容器中
     *4)、使用spring提供的FactoryBean方式(工厂Bean)：
     		1)、默认获取到的是工厂bean调用getObject创建的对象
     		2)、要获取工厂bean本身，我们需要给id前面加一个&
                 Object bean = applicationContext.getBean("&factoryBean");
                 System.out.println(bean);
     */
~~~

   - ImportSelector

```java
@Configuration
@ComponentScan(value = "com.learnAnno",includeFilters = {
    @ComponentScan.Filter(type = FilterType.ANNOTATION,classes = {Service.class}),
    @ComponentScan.Filter(type = FilterType.ASSIGNABLE_TYPE,classes = {AdminDao.class})
},useDefaultFilters = false)
@Import({Person.class, MySelector.class, MyImportBean.class})
public class SpringConfig {

}
```

~~~java
public class MySelector implements ImportSelector {
    /**
          *返回值，就是导入到容器中的组件全类名
          * AnnotationMetadata：当前标注Import注解的类额所有注解信息
          */
    @Override
    public String[] selectImports(AnnotationMetadata importingClassMetadata) {
        return new String[]{"com.learnAnno.pojo.Admin"};//返回的是全类名，不要返回null
    }
}
~~~

~~~java
public class MyImportBean implements ImportBeanDefinitionRegistrar {
    /**
          * AnnotationMetadata：当前类的注释信息
          * BeanDefinitionRegistry：BeanDefinition注册类
          *          把所有添加到容器中的bean，调用
          *          registry.registerBeanDefinition();手工注册进来
          */
    @Override
    public void registerBeanDefinitions(AnnotationMetadata importingClassMetadata, BeanDefinitionRegistry registry) {
        boolean b = registry.containsBeanDefinition("com.learnAnno.pojo.Admin");
        if(b==true){
            RootBeanDefinition rootBeanDefinition = new RootBeanDefinition(BeanDefRegist.class);
            registry.registerBeanDefinition("BeanDef", rootBeanDefinition);
        }
    }
}
~~~

​     

   - FactoryBean方式创建Bean

~~~java
public class MyFactoryBean implements FactoryBean<Person> {
    //    返回一个对象，这个对象会添加到容器中
    @Override
    public Person getObject() throws Exception {
        return new Person("ks","111");
    }
    @Override
    public Class<?> getObjectType() {
        return Person.class;
    }
    /**
          * true:单例模式
          * fasle:原型模式
          */
    @Override
    public boolean isSingleton() {
        return true;
    }
}
~~~

~~~java
@Bean("factoryBean")
public MyFactoryBean myFactoryBean(){
    return new MyFactoryBean();
}
~~~

### 生命周期

1. bean的生命周期：

​		bean创建---初始化---销毁的过程

​		容器管理bean的生命周期

​		我们可以自定义初始化和销毁方法；容器在bean进行到当前生命周期的时候来调用我们自定义的初始化和销毁方法

​	1）、指定初始化和销毁方法(@Bean())：

​					指定init-method=""和destory-method=""

​	2）、创建对象：

​					单实例：容器启动时候创建对象

​					多实例：每次获取时候创建对象

​	3）、初始化：

​					对象创建完成，并赋值好，调用初始方法

​			  销毁：

​					单实例：容器关闭的时候

​					多实例：容器不会管理这个bean

~~~java
@Data
@AllArgsConstructor
public class Person {
    private String username;
    private String password;

    public Person(){
        System.out.println("构造函数");
    }
    public void init(){
        System.out.println("初始化");
    }
    public void destory(){
        System.out.println("销毁方法");
    }
}
~~~

~~~java
@Bean(initMethod = "init",destroyMethod = "destory")
public Person person(){
    return new Person();
}
~~~

```java
AnnotationConfigApplicationContext applicationContext = new AnnotationConfigApplicationContext(SpringConfig2.class);
Object person = applicationContext.getBean("person");
applicationContext.close();
```

2. 实现InitializingBean, DisposableBean 接口来执行生命周期
3. 使用JSR250；@PostConstruct(构造方法之后)和@PreDestroy(销毁bean之前)注解

### 属性赋值

1. @Value

```java
@Data
@AllArgsConstructor
@NoArgsConstructor
public class Dog {
    @Value("hello world")
    private String name;
    @Value("${name}")
    private Integer age;
}
```

```java
@Configurable
@PropertySource("classpath:mess.properties")//中文乱码:encoding=utf-8
public class SpringConfig2 {
    @Bean
    public Dog dog(){
        return new Dog();
    }
}
```

2. @Autowired，自动注入

   - @Autowired按类型注入

     可与@Qualifier配合使用，此时按名称注入==>**注入的都是ioc容器中的其他bean**

   - 1)默认优先按照类型去容器中找对应的组件：applicationContext.getBean(Dog.class)

     2)如果找到多个相同类型的组件，再将属性的名称作为组件的id去容器中查找。

     ​				applicationContext.getBean("dog")

     3)@Qualifier("dog"),使用@Qualifier指定需要装配的组件的id，而不是属性名

     4)自动装配默认一定要将属性赋值好，没有就会报错

     ​		可以使用@Autowired(required = false)设置成非必须注入

     5)@Primary,让spring进行自动装配的时候，默认使用首选的bean

     ​		也可以继续使用@Qualifier指定需要装备的bean的名字

     6)@Resource,默认按照名称进行自动装配

   - @Autowired：构造器，参数，方法，属性，都是从容器中获取参数组件的值

     1）[标注在方法上]：@Bean+方法参数，参数从容器中获取；默认不写@Autowored效果是一样的，都能自动装配。

     2）[标在构造器上]：如果组件只有一个有参构造器。这个有参构造器的@Autowired可以省略,参数位置的组件还是可以自动从容器中获取。

     3）[放在参数位置]

   - 自定义组件想要使用Spring容器底层的一些组件（ApplicationContext,BeanFactory,xxx）,

     自定义组件实现xxxAware,在创建对象的时候，会调用接口规定的方法注入相关组件：Aware;

     把Spring底层一些组件注入到自定义的Bean中：

     xxxAware：功能使用xxxProcessor：

     ​			ApplicationContextAware==>ApplicationContextAwareProcessor

3. @Profile("dev")

   - *@Profile(“dev”)指定组件在那个环境的情况下才能被注册到容器中，不指定，任何环境下都能注册这个*

   -  加了环境标识的bean，只有这个环境被激活的时候才能注册到容器中

     @Profile(“default”)：默认环境

     ~~~java
     AnnotationConfigApplicationContext annotationConfigApplicationContext = new AnnotationConfigApplicationContext();
     annotationConfigApplicationContext.getEnvironment().setActiveProfiles("dev");
     annotationConfigApplicationContext.register(SpringConfig2.class);
     annotationConfigApplicationContext.refresh();
     
     String[] beanDefinitionNames = annotationConfigApplicationContext.getBeanDefinitionNames();
     for(String str:beanDefinitionNames){
         System.out.println(str);
     }
     ~~~

     ~~~java
     @Profile("dev")
     @Bean
     public Dog dog(){
         return new Dog();
     }
     ~~~

### Aop

1. Aop：【动态代理】

   指在程序运行期间动态的将某段代码切入到指定方法指定位置进行运行的编程方式。

   1）导入aop模块：Spring AOP(spring-aspects)

   2）定义一个业务逻辑类，在业务逻辑运行的时候将日志打印(之前，之后，返回，异常，环绕)

   3）定义一个日志切面类，切面类里面的方法需要动态感知业务逻辑类，

   4）给切面类的目标方法标注何时何地运行(通知注解)

   5）将切面类和业务逻辑类(目标方法所在类)都加入到容器中

   6）必须告诉Spring哪个类是切面类(给切面类上加一个注解@Aspect)

   7）给配置类上加@EnableAspectJAutoProxy注解(开启基于注解的aop)

   ​		相当于xml中<aop:aspectj-autoproxy/>

2. 代码实现

   **注解方式**

   ~~~java
   @Aspect
   public class SpringAopCon {
       /**
        * 配置切入点
        */
       @Pointcut("execution(* com.learnAnno.pojo.NumProcess.*(..))")
       public void pointcut(){}
   
       @Before("pointcut()")//JoinPoint一定要在参数第一位置
       public void before(JoinPoint joinPoint){
           Object[] args = joinPoint.getArgs();
           System.out.println(joinPoint.getSignature().getName()+"前置通知..."+ Arrays.asList(args));
       }
   
       @AfterReturning(value = "pointcut()",returning = "o")
       public void afterReturing(Object o){
           System.out.println("返回通知..."+o);
       }
   
       @AfterThrowing(value = "pointcut()",throwing = "exception")
       public void afterThrowing(Exception exception){
           System.out.println("异常通知..."+exception);
       }
   
       @After("pointcut()")
       public void after(JoinPoint joinPoint){
           System.out.println(joinPoint.getSignature().getName()+"最终通知...");
       }
   
   }
   ~~~

   ~~~java
   public class NumProcess {
       public Integer num(int i,int j){
           return i/j;
       }
   }
   ~~~

   ```java
   @Configurable
   @EnableAspectJAutoProxy
   public class SpringAopConfig {
   
       @Bean
       public NumProcess numProcess(){
           return new NumProcess();
       }
   
       @Bean
       public SpringAopCon springAopCon(){
           return new SpringAopCon();
       }
   
   }
   ```

   ```java
   AnnotationConfigApplicationContext annotationConfigApplicationContext = new AnnotationConfigApplicationContext(SpringAopConfig.class);
   NumProcess numProcess = (NumProcess) annotationConfigApplicationContext.getBean("numProcess");
   System.out.println(numProcess.num(1, 1));
   ```

   **xml方式**

   ```java
   public class SpringAopByXml {
   
       public void before(JoinPoint joinPoint){
           Object[] args = joinPoint.getArgs();
           System.out.println(joinPoint.getSignature().getName()+"前置通知..."+ Arrays.asList(args));
       }
   
       public void afterReturing(Object o){
           System.out.println("返回通知..."+o);
       }
   
       public void afterThrowing(Exception exception){
           System.out.println("异常通知..."+exception);
       }
   
       public void after(JoinPoint joinPoint){
           System.out.println(joinPoint.getSignature().getName()+"最终通知...");
       }
   }
   ```

   ```java
   public class NumProcess {
       public Integer num(int i,int j){
           return i/j;
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
                              http://www.springframework.org/schema/beans/spring-beans-4.1.xsd http://www.springframework.org/schema/context https://www.springframework.org/schema/context/spring-context.xsd http://www.springframework.org/schema/aop https://www.springframework.org/schema/aop/spring-aop.xsd">
   
       <bean class="com.learnAnno.pojo.NumProcess" id="numProcess"></bean>
       <bean class="com.learnAnno.Aop.SpringAopByXml" id="aopByXml"></bean>
   
       <aop:config>
           <aop:pointcut id="pointcut" expression="execution(* com.learnAnno.pojo.NumProcess.*(..))"/>
           <aop:aspect ref="aopByXml">
               <aop:before method="before" pointcut-ref="pointcut"></aop:before>
               <aop:after-returning method="afterReturing" pointcut-ref="pointcut" returning="o"></aop:after-returning>
               <aop:after-throwing method="afterThrowing" pointcut-ref="pointcut" throwing="exception"></aop:after-throwing>
               <aop:after method="after" pointcut-ref="pointcut"></aop:after>
           </aop:aspect>
       </aop:config>
   </beans>
   ```

   ```java
   ClassPathXmlApplicationContext classPathXmlApplicationContext = new ClassPathXmlApplicationContext("classpath:beans.xml");
   NumProcess numProcess = (NumProcess) classPathXmlApplicationContext.getBean("numProcess");
   System.out.println(numProcess.num(1, 0));
   String[] beanDefinitionNames = classPathXmlApplicationContext.getBeanDefinitionNames();
   Arrays.stream(beanDefinitionNames).forEach((e)->{
       System.out.println(e);
   });
   ```

### 声明式事务

1. 环境搭建：

   1）导入相关依赖

   - 数据源c3p0(0.9.2.1)、数据库驱动mysql-connection-java(5.1.44)、spring-jdbc(5.3.11)模块
   - spring-Jdbc(内部包含spring-jdbc、spring-tx其中spring-jdbc里面包含JdbcTemplate工具类，spring-tx掌管事务)

   2）配置数据源，JdbcTemplate(Spring提供的简化数据库操作的工具)，操作数据

   3）给方法上标注@Transactional 表示当前方法是一个事务方法

   4）@EnableTransactionManagement 开启基于注解的事务管理功能

   5）配置事务管理器来控制事务

   ~~~java
   @Bean
   public PlatformTransactionManager platformTransactionManager() throws Exception {
       return new DataSourceTransactionManager(dataSource());
   }
   ~~~

   个人理解：

   - 配置事务管理器来控制事务相当于xml配置中的org.springframework.jdbc.datasource.DataSourceTransactionManager
   - @EnableTransactionManagement相当于xml中的<tx:annotation-driven transaction-manager="dataSourceTransactionManager"/>
   - @Transactional和xml一样

   

   

2. configuration和configurable区别

   - configuration声明这个类是一个配置类，相当于原来的applicationContext.xml

   - configurable当我们手动new对象时，该对象内部无法实现依赖注入(@Autowired不好使)

     现在我们想在非Spring管理的类中使用依赖注入，可以使用@configurable

     ~~~java
     @Configurable(preConstruction = true)//告诉Spring在构造函数运行之前将依赖注入到对象中
     @Component
     public class Car {
      
         @Autowired
         private Engine engine;
         @Autowired
         private Transmission transmission;
      
         public void startCar() {
             transmission.setGear(1);
             engine.engineOn();
      
             System.out.println("Car started");
         }
     }
     ~~~

### spring注解整合（servlet3.0以上）

1. 
~~~java
//web容器启动的时候创建对象，调用方法来初始化容器以前前端控制器
public class MyWebAppInitializer extends AbstractAnnotationConfigDispatcherServletInitializer {
    //获取根容器的配置类：(Spring的配置文件) 父容器:
    @Override
    protected Class<?>[] getRootConfigClasses() {
        return new Class[]{SpringApplicationContext.class};
    }

    //获取web容器的配置类(SpringMvc配置文件) 子容器
    @Override
    protected Class<?>[] getServletConfigClasses() {
        return new Class[]{SpringMvc.class};
    }

    //获取DispatcherServlet的映射信息
    //  / :拦截所有请求(包括静态资源(xx.js,xx.png)) 但是不包括*.jsp
    //  /* :拦截所有请求，连*.jsp都拦截，jsp页面是tomcat的jsp引擎解析的。
    @Override
    protected String[] getServletMappings() {
        return new String[]{"/"};
    }
}
~~~

```java
@ComponentScan(basePackages = "com.myTest",excludeFilters = {
    @ComponentScan.Filter(type = FilterType.ANNOTATION,value = {Controller.class})
})
public class SpringApplicationContext {
}
```

```java
@ComponentScan(basePackages = "com.myTest",includeFilters = {
    @ComponentScan.Filter(type = FilterType.ANNOTATION,value = {Controller.class})
},useDefaultFilters = false)
public class SpringMvc {
}
```