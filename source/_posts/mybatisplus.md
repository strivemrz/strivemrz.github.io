---
title: BaseMapper和通用Service
categories: mybatis-plus
tags:
  - mybatis-plus
excerpt: ''
cover: https://desk-fd.zol-img.com.cn/t_s960x600c5/g5/M00/02/02/ChMkJ1bKxXKINmaMACMkdGofwOEAALHWAHFrGQAIySM530.jpg
---

#### 简介

mybatis-plus是一个mybatis的增强工具，在mybatis的基础上只做增强不做改变，为简化开发，提高效率而生。mybatis-plus提供了通用的mapper和service，可以在不编写任何sql语句的情况下，快速的实现对单表的CRUD，批量，逻辑删除，分页等操作。

#### 特性

- 无侵入：只做增强不做改变，引入他不会对现有工程产生影响。
- 损耗小：启动即会自动注入基本CRUD，性能基本无损耗，直接面向对象操作
- 强大的CRUD操作：内置通用mapper，通用service，仅仅通过少量配置即可实现单表大部分CRUD操作，更有强大的条件构造器，满足各类使用需求。
- 支持Lambda形式调用：通过Lambda表达式，方便的编写各类查询条件，无需再担心字段写错。
- 支持主键自动生成：支持多达4中主键策略(内含分布式唯一ID生成器)，可自由配置，完美解决主键问题
- 支持ActiveRecord模式：支持ActiveRecord形式调用，实体类只需继承Model类即可进行强大的CRUD操作。
- 支持自定义全局通用操作：支持全局通用方法注入
- 内置代码生成器：采用代码或者Maven插件可快速生成mapper，model，service，controller层代码，支持模板引擎
- 内置分页插件：基于mybatis物理分页，开发者无需关心具体操作，配置好插件之后，写分页等同于普通List查询。
- 分页插件支持多种数据库：支持mysql，oracle，db2，sqlserver等多种数据库
- 内置性能分析插件：可输出sql语句以及执行时间
- 内置全局拦截插件：提供全表delete，update操作智能分析阻断，也可以自定义拦截规则，预防误操作。

#### 构造器简介

mybatis-plus通过EntityWrapper(简称EW，MP封装的一个查询条件构造器)或者Condition(与EW类似)，来让用户自由的构建查询条件，简单便捷，没有额外的负担，能够有效提高开发效率，他主要用于处理sql拼接，排序，实体参数查询等。

> 注意：使用的是数据库字段，不是 Java 属性！
>

> 警告：MyBatis-Plus不支持以及不赞成在 RPC 调用中把 Wrapper 进行传输，Wrapper 很重，传输 Wrapper 可以类比为你的 controller 用 map 接收值(开发一时爽，维护火葬场)，正确的 RPC 调用姿势是写一个 DTO 进行传输，被调用方再根据 DTO 执行相应的操作。

#### BaseMapper

##### 查询所有数据

- 导入依赖

```xml
<!-- mybatis-plus 这里数据库连接池用的是druid，所以把HikariCP去掉了-->
<dependency>
    <groupId>com.baomidou</groupId>
    <artifactId>mybatis-plus-boot-starter</artifactId>
    <version>3.5.2</version>
    <exclusions>
        <exclusion>
            <artifactId>HikariCP</artifactId>
            <groupId>com.zaxxer</groupId>
        </exclusion>
    </exclusions>
</dependency>
```

- 写mapper接口类

```java
// 这里可以写@Mapper 或者直接在启动类加 @MapperScan("包路径")，当遇到@mapper注解时，会先将mapper类生成代理类加入到IOC容器中，后者是扫描整体包，把所有mapper类都生成代理类，加入到IOC中
public interface AdminDao extends BaseMapper<Admin> {
}
```

- 测试

```java
@SpringBootTest
@MapperScan("com.mybatisplus.mybatisplus.dao")
class MybatisPlusApplicationTests {

    @Autowired
    private AdminDao adminDao;

    @Test
    void contextLoads() {
        List<Admin> admins = adminDao.selectList(null);
        admins.forEach(System.out::println);
    }
}
```

##### 日志功能配置

