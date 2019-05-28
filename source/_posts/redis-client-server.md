---

title: Redis 客户端与服务器实现

date: 2019-05-28 11:17

updated: 2019-05-28 11:17

tags:
- Redis

categories: Redis

permalink: redis-client-server

---

## 简介

这篇POST将会梳理Redis客户端与服务器主要功能的具体实现方式。



## Redis客户端

1. Redis是个1对n的服务器程序，一个服务器可以跟多个客户端建立网络连接，每个客户端可以向服务器发送命令

2. 对于每个与自己建立连接的客户端，服务端会以**链表**的形式为保存所有客户端的当前信息，链表里每个元素类型是`redisClient`

3. redisClient的属性
   - `fd` -1时代表这是个伪客户端，用于代表AOF或Lua发送命令；大于-1时代表客户端的stocket描述符
   - `name` 名称，默认为空白，可以通过`CLIENT setname`设置一个名字
   - `flags` 代表角色
     - 主从复制时，主、从互相是对方的客户端，他们的`flags`分别是`REDIS_MASTER`和`REDIS_SLAVE`
     - `REDIS_PRE_PSYNC`指服务器版本太低，无法PSYNC
     - `REDIS_LUA_CLIENT`指这是个处理Lua的伪客户端
     - 其它...
   - `querybuf`代表输入缓冲区，客户端每向服务器发一条命令，`querybuf`都会保存下整条命令的SDS
   - `argv` 将输入缓冲区分析后，将命令参数保存到`argv`数组，其中第0项是命令，后面的项是参数
   - `argc`是数据的长度，使获取长度的时间复杂度变为O(1)
   - `authenticated`记录客户端是否通过了身份验证，为0时服务器只接收`AUTH`命令
   - `ctime`创建客户端的时间，代表客户端与服务器连接多久了
   - `lastinteraction`客户端与服务器最后一次互动的时间
   - `obuf_soft_limit_reached_time`输出缓冲区第一次达到软限制的时间
   
4. 服务器是如何解析命令的？

  1. 服务端解析得到`argv`和`argc`，根据`argv[0]`从命令表查找对应的函数
  
     命令表是个字典，字典的值保存了函数标志、参数列表、总执行次数、总耗时等
  
  2. 执行命令，命令返回值会保存在输出缓冲区
  
     每个客户端有2个输出缓冲区，一个固定总长度，用于保存长度小的返回值，是个定长字符数组；另一个可变总长度，用于保存长返回值，是个字符串链表。

5. 当客户端的输出缓冲区超过了硬性限制，服务器会立即关闭客户端
6. 当客户端的输出缓存区超过了软性限制，服务器会根据`obuf_soft_limit_reached_time`来监视客户端，如果长时间超过软性限制，服务器会关闭客户端，如果没有，`obuf_soft_limit_reached_time`清零
7. 服务器的生命周期内，负责执行Lua脚本的伪客户端 会一直存在。
8. 服务器载入AOF文件期间，会创建负责执行AOF文件内命令的伪客户端，载入AOF完毕后这个客户端会被关闭



## Redis服务器执行命令



1. 客户端把用户输入的命令转化为协议，发向服务器

2. 客户端与服务器的Stock因为客户端写入变得可读，服务器读取协议，保存到输入缓冲区，调用**命令执行器**

3. 通过命令表找个函数，至此，函数、参数、参数个数集齐

4. 检查找到的函数是否为NULL

   检查函数所需参数个数与`argc`代表的参数个数是否一致

   检查身份验证

   检查内存是否足够

   BGSAVE、订阅、数据载入、Lua时相关限制的检查

5. 调用函数，返回值保存到输出缓冲区

6. 慢查询日志（如果开启了的话）

   更新客户端的`milliseconds`属性、`calls`自增1

   AOF持久化这条命令（如果开启了的话）

   将命令传播到从服务器（如果有从服务器的话）

7. Stocket变为可写状态，**命令回复器**将输入缓冲区的返回值转化为协议，发向客户端。
8. 客户端将协议转化为返回值，打印到终端



## serverCron函数

1. 默认100ms执行一次
2. 服务器状态保存了当前系统时间的缓存，serverCron每次执行会更新时间缓存，所以精确度不高
3. 对时间精确度要求不高的任务，会基于系统时间缓存来定时执行，以减少系统调用次数

4. `trackOperationsPerSecond`每100ms执行一次，抽样计算，估出最近1s内处理的命令数量

   估算方式是统计两次函数调用之间执行的命令数，算出每ms的命令数，乘以1000

5. serverCron会查看服务器当前的内存使用量

6. serverCron会检查客户端与服务器是否长时间无互动，释放占用过高的客户端输入缓冲区

7. serverCron会检查并删除过期键，收缩字典等操作

   