---
title: sql常用语句
categories: mysql
tags:
  - mysql

excerpt: ''

cover: https://ts1.cn.mm.bing.net/th/id/R-C.b4d9f7a27240c1ae58f40969493088f6?rik=QiDjrXnvYUVHDA&riu=http%3a%2f%2fup.jpgjpg.com%2fpic%2fbb%2f23%2fb9%2fbb23b90225b42f539fd0f95cf965baa7.jpg&ehk=O7Z2uWfbzMXohEdUOEhW6eC77JC%2f6cs%2bacGNsQHXqC0%3d&risl=&pid=ImgRaw&r=0

---

#### **模糊查询**

- _：表示一个字符

~~~sql
select username from admin where username like '____';
#列出username字段有4个字符的记录
~~~

- %：表示0~n个字符

~~~sql
select username from admin where username like '%a%';
#列出username的值含字符a的记录
~~~

#### **正则表达式**

- 用法

where 字段名 regexp '正则表达式';

正则元字符 ^ $ . [] * |

- 列出以a开头或以s开头的记录

~~~sql
select username from admin where username regexp '^k|^a';
~~~

~~~sql
select username from admin where username regexp '^[at]';
~~~

- 列出字段username的值含有字符a或t的记录

~~~sql
select username from admin where username regexp '[at]';
~~~

- 列出username的值含有数字的记录

~~~sql
select username from admin where username regexp '[0-9]';
~~~

#### **聚集函数**

> mysql服务内置的对数据做统计的命令

- ave()   //统计字段平均值
- sum()  //统计字段之和
- min()  //统计字段最小值
- max()  //统计字段最大值
- count() //统计字段值个数

~~~sql
select min(age) from admin where age>=10 and age <=20
~~~

#### **查询结果分组**

~~~sql
select username from admin group by username;//查询admin中所有username的种类
~~~

~~~sql
select * from admin where age >=10 group by username //查询admin中age>=10的数据并以 username进行分组
~~~

#### **查询结果过滤**

~~~sql
select * from admin where username is not null having age>=18;//查询admin中username不为空的数据，并且过滤出age>=18的
~~~

#### **限制插寻结果显示行数**

~~~sql
select * from admin where age >=18 and age <=28 limit 0,3;//查询admin中age>=18 and age <=28 的从第一行开始的3条数据
~~~