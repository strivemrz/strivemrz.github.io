---
title: mybatis-plus插件使用
categories: mybatis-plus
tags:
  - mybatis-plus
excerpt: ''
cover: https://img.tt98.com/d/file/tt98/2020032519004984/5e7b285f92300.jpg
---

#### mybatis-plus分页插件的配置和使用

##### 基础使用

- 加入PaginationInnerInterceptor拦截器

```java
@Configuration
@MapperScan("com.mybatisplus.mybatisplus.dao")
public class MybatisPlusConfig {

    @Bean
    public MybatisPlusInterceptor mybatisPlusInterceptor(){
        MybatisPlusInterceptor mybatisPlusInterceptor=new MybatisPlusInterceptor();
        mybatisPlusInterceptor.addInnerInterceptor(new PaginationInnerInterceptor(DbType.MYSQL));
        return mybatisPlusInterceptor;
    }
}
```

- 测试分页效果

```java
@Test
public void test01(){
    IPage<Admin> iPage=new Page<>(2,2);
    IPage<Admin> iPage1 = adminDao.selectPage(iPage, null);// 第几页，查询条件
    System.out.println("当前页:"+iPage1.getCurrent());
}
```

##### 分页相关数据获取

```java
@Test
public void testGetPages(){
    Page<Admin> page=new Page<>(1,3);
    adminDao.selectPage(page,null);

    System.out.println("有多少页:"+page.getPages());
    System.out.println("当前第几页:"+page.getCurrent());
    List<Admin> records = page.getRecords();
    System.out.println("获取到的数据信息:"+records);
    System.out.println("每页数量:"+page.getSize());
    System.out.println("总数据量:"+page.getTotal());
    System.out.println("是否有下一页:"+page.hasNext());
    System.out.println("是否有上一页:"+page.hasPrevious());
}
```

##### 自定义分页功能

- 自定义接口

```java
public interface AdminDao extends BaseMapper<Admin>{

    /**
     * 根据uid长度进行划分
     * 因为加了PaginationInnerInterceptor拦截器，当这个接口用到Page对象的时候，会在sql语句后面默认加limit进行分割
     * @param page
     * @param uid
     * @return
     */
    public Page<Admin> selectAdminPages(@Param("page") Page<Admin> page,@Param("uid") String uid);
}
```

- xml对应映射

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper
        PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="com.mybatisplus.mybatisplus.dao.AdminDao">
    <select id="selectAdminPages" resultType="admin">
        select * from t_admin where uid >= #{uid}
    </select>
</mapper>
```

- 测试调用

```java
@Test
public void testGetDiy(){
    /**
    * ==>  Preparing: select * from t_admin where uid >= ? LIMIT ?,?
    * ==> Parameters: 3(String), 2(Long), 2(Long)
    */
    Page<Admin> page=new Page<>(2,2);
    adminDao.selectAdminPages(page, "3");
    List<Admin> records = page.getRecords();
    System.out.println("获取到的数据:"+records);
}
```



#### 乐观锁和悲观锁

##### 说明

一件商品，成本价是80元，售价是100元，老板先通知小李把商品价格增加50，小李有事耽搁了一个小时，正好一个小时后，老板觉得商品增加到150元，价格太高，可能会影响销量，又通知小王，你把商品价格降低30元。

此时，小李和小王同时操作商品后台系统，小李操作的时候，系统先取出商品价格100元，小王也在操作，取出的商品价格也是100元。小李将价格加了50元，并将150元存入了数据库；小王将商品减了30元，并将70元存入了数据库。如果没有锁，小李的操作完全被小王覆盖了。现在商品价格比成本价还要低10元

- 乐观锁

如果是乐观锁，小王保存价格前，会检查下价格是否被人修改过了。如果被修改过了，则重新取出的被修改后的价格150元，这样它会将120元存入数据库。

- 悲观锁

如果是悲观锁，小李取出数据后，小王只能等小李操作完成后，才能对价格进行操作，也会保证最终的价格是120元。



##### 乐观锁样例

- 添加乐观锁插件

```java
@Configuration
@MapperScan("com.mybatisplus.mybatisplus.dao")
public class MybatisPlusConfig {

    @Bean
    public MybatisPlusInterceptor mybatisPlusInterceptor(){
        MybatisPlusInterceptor mybatisPlusInterceptor=new MybatisPlusInterceptor();
        //添加分页插件
        mybatisPlusInterceptor.addInnerInterceptor(new PaginationInnerInterceptor(DbType.MYSQL));
        //添加乐观锁插件
        mybatisPlusInterceptor.addInnerInterceptor(new OptimisticLockerInnerInterceptor());
        return mybatisPlusInterceptor;
    }
}
```

- 在对应的实体类中指定版本号字段

```java
@TableName("commodity")
@Data
@AllArgsConstructor
@NoArgsConstructor
public class Commodity {
    @TableId
    private Long id;
    private String name;
    private Double price;

    @Version
    private Integer version; //乐观锁版本号
}
```

- 测试

```java
@Test
public void testOptimistic(){
    //原价100
    Commodity commodityHong = commodityDao.selectById(1);
    System.out.println("小红查询到的:"+commodityHong.getPrice());

    Commodity commodityLi = commodityDao.selectById(1);
    System.out.println("小李查询到的"+commodityLi.getPrice());
    //小红修改+50
    commodityHong.setPrice(commodityHong.getPrice()+50);
    commodityDao.updateById(commodityHong);
    //小李修改-30
    commodityLi.setPrice(commodityLi.getPrice()-30);
    int result = commodityDao.updateById(commodityLi);
    if(result==0){ //自旋一次
        Commodity commodity = commodityDao.selectById(1);
        commodity.setPrice(commodity.getPrice()-30);
        commodityDao.updateById(commodity);
    }
    //老板查询最后结果
    Commodity commodity = commodityDao.selectById(1);
    System.out.println("老板查询到的:"+commodity.getPrice()); //150
}
```



#### 通用枚举

- 创建枚举

```java
public enum SexEnum {
    MAN("男",1),
    WOMAN("女",0);

    @EnumValue //通过枚举获取的值
    private Integer sex; 
    private String sexName;

    SexEnum(String sexName, Integer sex) {
        this.sexName = sexName;
        this.sex = sex;
    }
}
```

- 测试

```java
@Test
public void testEnum(){
    Admin admin=new Admin();
    admin.setId(99);
    admin.setMessage("消息");
    admin.setPassword("123");
    admin.setName("小红");
    admin.setSexEnum(SexEnum.MAN);
    adminDao.insert(admin);
}
```

#### MybatisX插件

官网地址:https://baomidou.com/pages/ba5b24/#功能

- 安装MybatisX

安装方法：打开 IDEA，进入 File -> Settings -> Plugins -> marketplace，输入 `mybatisx` 搜索并安装。

- 导入依赖

```xml
<dependency>
    <groupId>mysql</groupId>
    <artifactId>mysql-connector-java</artifactId>
</dependency>
<!-- mybatis-plus -->
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
<dependency>
    <groupId>com.alibaba</groupId>
    <artifactId>druid</artifactId>
    <version>1.2.11</version>
</dependency>
<dependency>
    <groupId>org.projectlombok</groupId>
    <artifactId>lombok</artifactId>
</dependency>
```

- 在database中连接数据库，右键mybatisx-generator生成，选择需要的

- 除mybatis-plus基础的功能之外，还可以根据提示，自定义功能