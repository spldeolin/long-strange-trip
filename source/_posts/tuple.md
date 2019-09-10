---

title: 使用元组为方法声明多个返回值

date: 2019-09-10 08:21

updated: 2019-09-10 08:21

tags:
- Java

categories: Java

permalink: tuple

---

## 简介

回顾《Thinking in Java》的泛型那章时，看到的一个很好的实践方式——元组。

利用元祖，可以很优雅的实现方法多返回值

## 利用POJO实现多返回值

1. 声明POJO类

   ~~~java
   @Data
   @AllArgsConstructor
   public class UserInfo() {
       private final Integer age;
       private final String username;
   }
   ~~~

2. 业务代码

   ~~~java
       public void doSomething(Long userId) {
           UserInfo userInfo = this.getUserInfo(userId);
           printAsLog(userInfo);
       
           // ... do something
       }
   
       /**
        * 获取用户名和用户年龄
        */
       private UserInfo getUserInfo(Long userId) {
           // 从DB获取User实体
         	// 获取用户名和用户年龄
           // 最后构建UserInfo对象并返回
       }
   ~~~

3. 说明

   很简单的一个示例，`doSomething()`需要通过一个私有方法`UserInfo()`获取用户ID和用户名，我们不得不定义一个POJO`UserInfo`，因为Java的方法只能声明一个返回类型，如果想要一次返回多个返回值，似乎只能自定义一个POJO。

4. 使用POJO作为返回类型存在的问题

   这个POJO很难复用，因为它用来代表“用户名+用户年龄”，如果有一个方法需要返回“用户名+用户性别”，那么必须再自定义一个POJO。

   当然，你可能会觉得完全可以定义一个叫做“用户名+用户年龄+用户性别”的POJO，它完全可以作为这两个方法的返回值，但是，这样设计的话，方法签名的约束性就没有了，因为这个方法的返回值声明了3个字段，而方法的具体实现只会返回2个字段，不去查看方法的具体实现，根本无从得知这个方法会具体返回什么，毕竟这个返回值被设计得“大而全”了。

## 泛型

说了那么久，我们究竟想要什么？

我们受到了**<u>Java方法只能声明一个返回值类型</u>**的限制，我们想要通过一个方法解除这个限制，使用POJO是一个方法，但POJO的缺点是，每出现一种返回类型的组合，就必须定义一个新的POJO，这样做太麻烦，早晚会有开发者会因为麻烦而去往已有的POJO中加field，这会导致方法签名的约束性丧失。

现在的问题是如何对应多种多样的类型？

多种多样的类型，能令人想到什么？显然，是——泛型

为什么要固定地声明DTO的类型呢？完全可以用泛型来代替它们

~~~java
@Getter
@AllArgsConstructor
public class UserInfo<A, B>() {
    private A first;
    private B second;
}
~~~

如此依赖，只需要一个`UserInfo<A, B>`类，就既可以作为“获取用户名、用户年龄”和“获取用户名、用户性别”的返回类型了

~~~java
private UserInfo<String, Integer> getUsernameAndAge(Long id);
~~~

~~~java
private UserInfo<String, Byte> getUsernameAndGender(Long id);
~~~

## 元组

这个时候，`UserInfo`显然不再合适了，因为除了“用户信息”，它还可以代表很多的信息，这种用来代表<u>**返回值列表**</u>的类，称为元组（Tuple），声明了两个field的元组，就是二元祖（DoubleTuple）

## 使用元组的建议

不建议使用四元祖以上的元祖，超过4个返回值类型建议使用普通的POJO，否则你可能会看不清他们。（这个建议就跟方法的参数列表超过4个时建议使用DTO一样）

可以看看下面这个例子

~~~java
/**
 * 丧心病狂的六元祖
 *
 * 这个方法想要返回学生ID、到校时间、放学时间、上课时长、课间时长和午休时长
 * 如果没有这个注释，仅仅通过方法签名，根本无法得知每个泛型代表什么意思
 */
public SiveFoldTuple<Long, Date, Date, Integer, Integer, Integer> getStudentDaily(Long id);
~~~

这种方法使用POJO作为返回值类型显然更好

~~~java
@Getter
@AllArgsConstructor
public class StudentDailyDTO {
    private final Long studentId;
    private final Date arrivedAt;
    private final Date leftAt;
    private final Integer lessonDuration;
    private final Integer breakDuration;
    private final Integer noonRestDuration;
}
~~~

~~~java
public StudentDailyDTO getStudentDaily(Long id);
~~~





