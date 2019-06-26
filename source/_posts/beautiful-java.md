---

title: 让Java代码变得好看些（持续更新）

date: 2019-05-27 09:20

updated: 2019-06-25 16:48

tags:
- 总结

categories: Java

permalink: beautiful-java

---

## 简介

这篇POST将会收录一些技巧，这些技巧能够使Java代码看上去更漂亮些。

Deolin假定了你的项目都依赖 `Java 8` 、 `Google Guava`和`Apache Commons Lang3`。



##防止下标越界

原来的做法

~~~java
User user;
if (users.size() > 0) {
    user = users.get(0);
} else {
    user = null;
}
~~~



更好的做法

~~~java
User user = Iterables.getFirst(users, null);
~~~



`Iterables`还提供了`getLast()`方法，与`getFirst()`同理



## 为空时的默认值

原来的做法

~~~java
Integer age1 = user.getAge() != null ? user.getAge() : DEFAULT_AGE;

Integer age2 = user.getAge();
if (age2 == null) {
    age2 = DEFAULT_AGE;
}
~~~



更好的做法

~~~java
Integer age = com.google.common.base.Objects.firstNonNull(user.getAge(), DEFAULT_AGE)
~~~



## String -> Long时的null-safe

原来的做法

~~~java
String b = user.getValue();

Long a = null;
if (b != null) {
    a = Long.valueOf(b);
}
~~~



更好的做法

~~~java
Long a = NumberUtils.createLong(user.getValue());
~~~



## 字符串拼接

原来的做法

~~~java
throw new Exception("第" + invalidSheet + "个sheet，第" + invalidRow + "行数据格式不正确。错误信息：" + e.getMessage());
~~~



更好的做法

~~~java
throw new Exception(Joiner.join("第", invalidSheet, "个sheet，第", invalidRow, "行数据格式不正确。错误信息：", e.getMessage()))
~~~



