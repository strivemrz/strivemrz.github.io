---
title: Redis五大基本类型
categories: Redis
tags:
  - Redis
excerpt: ''
cover: https://tse1-mm.cn.bing.net/th/id/R-C.d566e6584ef9b1f8cbc4b7ac4931e805?rik=pOQ9h19W8W5mGw&riu=http%3a%2f%2fimg.mm4000.com%2ffile%2f9%2f0f%2fa2d66bfebd.jpg&ehk=BcC5fnD2Qd%2fkuzlrRO3HwYQwjRUbzckLjusiB9g%2fx0E%3d&risl=&pid=ImgRaw&r=0
---



### **Redis**

*远程字典服务*

1. 非关系型数据库(NoSQL)

   不仅仅是sql

   在web2.0时代之前的关系型数据库难以应付高并发

   nosql是一种细粒度的缓存，性能会比较高

   数据类型是多样型的（不需要事先设计数据库，随取随用）

2. Map<String,String> map;使用键值对来控制

### Redis能干啥？

1. 内存存储，持久化，内存中是断电即失

### 安装Redis

1. 下载redis(若版本较高，安装到linux需要升级gcc)

   升级gcc

   sudo yum install centos-release-scl

   sudo yum install devtoolset-7-gcc*

   scl enable devtoolset-7 bash

   这种只是临时的退出linux之后还是原来的gcc版本

   echo "source /opt/rh/{对应的redis文件}/enable" >>/etc/profile

2. 利用上传工具上传到linux，这里用的是Xftp

3. 将redis压缩包移到/opt目录下

   mv redis文件 目标地址

   cd 目标地址

4. 解压redis压缩包

   tar -zxvf redis文件

5. 进入redis文件内部

6. 安装c++环境(redis是基于c++的)

   yum install gcc-c++

7. make(编译)

   把需要的文件全部配上

8. make install(安装)

9. 安装目录(cd /usr/local/bin)

   ![image-20220511163649719](C:\Users\19651\AppData\Roaming\Typora\typora-user-images\image-20220511163649719.png)

10. 将redis配置文件，复制到当前目录下

    mkdir zconfig

    cp /opt/redis-7.0.0/redis.conf zconfig

    ![image-20220511164320523](C:\Users\19651\AppData\Roaming\Typora\typora-user-images\image-20220511164320523.png)

11. redis默认不是后台启动的

    进入redis.conf

    ![image-20220511164630556](C:\Users\19651\AppData\Roaming\Typora\typora-user-images\image-20220511164630556.png)

12. 运行redis

    redis-server zconfig/redis.conf

    redis-cli -p 6379(使用redis客户端连接)

    ![image-20220511165132959](C:\Users\19651\AppData\Roaming\Typora\typora-user-images\image-20220511165132959.png)

    ping(查看是否接通)

13. 退出redis

    shutdown

    exit

### Redis工具

1. redis-benchmark是一个压力测试工具

   redis-benchmark -h localhost -p 6379 -c 50 -n 10000

   | 序号 | 选项 | 描述                                    | 默认值    |
   | :--- | :--- | :-------------------------------------- | :-------- |
   | 1    | -h   | 指定redis server 主机名                 | localhost |
   | 2    | -p   | 指定redis 服务端口                      | 6379      |
   | 3    | -s   | 指定服务器socket                        |           |
   | 4    | -c   | 指定并发连接数                          | 50        |
   | 5    | -n   | 指定请求数                              | 10000     |
   | 6    | -d   | 以字节形式指定SET/GET值的数值大小       | 2         |
   | 7    | -k   | 1=keepalive 0=reconnect                 | 1         |
   | 8    | -r   | SET/GET/INCR使用随机key, SADD使用随机值 |           |
   | 9    | -P   | 通过管道传输<numreq>请求                | 1         |
   | 10   | -q   | 强制退出redis.仅显示query/sec值         |           |
   | 11   | -csv | 以csv格式输出                           |           |
   | 12   | -l   | 生成循环 永久执行测试                   |           |
   | 13   | -t   | 仅运行以逗号分隔的测试命令列表          |           |
   | 14   | -I   | Idle模式,仅打开N个idle连接并等待        |           |

   ![image-20220511175118539](C:\Users\19651\AppData\Roaming\Typora\typora-user-images\image-20220511175118539.png)

### Redis基本知识说明

1. redis默认有16个数据库

   默认使用的是第0个

   select num 切换数据库
