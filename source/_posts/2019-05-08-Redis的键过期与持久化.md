---

title: Redis的键过期与持久化

date: 2019-05-08 11:00

updated: 2019-05-08 11:00

tags:
- Redis

categories: Redis

permalink: redis-expire-persistence

---

## 简介

这篇POST将会整理出Redis是如何实现键过期的，以及Redis数据库持久化的两个方式



## Redis的结构

1. 一个单机Redis服务由底层的`RedisServer`结构体表示，`RedisServer`内部持有了`RedisDb[]`数组，后者代表每个Redis数据库，默认16个
2. RedisDb中维护了两个字典
   - 保存了键值对的的字典，称为键空间
   - 保存了键与失效时间的字典，称为过期字典
3. 什么样的键算是过期了？
   1. 检查键在过期字典中有没有过期时间
   2. 有的话检查当前UNIX时间戳是否大于键的过期时间
4. 过期策略
   1. 定时器删除，对内存友好，多CPU不友好
   2. 每次GET的时候惰性删除，对CPU友好，对内存不友好
   3. 定时、惰性结合的方式，启用惰性删除，并且每个一定时间定时删除一波

5. 惰性删除的实现

   在执行读写命令之前，会先调用`expireIfNeeded`函数，如果过期，则删除，然后执行原命令

6. 定期删除的实现

   1. 周期性调用`activeExpireCycle`函数，每次从一定量的`redisDb`中随机取出一定量的键进行检查，删除过期键

   2. 全局变量current_db会记录函数的检查进度，下次调用函数时从本次的进度开始处理
   3. 当所有redisDb都被检查过一遍时，current_db清0，准备开始新一轮的检查

7. RDB遇到过期键

   保存：当RDB持久化时遇到过期键，新生成的RDB文件不会保存过期键

   载入：主服务器载入RDB文件时，遇到过期键会忽略，而从服务器则会将过期键载入，不过下次从主同步时，过期的会被清除

8. AOF遇到过期键

   保存：AOF只会在过期键被惰性删除或是定期删除后，才会在AOF文件中追加一条DEL指令，显示地删除

   载入：与RDB类似



## RDB持久化

1. SAVE和BGSAVE命令可以让Redis进行RDB持久化，两者的区别是是否阻塞Redis服务器
2. Redis没有载入RDB文件的命令，载入RDB都是在服务器启动时执行的
3. BGSAVE期间，SAVE和BGSAVE命令都会被禁止
4. BGSAVE期间，BGREWRITEAOF会被延迟到BGSAVE结束，因为两者都是大型的硬盘写入操作，所以不应该同时执行



## AOF持久化

1. RDB文件保存的是键指对，AOF保存的是键值对的插入命令语句

2. 当Redis启用AOF时，服务器每执行完一个写命令，就是在aof_buf追加这条写命令，服务器每结束一个事件循环，都会调用flushAppendOnlyFile，将aof_buf里的语句写入到文件中
3. appendfsync选项决定了flushAppendOnlyFile函数的行为
   - always，每次调用函数时都写入并同步到文件，效率最低，但安全性最高，即便服务挂了，最坏的请求也只是损失一次时间循环的数据
   - everysec，如果上次AOF距现在超过1秒，才进行同步，默认为这种方式
   - no，有操作系统决定何时同步
4. 当AOF文件足够大时，Redis会对AOF文件进行重写，创建一个新的AOF文件，保存的状态与原AOF文件相同，但不会包含冗余命令

 