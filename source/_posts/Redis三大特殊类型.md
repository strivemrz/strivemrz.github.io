---
title: Redis三大特殊类型
categories: Redis
tags:
  - Redis
excerpt: ''
cover: https://tse1-mm.cn.bing.net/th/id/R-C.519af1c754eef9c4fd575a969d48892c?rik=KBPrq2rCn96vWg&riu=http%3a%2f%2fdl.ppt123.net%2fpptbj%2f51%2f20181115%2f1fr03hw3vuj.jpg&ehk=SPHrJMz0MyhBOfgKsjJCAy8oAPgFbvpSJK1spZ%2fsp3g%3d&risl=&pid=ImgRaw&r=0
---
### Geospatial地理位置详解

geo底层的实现原理就是zset

~~~bash
127.0.0.1:6379> geoadd chine:city 116 39 beijin#geospatial数据添加
(integer) 1
127.0.0.1:6379> geoadd chine:city 121 31 shanghai
(integer) 1
127.0.0.1:6379> geopos chine:city beijin#geospatial数据获取
1) 1) "116.00000113248825073"
   2) "38.99999918434559731"
127.0.0.1:6379> geodist chine:city beijin shanghai km#geospatial两地数据之间的直线距离
"999.2077"
127.0.0.1:6379> GEORADIUS chine:city 110 38 1000 km#附近的人，获取key中 110 38 位置半径1000km的人
1) "beijin"
127.0.0.1:6379> georadius chine:city 110 38 1000 km withdist#widthdist显示距离
1) 1) "beijin"
   2) "533.8827"
127.0.0.1:6379> georadius chine:city 110 38 1000 km withcoord asc#withcoord显示经纬度，asc升序
1) 1) "beijin"
   2) 1) "116.00000113248825073"
      2) "38.99999918434559731"
127.0.0.1:6379> GEORADIUSBYMEMBER chine:city beijin 1000 km#成员为中心点进行查找
1) "beijin"
2) "shanghai"
127.0.0.1:6379> zrem chine:city beijin#删除georadius元素
(integer) 1
127.0.0.1:6379> zrange chine:city 0 -1#查找所有georadius元素
1) "shanghai"

~~~

### Hyperloglog

Redis Hyperloglog基于统计的算法！

**网页的UV（一个人访问一个网页多次，但是还是算作一个人！）**

传统的方式，set保存用户的id，然后就可以统计set中的元素作为标准判断！

这个方式如果保存了大量的用户id，就会比较麻烦！我们的目的是为了计数，而不是保存用户id；

0.81%错误率！统计UV任务，可以忽略不记！

如果允许容错，那么一定可以使用hyperloglog

如果不允许容错，就使用set或者自己的数据类型即可！

~~~bash
127.0.0.1:6379> pfadd mykey a b c d e f f#添加hyperloglog元素
(integer) 1
127.0.0.1:6379> pfcount mykey#统计hyperloglog数量
(integer) 6
127.0.0.1:6379> pfadd mykey2 f g h i
(integer) 1
127.0.0.1:6379> pfcount mykey2
(integer) 4
127.0.0.1:6379> pfmerge mykey3 mykey mykey2#将mykey和mykey2中的元素拼到mykey中
OK
127.0.0.1:6379> pfcount mykey3
(integer) 9
~~~

### Bitmaps

位存储

统计用户信息，活跃，不活跃，登录，未登录，打卡，365打卡！两个状态的都可以使用Bitmaps！

Bitmaps位图，数据结构！都是操作二进制位来进行记录，就只有0和1两个状态。

~~~bash
127.0.0.1:6379> setbit sign 0 1#存储Bitmaps类型数据，位存储
(integer) 0
127.0.0.1:6379> setbit sign 1 0
(integer) 0
127.0.0.1:6379> setbit sign 1 0
(integer) 0
127.0.0.1:6379> setbit sign 1 1
(integer) 0
127.0.0.1:6379> setbit sign 1 0
(integer) 1
127.0.0.1:6379> setbit sign 2 0
(integer) 0
127.0.0.1:6379> setbit sign 31
(error) ERR wrong number of arguments for 'setbit' command
127.0.0.1:6379> setbit sign 3 1
(integer) 0
127.0.0.1:6379> setbit sign 4 1
(integer) 0
127.0.0.1:6379> setbit sign 5 1
(integer) 0
127.0.0.1:6379> setbit sign 6 1
(integer) 0
###查看某一天是否打卡
127.0.0.1:6379> getbit sign 3
(integer) 1
###统计操作，统计打卡的天数！
127.0.0.1:6379> bitcount sign
(integer) 5
~~~