```yml
#mybatis-plus日志功能
mybatis-plus:
  configuration:
    log-impl: org.apache.ibatis.logging.stdout.StdOutImpl
```

##### 新增数据

```java
/**
 * 新增数据
 */
@Test
public void testInsert(){
    Admin admin=new Admin();
    admin.setUsername("张三");
    admin.setPassword("123");
    admin.setMessage("无信息");
    int insert = adminDao.insert(admin); //返回受影响行数
    System.out.println("result:"+insert);
    System.out.println(admin.getId()); //返回主键id
}
```

##### 删除数据

```java
/**
 * 删除数据
 */
@Test
public void testDelete(){
    //根据id删除
    int result = adminDao.deleteById(2000695123298L);
    System.out.println("result:"+result);
    
    //根据map条件删除
    Map<String,Object> map=new HashMap<>();
    map.put("id",2);
    map.put("username","user1");
    int result = adminDao.deleteByMap(map);
    System.out.println("result:"+result);
    
    //批量删除
    List<Long> longs = Arrays.asList(1L, 2L, 3L);
    int result = adminDao.deleteBatchIds(longs);
    System.out.println("result:"+result);
}
```

##### 修改数据

```java
/**
 * 修改数据
 */
@Test
public void testUpdate(){
    //根据id修改数据
    Admin admin=new Admin();
    admin.setId(4);
    admin.setMessage("修改信息...");
    int result = adminDao.updateById(admin);
    System.out.println("result:"+result);
}
```

##### 查询数据

```java
/**
 * 查询数据
 */
@Test
public void testSelect(){
    //根据id查询
    Admin admin = adminDao.selectById(2);
    System.out.println("单个数据:"+admin);
    
    //根据id批量查询
    List<Admin> admins = adminDao.selectBatchIds(Arrays.asList(1, 2));
    admins.forEach(adminMes->{
        System.out.println("根据id查询数据:"+adminMes);
    });
    List<Admin> admins1 = adminDao.selectList(null);
    admins1.forEach(allMess->{
        System.out.println("所有数据:"+allMess);
    });
}
```

##### 自定义功能

- 编写新接口

```java
@Repository
public interface AdminDao extends BaseMapper<Admin> {

    /**
     * 查询数据ById
     * @param id
     * @return
     */
    public Map<String,Object> selectByMap(String id);
}
```

- 接口实现(mapper-location默认是classpath:/mapper/**/*.xml)

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper
        PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="com.mybatisplus.mybatisplus.dao.AdminDao">
    <select id="selectByMap" parameterType="string" resultType="map">
        select id,username,password,message from admin where id=#{id}
    </select>
</mapper>
```

- 测试

```java
/**
 * 自定义查询
 */
@Test
public void testCustomization(){
    Map<String, Object> map = adminDao.selectByMap("2");
    System.out.println(map);
}
```



#### 通用service接口

##### 基础搭建

- 创建service接口

```java
public interface AdminService extends IService<Admin> { //这里泛型写的是实体类
}
```

- 创建service接口实现类

```java
//通过继承ServiceImpl，从而不用自己实现Iservice中的抽象方法
//ServiceImpl类中，第一个泛型是操作的mapper接口，第二个是返回的实体类
@Service
public class AdminServiceImpl extends ServiceImpl<AdminDao, Admin> implements AdminService {
}
```

##### 查询数据总数

```java
@Slf4j
@SpringBootTest
@MapperScan("com.mybatisplus.mybatisplus.dao")
public class MybatisPlusService {

    @Autowired
    private AdminService adminService;

    @Test
    public void selectCount(){
        long count = adminService.count();
        System.out.println("数据总数:"+count);
    }
}
```

##### 批量添加

```java
@Test
public void batchAdd(){
    List<Admin> list=new ArrayList<>();
    for(int i=0;i<50;i++){
        Admin admin=new Admin();
        admin.setUsername("zdh"+(i+1));
        admin.setPassword("pass"+(i+1));
        admin.setMessage("无添加信息");
        list.add(admin);
    }
    boolean isBool = adminService.saveBatch(list);//有id修改，没有id新增，多次循环insert，不是在一条sql语句上无限扩充
    System.out.println("是否添加成功?"+isBool);
}
```