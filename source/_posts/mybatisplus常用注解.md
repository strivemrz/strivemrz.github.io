---
title: mybatis-plus常用注解
categories: mybatis-plus
tags:
  - mybatis-plus
excerpt: ''
cover: https://ts1.cn.mm.bing.net/th/id/R-C.b17d24660509d2e47714b0b94bfc0ebb?rik=X%2fAB0hc50VfHxg&riu=http%3a%2f%2fimg.mm4000.com%2ffile%2f5%2fb6%2f24f356ee2c.jpg&ehk=k3LV0YERgsdechEBWcSzd4eXw8%2f%2b%2bqrYh3OgNfVPpn4%3d&risl=1&pid=ImgRaw&r=0
---

#### 全局设置统一前缀/主键生成策略

```yml
#mybatis-plus日志功能
mybatis-plus:
  configuration:
    log-impl: org.apache.ibatis.logging.stdout.StdOutImpl
  type-aliases-package: com.mybatisplus.mybatisplus.pojo
  global-config:
    db-config:
      #统一前缀 t_
      table-prefix: t_
      #主键生成策略 auto递增
      id-type: auto
```

#### 常用注解

##### @TableName

> 当实体类和数据库表名不一致情况下，可以用@TableName注解修饰

```java
@Data
@NoArgsConstructor
@AllArgsConstructor
@TableName("t_admin") //此时操作的就是t_admin这张表
public class Admin {
    private Integer id;
    private String username;
    private String password;
    private String message;
}
```

##### @TableId

> 将属性对应的字段指定为主键
>
> @TableId注解的value属性用于指定主键的字段(指定成跟数据库中的主键字段一样)
>
> @TableId注解的type属性设置主键生成策略

```java
@Data
@NoArgsConstructor
@AllArgsConstructor
@TableName("t_admin")
public class Admin {
    @TableId(value = "uid",type = IdType.AUTO) //主键自增策略
    private Integer id;
    private String username;
    private String password;
    private String message;
}
```

```java
@Test
public void testSelect(){
    Admin admin = adminDao.selectById(2);
    System.out.println("查询结果:"+admin);
}
```

##### @TableField

> mybatis-plus中会把实体类中的半驼峰命名和下划线命名全部转换成下划线形式
>
> @TableField可以解决数据库与实体类字段不一致情况

```java
@Data
@NoArgsConstructor
@AllArgsConstructor
@TableName("t_admin")
public class Admin {
    @TableId(value = "uid")
    private Integer id;

    @TableField(value = "user_name") //将username与数据库中的user_name字段匹配
    private String username;
    private String password;
    private String message;
}
```

##### @TableLogic(逻辑删除)

```java
@Data
@NoArgsConstructor
@AllArgsConstructor
@TableName("t_admin")
public class Admin {
    @TableId(value = "uid")
    private Integer id;

    @TableField(value = "user_name")
    private String name;
    private String password;
    private String message;

    @TableLogic //逻辑删除 0正常状态，其他删除状态
    private String isDeleted;
}
```