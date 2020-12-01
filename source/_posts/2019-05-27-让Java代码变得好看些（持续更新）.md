---
title: 让Java代码变得好看些（持续更新）

date: 2019-05-27 09:20

updated: 2020-04-14 14:31

tags:
- 总结

categories: Java

permalink: beautiful-java

---

## 简介

收录一些技巧，它们能够使Java代码看上去更漂亮些。



## 1. 使用现成的类

1. 防止下标越界

   ~~~java
   import com.google.common.collect.Iterables;
   
   User firstOne = Iterables.getFirst(users, null);
   User lastOne = Iterables.getLast(users, null);
   ~~~

   

2. null缺省

   ~~~java
   import com.google.common.base.MoreObjects;
   
   Integer age = MoreObjects.firstNonNull(user, DEFAULT_USER).getAge();
   ~~~

   在Guava的早期版本中，`firstNonNull`是声明在`com.google.common.base.Objects`中的，这容易与`java.util.Objects`产生冲突，所以不推荐使用过于早版本的Guava

   

3. String转Long时的null安全

   ~~~java
   import org.apache.commons.lang3.math.NumberUtils;
   
   Long a = NumberUtils.createLong(user.getValue());
   ~~~

   

4. 字符串拼接

   ~~~java
   import com.google.common.base.Joiner;
   
   StringBuilder msg = Joiner.on("、").appendTo(new StringBuilder("这些用户已被禁用："), userIds).append("。");
   throw new Exception(msg.toString()); // e.g.: 这些用户已被禁用：1、2、3。
   ~~~

   

5. 使用`Multimap<K， V>`代替`Map<K, List<V>>`

   

6. 使用`Table<K1, K2, V>`代替`Map<K1, Map<K2, V>>`

   

7. 对java.util.Optional里的值做判断

   ~~~java
   boolean isKilled(Long studentId) {
       Optional<Student> opt = studentService.get(studentId);
       return opt.filter(Student::getKillFlag).isPresent();
   }
   ~~~

   

8. 获取唯一元素

   ~~~java
   import com.google.common.collect.Iterables;
   
   Foo foo = Iterables.getOnlyElement(foos);
   ~~~

   如果数据异常，`Iterables.getOnlyElement`会自己抛出`NoSuchElementException`或是`IllegalArgumentException`



## 2. 使用`java.time`包代替`java.util.Date`

不推荐后者的原因可以参考官方文档

https://docs.oracle.com/javase/tutorial/datetime/iso/legacy.html

前者的具体用法可以参考Deolin的这篇POST