~~~bash
   127.0.0.1:6379> select 0 #切换数据库0-15
   OK
   127.0.0.1:6379> dbsize #当前数据库大小
   (integer) 5
   127.0.0.1:6379> keys * #查看当前数据库所有的键值
   1) "myhash"
   2) "name"
   3) "counter:__rand_int__"
   4) "key:__rand_int__"
   5) "mylist"
   127.0.0.1:6379> flushdb #清空当前数据库
   127.0.0.1:6379[1]> flushall #清空所有数据库
   127.0.0.1:6379[1]> exists key #查看key是否存在
   127.0.0.1:6379> move name 1 #移动数据到第几个数据库
   127.0.0.1:6379> expire he 10 #设置过期时间为10秒
   .1:6379> ttl he #查看元素还剩多长时间过期
   127.0.0.1:6379> del he #删除当前元素
   127.0.0.1:6379> type na #查看当前元素什么类型
~~~

3. redis是单线程的

   Redis是很快的，官方表示，Redis是基于内存操作，CPU不是Redis性能瓶颈，Redis的瓶颈是根据机器的内存和网络带宽，既然可以使用单线程来实现，就使用单线程了

   cpu>内存>硬盘

   多线程(cpu上下文会切换) 不一定会比单线程效率高

   核心：redis是将所有的数据全部放在内存中的，所有说使用单线程去操作效率就是最高的，多线程(cpu会上下文切换，耗时的操作)对于内存系统来说，如果没有上下文切换效率就是最高的，每次读写都是在一个cpu上的，在内存情况下，这个就是最佳方案

4. Redis五大数据类型

   Redis是一个开源的，内存中的数据结构存储系统，它可以用作**数据库**，**缓存**和**消息中间件MQ**

   Redis-key

   String

   List

   Set

   Hash

   Zset

5. 三种基本数据类型

   geospatial

   hyperloglog

   bitmaps

### String字符串类型详解

~~~bash
127.0.0.1:6379> dbsize
(integer) 1
127.0.0.1:6379> keys *
1) "na"
127.0.0.1:6379> append na whe # 往na后添加字符串，如果字符串不存在就相当于set key
127.0.0.1:6379> strlen na #获取na的长度
127.0.0.1:6379> set num 1
OK
127.0.0.1:6379> type num
string
127.0.0.1:6379> incr num #一次增加一个
(integer) 2
127.0.0.1:6379> incrby num 10 #一次增加n个
(integer) 12
127.0.0.1:6379> get num
"12"
127.0.0.1:6379> decr num #一次减少一个
(integer) 11
127.0.0.1:6379> decrby num 10 #一次减少n个
(integer) 1
127.0.0.1:6379> getrange title 0 2 #截取字符串显示
"hel"
127.0.0.1:6379> getrange title 0 -1 #全部输出
"hello,world"
127.0.0.1:6379> setrange title 1 push #替换，相当于replease
(integer) 11
127.0.0.1:6379> get title
"hpush,world"
###################################################
127.0.0.1:6379> setex title2 10 hello,world #10秒后过期
OK
127.0.0.1:6379> ttl title2
(integer) 4
127.0.0.1:6379> setnx title2 jelly #如果key不存在添加成功，存在添加失败(常用在分布式锁中)
(integer) 0
127.0.0.1:6379> mset k1 v1 k2 v2 k3 v4#同时设置多个值
OK
127.0.0.1:6379> mget k1 k2 k3#同时获取多个值
1) "v1"
2) "v2"
3) "v4"
127.0.0.1:6379> msetnx k5 v5 k7 v7#不存在时设置成功，原子操作
(integer) 1
##对象
127.0.0.1:6379> set user:1 {name:zs,age:15} #获取key之后类似于json数据
OK
127.0.0.1:6379> mset user:1:name zs user:1:age 15
127.0.0.1:6379> getset user:1:name lb#将给定key的值设置为value并返回旧值
"zs"
~~~

### List类型详解

~~~bash
#头插，顺序是反的
127.0.0.1:6379> lpush list list1
(integer) 1
127.0.0.1:6379> lpush list list2
(integer) 2
127.0.0.1:6379> lrange list 0 -1
1) "list2"
2) "list1"
127.0.0.1:6379> lrange list 0 0
1) "list2"

#尾插，顺序是正的
127.0.0.1:6379> rpush list list1
(integer) 1
127.0.0.1:6379> rpush list list2
(integer) 2
127.0.0.1:6379> rpush list list3
(integer) 3
127.0.0.1:6379> lrange list 0 -1
1) "list1"
2) "list2"
3) "list3"
127.0.0.1:6379> lpop list#左边移除
"list1"
127.0.0.1:6379> rpop list#右边移除
"list3"
127.0.0.1:6379> lindex list 0 #从左边获取下标为0的元素
"one"
127.0.0.1:6379> llen list #获取list长度
(integer) 3
127.0.0.1:6379> lrem list 1 three#删除一个list集合中的 three元素
(integer) 1
127.0.0.1:6379> ltrim list 1 2#截取list中间的元素，两边删除
OK
127.0.0.1:6379> lrange list 0 -1
1) "two"
2) "three"
127.0.0.1:6379> rpoplpush list newlist#删除最后一个数据并添加到newlist的第一个数据
"three"
127.0.0.1:6379> lset list 0 hello #替换list中第0个元素为hello。如果不存在就会报错
OK
127.0.0.1:6379> linsert list before two one#在list集合中two的前面插入one元素
(integer) 4

