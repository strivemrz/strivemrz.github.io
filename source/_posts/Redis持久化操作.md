---
title: Redis持久化操作
categories: Redis
tags:
  - Redis
excerpt: ''
cover: https://tse4-mm.cn.bing.net/th/id/OIP-C.qVW8uwdLRCEk8Shb4mZIOgHaEo?pid=ImgDet&rs=1
---

### 持久化

redis是内存数据库，如果不将内存中的数据库状态保存到磁盘，那么一旦服务器进程退出，服务器中的数据库状态也会消失，所以Redis提供了持久化功能！

### 持久化之RDB操作(默认,一般情况不修改配置)

在指定的时间间隔内将内存中的数据集快照写入磁盘，也就是Snapshot快照，它恢复时是将快照文件直接读到内存中。

Redis会单独创建(fork)一个子进程来进行持久化，会先将数据写入到一个临时文件中，待持久化过程都结束了，再用这个临时文件替换上次持久化好的文件。整个过程中，主进程是不进行任何IO操作的。这就确保了极高的性能。如果需要进行大规模数据的恢复，且对于数据恢复的完整性·不是非常敏感，那RDB方式要比AOF方式更加的高效。

> 缺点:RDB的缺点是最后一次持久化后的数据可能丢失。



rdb保存的文件是dump.rdb

**rdb触发机制**

- save的规则满足的情况下，会自动触发rdb规则(必须等到规定时间才会检查有几次修改key操作，如果达到规定值，才会触发)
- 执行flushall命令，也会触发rdb规则
- 退出redis也会产生rdb文件(shutdown)



**恢复rdb文件**

- 将rdb文件放在redis启动目录就可以，redis启动的时候会自动检查dump.rdb恢复其中的数据

- 查看需要存在的位置

  ~~~bash
  127.0.0.1:6379> config get dir
  1) "dir"
  2) "/usr/local/bin"
  ~~~

优点：

1、适合大规模的数据恢复

2、对数据的完整性要求不高

缺点：

1、需要一定的时间间隔进程操作！如果redis意外宕机了，这个最后一次修改数据就没有了

2、fork进程的时候会占用一定的内存空间



### 持久化之AOF(Append Only File)操作

~~~bash
auto-aof-rewrite-percentage 100
auto-aof-rewrite-min-size 64mb
~~~

重写规则：默认无限追加，文件会越来越大。可以使用auto-aof-rewrite-min-size 64mb来判断如果文件大于64mb另写一个新文件

默认是不开启的，我们需要手动进行配置！我们只需要将appendonly改为yes就开启了aof！

重启，redis就可以生效了

如果这个aof文件有错，这时候redis是启动不起来的，需要修复这个aof文件，使用redis-check-aof --fix

~~~bash
appendonly no # 默认是不开期aof模式的，默认使用rdb方式持久化的，在大部分的情况下，rdb完全够用
appendfilename "appendonly.aof" # 持久化的文件的名字

# appendfsync always # 每次修改都会 sync 消耗性能
appendfsync everysec # 每秒执行一次sync，可能会丢失这ls的数据
# appendfsync no # 不执行sync 这个时候操作系统自己同步数据，速度最快！
~~~

优点：

1、每一次修改都同步，文件的完整会更好

2、每秒同步一次，可能会丢失一秒的数据

3、从不同步，效率是最高的！

缺点：

1、相对于数据文件来说，aof远远大于rdb，修复的速度也比rdb慢！

2、Aof运行效率也要比rdb慢，所以我们redis默认的配置就是rdb持久化。