[Java8 time包指南](https://spldeolin.com/posts/java8-time-guide/)



## 3. 谨慎使用继承（extends）

继承代表父子耦合，使用继承前想一想，“这里需要用到多态吗？不需要的话用“组合”行不行？”

继承会引发的BUG可以参考 `add` 和 `addAll` 的那个例子



## 4. 建议使用`ValidEnumValue`注解

参考Deolin的这篇POST

[如何校验枚举值]( https://spldeolin.com/posts/validate-enum-value/ )



## 5. 预测上层会对自己返回值进行的处理

通过这样的预测，并根据预测适当地封装成一些新的方法，以此来提升上层调用自身时的手感，以及降低上层处理返回值的复杂度

~~~java
// before
public UserDto getUser(Long userId) {
    return userDao.get(userId);
}
~~~

~~~java
// after
public Optional<UserDto> getUser(Long userId) {
    return Optional.ofNullable(userDao.get(userId));
}

public UserDto getUserOrElseThrow(Long userId) {
    return getUser().orElseThrow(() -> new UserAbsentException(userId));
}

public UserDto getUserOrElseNew(Long userId) {
    return getUser().orElseThrow(new UserDto());
}
~~~



## 6. 还可以为Controller以外的Component进行参数校验

这样做能——

- 确保API方法的安全性
- 提高定位错误地调用的速度

为了达到这个效果需要做3件事——

- 在interface的方法签名的参数上声明校验注解

- 在实现类上声明@Validated

- 使用切面报告非法调用行为

原理以及设计思路可以参考Deolin的这篇POST

[Spring 组件参数校验](https://spldeolin.com/posts/spring-component-validated/)



## 7. 适合打印日志的4个场景

1. “值类型”或者元素是“值类型”的容器，构建/初始化完毕后，打印日志是有必要的。

   比起Javabean，“值类型”对象toString的返回值往往很短，打印它们开销很小，同时他们代表的信息对于调试又很关键（例如`List<Long>`，它们往往代表着一些实体的外键）

   ~~~java
   Map<Long, Integer> extractIds = Maps.newHashMap();
   javabeans.forEach(one -> extractIds.add(one.getId()));
   log.info("extractIds={}", extractIds.toString())
   ~~~

   

2. 调用其他模块的API之后

   ~~~java
   List<String> itemAttrValues = goodsApi.getAttrValues(itemIds, true, 500);
   log.info("goodsApi.getAttrValues({}, true, 500)={}", itemIds, itemAttrValues);
   ~~~

   

3. return null之前

   ~~~java
   if (items.size() == 0) {
       log.warn("找不到任何商品，itemName={}", itemName); // warn还是info取决于业务
       return null;
   }
   ~~~

   

4. 请求无法继续执行时

   ~~~java
   try {
       stockApi.lockStock(id, delta);
   } catch (OutOfStockException e) {
       log.error("库存不足，可能是操作期间C端下单所致");
       throw new SubmitOrderException();
   }
   ~~~



## 8. 考虑使用一个`ApiImpl来`实现多个`Api`

当你的所有`ApiImpl`里实现的方法没有任何业务逻辑，只有调用`Service`和数据组装时，可以考虑使用一个`ApiImpl来`实现多个`Api`

这样做有助于减少IDE阅读代码时需要打开的窗口数，提高阅读代码的效率

由于改动的仅仅是实现，所以“**接口隔离原则**”既然是遵守的。



## 9. 本应该查询到1个但是查到多个时，考虑抛出异常

 ~~~java
/**
 * 根据手机号获取用户，不存在时返回null
 */
public User getUserOrElseNull(String telephone) 
        throws MoreThanOneResultsException {
    List<User> users = userDao.select(new Wrapper("telephone", telephone));
    
    if (users.size() == 0) {
        return null;
    }
    
    // 预想之外的情况
    if (users.size() > 1) { 
        // 抛出异常交给调用者处理，如果业务上不可能发生，算作BUG；如果是正常情况，那么调用方可以捕获
        throw new MoreThanOneResultsException(users);
    }
    
    return users.get(0);
}
 ~~~

~~~java
/**
 * 当本应返回1个满足条件结果的方法内部获取到不止1个结果时，应该抛出这个异常。
 */
public class MoreThanOneResultsException extends RuntimeException {

    private static final long serialVersionUID = 3999892488916264668L;

    private Object firstOne;

    private Collection<?> results;

    public MoreThanOneResultsException(Collection<?> results) {
        super();
        if (results == null || results.size() == 0) {
            throw new IllegalArgumentException("Results cannot be empty.");
        }
        this.results = results;
        this.firstOne = Iterables.getFirst(results, null);
    }

    @SuppressWarnings("unchecked")
    public <T> T orElseFirst() {
        return (T) firstOne;
    }

    @SuppressWarnings("unchecked")
    public <T> Collection<T> getResults() {
        return (Collection<T>) results;
    }

    @Override
    public String getMessage() {
        return "Method obtained more than one result, perhaps caused by data problems. " + results;
    }
}
~~~



## 10. 通过private构造方法内部直接抛出异常来防止一个类被实例化

- 仅仅是在类上加`abstract`是不可行的，因为它可以被用户派生出一个可以实例化的子类。也非常容易让用户误认为这个类设计出来的目的是被继承。
- 仅仅私有化唯一的构造方法也是不可行的，因为用户可以通过反射的方式修改访问修饰符
- 这种做法建议结合**“在类上声明`final`”**，否则一旦有用户继承它便会出现编译错误，编译错误虽说能第一时间报告错误，但可能依然会让用户困惑，直接final化这个类可以让用户直接明白你的设计意图