~~~

### Set类型详解

~~~bash
127.0.0.1:6379> sadd myset hello#往set类型的myset中添加元素
(integer) 1
127.0.0.1:6379> sadd myset ahui
(integer) 1
127.0.0.1:6379> smembers myset#查询所有元素
1) "ahui"
2) "hello"
127.0.0.1:6379> sismember myset ahui#查询set类型的集合中是否存在该值
(integer) 1
127.0.0.1:6379> scard myset#查询myset集合中有几个元素
(integer) 2
127.0.0.1:6379> srem myset hello#删除myset集合中的hello元素
(integer) 1
127.0.0.1:6379> srandmember myset#获取myset中随机的元素
"hello
127.0.0.1:6379> spop myset#随机删除元素
"ahui"
127.0.0.1:6379> smove myset myset2 hello#移动元素，将myset中的hello元素移到myset2中
(integer) 1
127.0.0.1:6379> sdiff set1 set2#查找set1中set2没有的元素
1) "b"
2) "a"
127.0.0.1:6379> sunion set1 set2#查找set1和set2的并集
1) "a"
2) "c"
3) "b"
4) "d"
5) "e"
127.0.0.1:6379> sinter set1 set2#查找set1和set2的交集
1) "c"

~~~

### Hash(哈希)

Map集合，key-Map形式

类似于String类型

hash变更的数据user name，age，尤其是用户信息之类，经常变动的信息！hash更适合于对象的存储，String跟适合字符的存储

~~~bash
127.0.0.1:6379> hset myhash f1 ahui f2 v2#设置hash值Map形式
(integer) 2
127.0.0.1:6379> hget myhash f1#获取值
"ahui"
127.0.0.1:6379> hmget myhash f1 f2#获取多个值
1) "ahui"
2) "v2"
127.0.0.1:6379> hgetall myhash#获取所有的值
1) "f1"
2) "ahui"
3) "f2"
4) "v2"
127.0.0.1:6379> hdel myhash f1#删除hash值
(integer) 1
127.0.0.1:6379> hgetall myhash
1) "f2"
2) "v2"
127.0.0.1:6379> hlen myhash#获取有多少个值
(integer) 1
127.0.0.1:6379> hkeys myhash#获取所有的key
1) "f2"
127.0.0.1:6379> hvals myhash#获取所有的值
1) "v2"
127.0.0.1:6379> hincrby mybash f3 1#增加1
(integer) 6
127.0.0.1:6379> hincrby mybash f3 -1#减1
(integer) 5
127.0.0.1:6379> hsetnx mybash f3 he#如果不存在时新增
(integer) 0

~~~

### Zset有序集合

ZRANGEBYSCORE是按照score排列

zrange是按照索引排列

[zrange和zrangebyscore区别](https://blog.csdn.net/jfqqqqq/article/details/102858508?ops_request_misc=%257B%2522request%255Fid%2522%253A%2522165236460416781685342181%2522%252C%2522scm%2522%253A%252220140713.130102334..%2522%257D&request_id=165236460416781685342181&biz_id=0&utm_medium=distribute.pc_search_result.none-task-blog-2~all~sobaiduend~default-2-102858508-null-null.142^v9^pc_search_result_control_group,157^v4^control&utm_term=zrange%E5%92%8Czrangebyscore%E5%8C%BA%E5%88%AB&spm=1018.2226.3001.4187)

~~~bash
127.0.0.1:6379> zadd myset 1 one#添加一个元素中间标志位起到排序作用
(integer) 1
127.0.0.1:6379> zadd myset 2 two
(integer) 1
127.0.0.1:6379> zadd myset 3 three
(integer) 1
127.0.0.1:6379> zrange myset 0 -1#查询所有元素
1) "one"
2) "two"
3) "three"
#ZRANGEBYSCORE myset 最小值 最大值
127.0.0.1:6379> ZRANGEBYSCORE myset -inf +inf#根据score排序
1) "one"
2) "two"
3) "three"
4) "four"
5) "five"
127.0.0.1:6379> zrem myset two#删除zset集合中的two元素
(integer) 1
127.0.0.1:6379> zcard myset#查看zset集合中有多少元素
(integer) 4
127.0.0.1:6379> ZREVRANGE myset 0 -1#降序排列
1) "five"
2) "four"
3) "three"
4) "one"
127.0.0.1:6379> zcount myset 4 6#获取指定区间的成员数量
(integer) 3

~~~

