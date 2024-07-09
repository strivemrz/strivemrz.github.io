---
title: url参数含义
categories: mysql
tags:
  - mysql
excerpt: ''
cover: https://desk-fd.zol-img.com.cn/t_s960x600c5/g2/M00/06/0D/ChMlWl5Au_GIVa-8AAbJ_1FhEJgAANQ-AIbLyoABsoX611.jpg

---


~~~properties
url=jdbc:mysql://localhost:3306/test?serverTimezone=GMT%2B8&useUnicode=true&characterEncoding=utf-8&useAffectedRows=true&allowMultiQueries=true&useSSL=false&&zeroDateTimeBehavior=convertToNull
~~~

- mysql：数据库是mysql

- localhost:3306 ：数据库服务器地址，端口号是3306

- test：数据库名称
- serverTimezone

> 如果你设置serverTimezone=UTC，连接不报错，
> 但是我们在用java代码插入到数据库时间的时候却出现了问题。
> 比如在java代码里面插入的时间为：2021-06-24 17:29:56
> 但是在数据库里面显示的时间却为：2021-06-24 09:29:56
> 有了8个小时的时差
> UTC代表的是全球标准时间 ，但是我们使用的时间是北京时区也就是东八区，领先UTC八个小时。
>
> //北京时间==东八区时间！=北京当地时间
> serverTimezone=GMT%2B8
> //或者使用上海时间
> serverTimezone=Asia/Shanghai

- useUnicode：是否使用useUnicode字符集，如果参数characterEncoding为gbk、gb2312或utf-8时，必须将其设置成true

- characterEncoding：数据库字符编码格式
- useAffectedRows

> useAffectedRows的作用在于是否用受影响的行数替代查找到的行数来返回数据，默认是false，常用在update

~~~sql
update task_info SET task_status=2 where id=? and task_status=0
- 没使用之前返回1，使用之后返回0
~~~

- allowMultiQueries：允许多条sql同时执行。
- useSSL：mysql高版本需要指明是否进行SSL连接。

~~~makefile
SSL协议提供服务主要： 		
1）认证用户服务器，确保数据发送到正确的服务器； 　　 .
2）加密数据，防止数据传输途中被窃取使用；
3）维护数据完整性，验证数据在传输过程中是否丢失；
~~~

- zeroDateTimeBehavior(可以不用设置)：数据库的某个字段(一般是时间字段)的类型是timestamp，假设某条记录的这个字段的值是0，java连接mysql数据库，操作该字段时，默认会抛出一个异常，即

~~~basic
java.sql.SQLException: Cannot convert value '0000-00-00 00:00:00' from column 7 to TIMESTAMP
~~~

​				zeroDateTimeBehavior，该属性专门用来处理这类问题。该属性有三个选项值：

​				1）exception ,默认值，即抛出SQL connot covert value...的异常

​				2）converToNull ，将日期值(为0的值) 替换成最近的日期，0001:01:01