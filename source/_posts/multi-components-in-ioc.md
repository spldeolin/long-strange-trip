---
title: Spring 批量装配组件

date: 2019-01-11 09:52

updated: 2019-01-11 09:52

tags:
- Spring

categories: Java

permalink: multi-components-in-ioc
---



## 简介

今天在项目中看到一种很有意思的实践，这种做法依赖了Spring 自动装配（@Autowired）的一个特性。Deolin还是第一次知道有这样的特性...



## 特性示例

首先，简单介绍一下这个特性。

1. 定义一个接口和一批派生类

   ~~~java
   public interface Facede {
       Object doSomething(Object args);
   }
   ~~~

   ~~~java
   @Component public class Impl1 implements Facede {
       @Override public Object doSomething(Object arg) {return "111";}
   }
   ~~~

   ~~~java
   @Component public class Impl2 implements Facede {
       @Override public Object doSomething(Object arg) {return "222";}
   }
   ~~~

   ~~~java
   @Component public class Impl3 implements Facede {
       @Override public Object doSomething(Object arg) {return "333";}
   }
   ~~~

2. 然后，就可以一口气注入它们了！

   ~~~java
   @Component
   public class Client {
   
       @Autowired                      // autowired 的批量装配特性
       private List<Facede> facedes;
   
   }
   ~~~

3. 验证一下

   ~~~java
       public report() {
           facedes.forEach(one -> log.info(one.doSomething()))
       }
   ~~~

   ~~~
   17:08:19 INFO [Client:23] - success 1
   17:08:19 INFO [Client:23] - success 2
   17:08:19 INFO [Client:23] - success 3
   ~~~


## 应用

基于这个特性，可以设计出一种**基于枚举的实现类分发器**

~~~java
public enum SimpleEnum {

    OneStyle("no.1", Impl1.class),
    TwoStyle("no.2", Impl2.class),
    ThreeStyle("no.3", Impl3.class);

    @Getter @Setter
    private String name;

    @Getter @Setter
    private Class<? extends Facede> implType;

    Enum(String name, Class<? extends Facede> implType) {
        this.name = name;
        this.implType = implType;
    }

}
~~~

~~~java
@Component
@Log4j2
public class ImplDispatcher {
    
    private Map<Class<? extends Facede>, Facede> map;
    
    public ImplDispatcher(@Autowired List<Facede> facedes) {
        map = new HashMap<>(facedes.size());
        facedes.forEach(one -> map.put(one.getClass(), one));
    }
    
    /**
     * 根据枚举分发一个实现类对象
     */
    public Facede dispatch(SimpleEnum e) {
        return map.get(e.getImplType());
    }

}
~~~



## 源码

如果觉得难以理解可以参考这个[可运行的示例](https://github.com/spldeolin/multi-components-dispatcher-demo.git)。