---

title: Redis底层数据结构 下

date: 2019-05-07 16:36

updated: 2019-05-07 16:36

tags:
- 数据结构
- Redis

categories: Redis

permalink: redis-data-structure-2

---

## 简介

这篇POST紧接[上一篇](https://spldeolin.com/posts/redis-data-structure-1/)



## 整数集合

1. 当SET键内元素只有整数而且数量不多时，Redis会使用整数集合作为这个集合键的底层实现

2. 整数集合的结构

   ~~~c
   typedef struct intset {
       uint32_t encoding; // 编码，每个元素都使用相同的编码
       
       uint32_t length; // 元素个数
     
       int8_t contents[]; // 元素数组，有序，不重复
   }
   ~~~

3. `contents[]`不保存任何int8_t类型的元素，数组的元素类型由`encoding`决定

4. 类型的长度：`int64_t`长于`int32_t`，`int32_t`长于`int8_t`

5. 升级：当一个类型更长整数插入整数集合时，整个集合的所有元素都要升级到长类型

   - 根据新元素的类型，拓展`contents[]`的空间大小
   - 将`contents[]`中每个元素都转换成与新元素相同的类型，完毕后将元素放入正确的数组位置
   - 将新元素添加到`contents[]`中，`encoding`改为新类型，`length`自增`1`
   - Redis中的整数集合只能升级，不能降级



## 压缩列表

1. 当LIST键或HASH键的k-v较少时，而且k和v都是较小整数或者较短字符串时，Redis会使用压缩列表作为它们的底层实现。
2. 压缩列表的作用是节省内存，它类似于一个顺序的线性表，可以想象成一个数组
3. 压缩列表保存了哪些信息
   1. zlbytes 记录整个压缩列表占用的内存数，用于内存重分配或计算zlend位置
   2. zltail 记录尾结点距压缩列表起始地址的字节数，用于确定尾结点的地址
   3. zllen 结点数量
   4. entry1、entry2、entry3…… 每个结点
   5. zlend 压缩列表最后一个标记位，用于标记这个压缩列表结束了
4. 每个结点可以保存一个**字节数组**或和一个**整数值**
5. 每个结点由3个属性组成
   - previous_entry_length 记录前一个结点的长度，结合当前结点的起始位置可以计算前一个结点的起始位置
   - encoding 记录结点的数据类型和长度
   - content 数据域



## 对象

1. Redis基于SDS、链表、字典、整数集合、压缩列表这些底层数据结构构建了一个**对象**系统

2. 一共有5种对象——字符串对象、列表对象、哈希对象、集合对象、有序集合对象

3. Redis每插入一个k-v时，会至少创建2个对象，一个是k对象，另外n个是v对象

4. 对象的结构

   ~~~c
   typedef struct redisObject {
       unsigned type:4; // 类型，上文的那5种
       unsigned encoding:4; // 编码 
       void *ptr; // 代表这个对象是由哪种底层数据类型实现的
   }
   ~~~

5. `encoding`记录了对象的编码，每种编码都有对应的底层数据结构。每种对象类型可能有多种编码

6. 对象类型、编码、底层数据结构的关系是这样的

   ![](/images/redis-data-structure-2-01.png)

7. embstr是一种特殊的SDS，比起一般的SDS，它对短字符串作出了一些优化



## 对象的内存回收

1. `redisObject`用`refcount`属性记录对象的**引用计数**

   - 刚创建的新对象，计数为1
   - 每被一个新程序使用时，计数自增1
   - 每不被一个新程序使用时，计数自减1

2. 计数变为0时，释放内存
3. 当复数个k对应的v是同一个时，v会被共享使用，所以的k都会指向它，每被共享一次，计数自增1
4. 比较两个对象是否相同十分耗费CPU，所以Redis种只有整数字符串对象会被共享

5. `redisObject`的第5个属性是`lru`，记录了该对象最后一次被使用的时间，当Redis服务启用`maxmemory`并且回收算法是`volatile-lru`或者`allkeys-lru`时，空转时间更长的k-v会被释放掉

   