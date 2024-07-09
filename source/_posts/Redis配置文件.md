---
title: Redis配置文件
categories: Redis
tags:
  - Redis
excerpt: ''
cover: https://desk-fd.zol-img.com.cn/t_s960x600c5/g5/M00/02/00/ChMkJlbKw4GIDE9zAAn4DWid894AALG4AMofSkACfgl522.jpg
---

### Redis配置文件

1. 单位(对大小写不敏感,1GB和1gb是一样的)

   ![img](https://kuangstudy.oss-cn-beijing.aliyuncs.com/bbs/2021/01/06/kuangstudy426aece4-d812-4090-90fa-e06d0a41ceb7.png)

2. 文件包含(include)

   ![img](https://kuangstudy.oss-cn-beijing.aliyuncs.com/bbs/2021/01/06/kuangstudy8432e8b6-1335-4e49-ab21-9b5f23acb055.png)

3. 网络

   ~~~bash
   bind 127.0.0.1 #绑定的ip，注释之后是都可以来访问，可以填写连接那台主机ip
   protected-mode yes #保护模式，一般是yes，远程连接可以改为no
   port 6379 #端口号
   ~~~

4. 通用(GENERAL)

   ~~~bash
   daemonize yes #守护线程,以守护进程的方式运行，默认是no，我们需要自己开启yes！
   
   pidfile /var/run/redis_6379.pid #以后台方式运行，就需要指定一个pid文件！
   ~~~

   ![img](https://kuangstudy.oss-cn-beijing.aliyuncs.com/bbs/2021/01/06/kuangstudyec42ab56-40f4-4e87-ab49-0f0fde8211b5.png)

   ~~~bash
   #日志
   # 指定服务器详细级别。
   # 这可以是以下之一：
   # debug（大量信息，对开发/测试有用）
   # verbose（许多很少有用的信息，但不是像debug级别那样混乱）
   # notice（适量的信息，你可能在生产中想要什么）
   # warning（仅记录非常重要/关键消息）
   logfile "" # 日志的文件位置名
   databases 16 # 数据库数量 16 个数据库
   always-show-logo no # 是否总是显示LOGO
   ~~~

5. 快照(SNAPSHOTTING)

   持久化，在规定时间内，执行多少次操作，则会持久化文件.rdb,.aof

   redis是内存数据，如果没有持久化，断电即失

   ~~~bash
   #  如果900秒内，至少有一个key进行了修改，就执行持久化操作
   save 900 1
   #  如果300秒内，至少10个key进行了修改，就执行持久化操作
   save 300 10
   #  如果60秒内，如果10000 key进行了修改，就执行持久化操作
   save 60 10000
   
   stop-writes-on-bgsave-error yes# 持久化如果出错，是否还需要继续工作！
   rdbcompression yes# 是否压缩rdb文件 ，需要消耗一些cpu资源
   rdbchecksum yes# 保存rdb文件的时候，进行错误的检查校验
   dir ./ # rdb文件保存的目录！
   ~~~

6. 复制(REPLICATION)

   略

7. 安全(SECURITY)

   ~~~bash
   config get requirepass # 获取redis密码
   config set requirepass "123456" #设置redis密码
   ~~~

8. 客户端(CLIENT)

   ~~~bash
   # maxclients 10000  #设置连接redis的最大客户端为10000
   # maxmemory <bytes> #最大的内存容量
   # maxmemory-policy noeviction #内存达到上限的解决的策略（六种解决方案）
   1、volatile-lru：只对设置了过期时间的key进行LRU（默认值）
   2、allkeys-lru ： 删除lru算法的key
   3、volatile-random：随机删除即将过期key
   4、allkeys-random：随机删除
   5、volatile-ttl ： 删除即将过期的
   6、noeviction ： 永不过期，返回错误
   ~~~

9. APPEND ONLY模式 aof配置

   ~~~bash
   appendonly no # 默认是不开期aof模式的，默认使用rdb方式持久化的，在大部分的情况下，rdb完全够用
   appendfilename "appendonly.aof" # 持久化的文件的名字
   
   # appendfsync always # 每次修改都会 sync 消耗性能
   appendfsync everysec # 每秒执行一次sync，可能会丢失这ls的数据
   # appendfsync no # 不执行sync 这个时候操作系统自己同步数据，速度最快！
   ~~~