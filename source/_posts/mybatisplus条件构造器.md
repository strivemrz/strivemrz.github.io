---
title: Wrapper条件构造器
categories: mybatis-plus
tags:
  - mybatis-plus
excerpt: ''
cover: https://img.tt98.com/d/2020/2020060113003303/5ed4754cee5fe.jpg
---

#### wrapper条件构造器

![wrapper](/2022/09/05/mybatisplus条件构造器/wrapper接口.png)

##### 组装查询条件

```java
@Test
public void testWrapper(){
    //查询message中包含"新增"，并且uid>3的数据
    QueryWrapper<Admin> wrapper=new QueryWrapper<>();
    wrapper.like("message","新增")
                    .ge("uid",3);
    List<Admin> list = adminDao.selectList(wrapper);
    list.forEach(result->{
        System.out.println("条件构造器:"+result);
    });
}
```

##### 组装排序条件

```java
@Test
public void testSelectOrder(){
    QueryWrapper<Admin> queryWrapper=new QueryWrapper<>();
    queryWrapper.orderByDesc("uid");

    adminDao.selectList(queryWrapper);
}
```

##### 组装删除条件

```java
@Test
public void testDeleteWrapper(){
    QueryWrapper<Admin> queryWrapper=new QueryWrapper<>();
    queryWrapper.isNull("message");
    int result = adminDao.delete(queryWrapper);
    System.out.println("受影响行数:"+result);
}
```

##### 实现修改功能

```java
@Test
public void testUpdateWrapper(){
    QueryWrapper<Admin> queryWrapper=new QueryWrapper<>();
    //update admin set message="实现修改功能" where user_name like '%z%' and password='456' or uid ='6'
    //and 优先级要比 or高
    queryWrapper.like("user_name","z").
                    eq("password","456").
                    or().
                    eq("uid",6);
    Admin admin=new Admin();
    admin.setMessage("实现修改功能");
    adminDao.update(admin,queryWrapper);
}
```

##### 条件优先级

```java
@Test
public void testConditionPrefix(){
    //update admin set is_deleted =1 where user_name = 'aliyun' and (message='条件优先级' or password like '%8%')
    QueryWrapper<Admin> queryWrapper=new QueryWrapper<>();
    queryWrapper.eq("user_name","aliyun")
                    .and(i->i.eq("message","条件优先级").or().like("password","8"));
    Admin admin=new Admin();
    admin.setIsDeleted("1");
    int updateResu = adminDao.update(admin,queryWrapper);//第一个参数修改的字段，第二个参数条件
    System.out.println("受影响行数:"+updateResu);
}
```

##### 组装select语句

```java
@Test
public void testSplice(){
    //select uid,user_name from admin
    QueryWrapper<Admin> adminQueryWrapper=new QueryWrapper<>();
    adminQueryWrapper.select("uid","user_name");
    List<Map<String, Object>> maps = adminDao.selectMaps(adminQueryWrapper);
    maps.forEach(item->{
        System.out.println("查询结果:"+item);
    });
}
```

##### 组装子查询

```java
@Test
public void testSelectChild(){
    //select * from t_admin where password in (select password from t_admin where uid = 7)
    QueryWrapper<Admin> queryWrapper=new QueryWrapper<>();
    queryWrapper.inSql("password","select password from t_admin where uid=7");
    List<Admin> result = adminDao.selectList(queryWrapper);
    result.forEach(System.out::println);
}
```

##### UpdateWrapper实现修改功能

> QueryWrapper 设置查询条件
>
> UpdateWrapper 既可以设置查询条件，也可以设置实体类字段值

```java
@Test
public void testUpdateWrapper2(){
    //update admin set message = "updateWrapper实现修改功能" where user_name = 'user4'
    UpdateWrapper<Admin> updateWrapper=new UpdateWrapper<>();
    updateWrapper.eq("user_name","user4")
                    .set("message","updateWrapper实现修改功能");
    int update = adminDao.update(null, updateWrapper);
    System.out.println("影响行数:"+update);
}
```

##### 根据字段值是否为空组装条件

```java
@Test
public void testSpliceByCloumn(){
    //select * from admin where uid >=6;
    QueryWrapper<Admin> queryWrapper=new QueryWrapper<>();
    Admin admin=new Admin();
    admin.setId(6);
    if(!Objects.isNull(admin.getId())){
        queryWrapper.ge("uid",admin.getId());
    }
    List<Admin> list = adminDao.selectList(queryWrapper);
    list.forEach(System.out::println);
}
```

##### 使用condition组装条件

```java
@Test
    public void testSpliceByCondition(){
        Admin admin=new Admin();
        admin.setId(6);
        admin.setMessage("condition组装条件");
        QueryWrapper<Admin> queryWrapper=new QueryWrapper<>();
        queryWrapper.like(StringUtils.isNotBlank(admin.getMessage()),"message",admin.getMessage())
                        .eq(admin.getId()!=null,"uid",admin.getId());
        List<Admin> list = adminDao.selectList(queryWrapper);
        list.forEach(System.out::println);
    }
```

##### LambdaQueryWrapper条件构造器

```java
@Test
public void testLambdaQueryWrapper(){
    Admin admin=new Admin();
    admin.setId(7);
    LambdaQueryWrapper<Admin> lambdaQueryWrapper=new LambdaQueryWrapper<>();
    //和queryWrapper唯一区别就是中间字段名使用lambda形式，防止字段名写错
    //SELECT uid AS id,user_name AS name,password,message,is_deleted FROM admin WHERE (uid = ?)
    lambdaQueryWrapper.eq(admin.getId()!=null,Admin::getId,admin.getId());
    List<Admin> list = adminDao.selectList(lambdaQueryWrapper);
    list.forEach(System.out::println);
}
```

##### LambdaUpdateWrapper条件构造器

```java
@Test
public void testLambdaUpdateWrapper(){
    LambdaUpdateWrapper<Admin> updateWrapper=new LambdaUpdateWrapper<>();
    //和LambdaQueryWrapper一样，凡是涉及到数据库字段名的地方，都用lambda形式表示，防止写错
    updateWrapper.eq(Admin::getName,"user4")
            .set(Admin::getMessage,"updateWrapper实现修改功能");
    int update = adminDao.update(null, updateWrapper);
    System.out.println("影响行数:"+update);
}
```