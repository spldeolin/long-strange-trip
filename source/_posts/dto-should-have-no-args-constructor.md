---
title: 各种DTO类最好有 无参数的构造方法

date: 2017-12-23 10:05:00

updated: 2017-12-23 10:05:00

tags:

- lombok
- 报错

categories: Java

permalink: dto-should-have-no-args-constructor
---

以以下这个类为例

~~~java
@Data
class Cookie {

    private String name;

    public Cookie(String name) {
        this.name = name;
    }

}
~~~

这个`Cookie`，当它需要被序列化的时候（比如存入Reids等），一部分序列化工具可能会直接报错，因为有些序列化工具类是通过`clazz.newInstance()`方法生成初始对象的。

没有无参构造方法的类，`clazz.newInstance()`会报`java.lang.NoSuchMethodException`错误，所以需要给`Cookie`声明一个无参构造方法。



一般的DTO，构造方法里也不需要任何的初始化处理，那直接用lombok的注解就可以了

~~~
@Data
@NoArgsConstructor
class Cookie {

    private String name;

    public Cookie(String name) {
        this.name = name;
    }

}

~~~

