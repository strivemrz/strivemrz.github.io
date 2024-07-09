---
updated: 2022-06-19
title: 安装新版mysql
categories: mysql
tags:
  - mysql

excerpt: ''

cover: https://www.gqpic.cn/uploadfile/51/12/51124cd8718a26e6.jpg

---


## mysql安装新版本
1.备份旧版中的数据
- 在cmd进入mysql bin目录
~~~bash
mysqldump --all-database > D:\all_database.sql -u root -p
~~~
- 输入密码

2.删除旧版mysql

- 停止mysql服务
cmd进入services.msc
- 控制面饭删除旧版mysql
- 进入旧版安装目录确保删除干净

3.下载安装新版(这里是msi路径)
- https://dev.mysql.com/downloads/mysql/

![mysql2.png](/2022/06/19/安装新版mysql/mysql2.png)

![mysql3.png](/2022/06/19/安装新版mysql/mysql3.png)

4.安装mysql

- 这里我选择的是第二个(剩下的直接next)
![mysql1.png](/2022/06/19/安装新版mysql/mysql1.png)


5.写入备份的所有数据
- cmd进入新版mysql bin目录
- mysql -u root -p回车 输入密码，回车，进到mysql>状态下。
- source D:\all_database.sql 导入成功

6.查看mysql版本
~~~bash
select version();
~~~