---
title: 常用lambda（List相关）

date: 2018-05-27 09:25

updated: 2018-05-27 09:25

tags:
- lambda

categories: Java

permalink: lambda-list
---

过时原因：内容太简单，而且没有独特性，自己也全会了

## 提取出某个字段作为List

~~~java
// users [{id=1L, name=a}, {id=2L, name=b}, {id=3L, name=c}]

List<Long> userIds = users.stream().map(Order::getId).collect(Collectors.toList());

// userIds [1L, 2L, 3L]
~~~



## 从查找指定对象

~~~java
// users [{id=1L, name=a}, {id=2L, name=b}, {id=3L, name=Deolin}]

User user = users.stream().filter(User -> "Deolin".equals(user.getName()))
        .findFirst().orElseThrow(() - > new Exception("找不到"));

// user {id=3L, name=c}
~~~

> 这里`filter()`会去考虑**终止操作**是什么，这里终止操作是`findFirst()`，所以一旦找到符合条件的对象便不再继续寻找。



## 删除指定对象

~~~java
// users [{id=1L, name=a}, {id=2L, name=b}, {id=3L, name=null}]

users.removeIf(User -> User.getName() == null);

// users [{id=1L, name=a}, {id=2L, name=b}]
~~~



## 遍历

~~~java
// users [{id=1L, name=a}, {id=2L, name=b}, {id=3L, name=c}]

users.foreach(oneObject::oneMethod);

// 等价于 for (User user : users) {oneObject.oneMethod(user);}
~~~



## 改变每个对象

~~~java
// users [{id=1L, birthday=2017-01-02}, {id=2L, birthday=2018-01-02}]

users = users.stream().map(user -> user.setBirthday(user.getBirthday().replace('-', '/'))).collect(Collectors.toList());

// users [{id=1L, birthday=2017/01/02}, {id=2L, birthday=2018/01/02}]
~~~

> 注意null安全



## 排序

~~~java
// users [{id=3L, name=c}, {id=2L, name=b}, {id=1L, name=a}]

users.sort(Comparator.comparing(User::getId));

// users [{id=1L, name=a}, {id=2L, name=b}, {id=3L, name=c}]
~~~

> 复杂对比规则还是实现`Comparable`比较方便