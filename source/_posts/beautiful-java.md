---
title: 让Java代码变得好看些（持续更新）

date: 2019-05-27 09:20

updated: 2019-12-22 13:53

tags:
- 总结

categories: Java

permalink: beautiful-java

---

## 简介

收录一些技巧，它们能够使Java代码看上去更漂亮些。



## 1. 防止下标越界

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
import com.google.common.collect.Iterables;

User user = Iterables.getFirst(users, null);
~~~



同理还有`Iterables.getLast()`



## 2. null-to-default

原来的做法

~~~java
Integer age = user.getAge() != null ? user.getAge() : DEFAULT_USER.getAge();
~~~



更好的做法

~~~java
import com.google.common.base.MoreObjects;

Integer age = MoreObjects.firstNonNull(user, DEFAULT_USER).getAge();
~~~



在早期版本的Guava中，`firstNonNull`是声明在`com.google.common.base.Objects`中的，这容易与`java.util.Objects`产生冲突，所以不推荐使用过于早版本的Guava



## 3. String -> Long时的null-safe

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
import org.apache.commons.lang3.math.NumberUtils;
Long a = NumberUtils.createLong(user.getValue());
~~~



## 4. 字符串拼接

原来的做法

~~~java
if (userIds.size() > 0) {
    StringBuilder msg = new StringBuilder("这些用户已被禁用：");
    userIds.forEach(userId -> msg.append(userId).append("、"));
    msg.deleteCharAt(msg.length() - 1);
    msg.append("。")
    throw new Exception(msg.toString()); // e.g.: 这些用户已被禁用：1、2、3。
}
~~~



更好的做法

~~~java
import com.google.common.base.Joiner;

if (userIds.size() > 0) {
    StringBuilder msg = Joiner.on("、").appendTo(new StringBuilder("这些用户已被禁用："), userIds).append("。");
    throw new Exception(msg.toString()); // e.g.: 这些用户已被禁用：1、2、3。
}
~~~



## 5. 判断Optional里的value

原来的做法

~~~java
boolean isKilled(Long studentId) {
    Optional<Student> opt = studentService.get(studentId);
    if (opt.isPresent()) {
        return opt.get().getKillFlag();
    }
    return false;
}
~~~



更好的做法

~~~java
boolean isKilledV2(Long studentId) {
    Optional<Student> opt = studentService.get(studentId);
    return opt.filter(Student::getKillFlag).isPresent();
}
~~~



## 6. 获取容器中唯一的元素

原来的做法

~~~java
Foo foo;
if (foos.size == 0) {
    throw new OneRuntimeException();
} else if (foos.size > 1) {
    throw new AnotherRuntimeException();
} else {
    foo = foos.get(0);
}
~~~



更好的做法

~~~java
import com.google.common.collect.Iterables;

Foo foo = Iterables.getOnlyElement(foos);
~~~



如果数据异常，`Iterables.getOnlyElement`会自己抛出`NoSuchElementException`或是`IllegalArgumentException`