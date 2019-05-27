---

title: 让Java代码变得好看些（持续更新）

date: 2019-05-27 09:20

updated: 2019-05-27 09:20

tags:
- 技巧

categories: Java

permalink: make-java-beautiful

---

## 简介

这篇POST将会收录一些技巧，这些技巧能够使Java代码看上去更漂亮些。

所有技巧都依赖 `Java 8` 和 `Google Guava`。



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

