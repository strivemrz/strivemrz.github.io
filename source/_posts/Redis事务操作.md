---
title: Redis事务操作
categories: Redis
tags:
  - Redis
excerpt: ''
cover: https://tse1-mm.cn.bing.net/th/id/R-C.40bbe995e5b4b15bf26959dd2492ef3d?rik=8%2bD0pOujE645wg&riu=http%3a%2f%2fwww.pp3.cn%2fuploads%2f1212qxn%2f476.jpg&ehk=na1tDwMwrncjRAg1A9ROG%2bkj4YZrklFwjUCYQl2c8yo%3d&risl=&pid=ImgRaw&r=0
---

### Redis基本的事务操作

1. **Redis单条命令是保证原子性的，但是Redis事务不保证原子性。**

2. 事务的四个特性(ACID)

   **原子性（Atomicity）**
   原子性是指事务是一个不可分割的工作单位，事务中的操作要么都发生，要么都不发生。

   **一致性（Consistency）**
   事务前后数据的完整性必须保持一致。

   **隔离性（Isolation）**
   事务的隔离性是多个用户并发访问数据库时，数据库为每一个用户开启的事务，不能被其他事务的操作数据所干扰，多个并发事务之间要相互隔离。

   **持久性（Durability）**
   持久性是指一个事务一旦被提交，它对数据库中数据的改变就是永久性的，接下来即使数据库发生故障也不应该对其有任何影响

3. Redis事务本质：一组命令的集合！一个事务中的所有命令都会被顺序化，在事务执行过程中，会按照顺序执行！

   ~~~bash
   -----------队列 set set set 执行-----------
   ~~~

   Redis事务没有隔离级别的概念！

   所有的命令在事务中，并没有直接被执行！只有发起执行命令的时候才会执行！Exec

   Redis单条命令是保证原子性的，但是Redis事务不保证原子性。

   Redis的事务：

   - 开启事务(multi)

   - 命令入队

   - 执行事务(exec)

~~~bash
127.0.0.1:6379> multi
OK
127.0.0.1:6379(TX)> flushdb
QUEUED
127.0.0.1:6379(TX)> set name username
QUEUED
127.0.0.1:6379(TX)> get name
QUEUED
127.0.0.1:6379(TX)> exec
1) OK
2) OK
3) "username"
###中途取消事务
127.0.0.1:6379> multi
OK
127.0.0.1:6379(TX)> set k1 v1
QUEUED
127.0.0.1:6379(TX)> DISCARD ##取消事务
OK
127.0.0.1:6379> get k1
(nil)

~~~

代码编译型异常(代码有问题！命令有错！)，事务中所有的命令都不会被执行。

运行时异常(1/0)，如果事务队列中存在语法性，那么执行命令的时候，其他命令是可以执行的，错误命令抛出异常。

~~~bash
127.0.0.1:6379> set k1 hel
OK
127.0.0.1:6379> multi
OK
127.0.0.1:6379(TX)> incr k1
QUEUED
127.0.0.1:6379(TX)> get k1
QUEUED
127.0.0.1:6379(TX)> set k2 v2
QUEUED
127.0.0.1:6379(TX)> exec
1) (error) ERR value is not an integer or out of range
2) "hel"
3) OK
~~~

Redis监控

悲观锁：

- 很悲观，认为什么时候都会出现问题，无论做什么都会加锁！

乐观锁：

- 很乐观，认为什么时候都不会出问题，所以不会上锁，更新数据的时候去判断一下，在此期间是否有人修改过这个数据。Redis使用watch实现监听数据(CAS机制)
- CAS是项乐观锁技术，当多个线程尝试使用CAS同时更新同一个变量时，只有其中一个线程能更新变量的值，而其它线程都失败，失败的线程并不会被挂起，而是被告知这次竞争中失败，并可以再次尝试。
- 获取version
- 更新的时候比较version

~~~bash
127.0.0.1:6379> set money 100
OK
127.0.0.1:6379> set out 0
OK
127.0.0.1:6379> watch money ##监视money对象
OK
127.0.0.1:6379> multi##如果在此期间其他进程修改了money，在exec的时候发现money与之前监听的不一致，事务取消
OK
127.0.0.1:6379(TX)> INCRBY out 10
QUEUED
127.0.0.1:6379(TX)> DECRBY money 10
QUEUED
127.0.0.1:6379(TX)> exec##执行exec会自动释放money锁
1) (integer) 10
2) (integer) 90
##当事务取消之后还可以利用unwatch取消监视，然后再次执行上面操作
127.0.0.1:6379> unwatch##如果发现事务执行失败，就先解锁
OK
127.0.0.1:6379> watch money ##获取最新的值，再监视 select version
OK
127.0.0.1:6379> multi
OK
127.0.0.1:6379(TX)> INCRBY out 10
QUEUED
127.0.0.1:6379(TX)> DECRBY money 10
QUEUED
127.0.0.1:6379(TX)> exec##比对监视的值是否发生变化，如果没有变化，可以执行成功，如果变了执行失败
1) (integer) 20
2) (integer) 80

~~~
