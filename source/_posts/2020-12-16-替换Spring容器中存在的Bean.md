---

title: 替换Spring容器中存在的Bean

excerpt: 不得已的时候，需要重写一些别人的提供的组件

date: 2020-12-16 18:59

updated: 2020-12-17 14:45

tags:
- Spring

categories: Java

permalink: spring-replace-bean

---



找到了个方法——用于替换Spring容器中的Bean

效果是不改变Bean本身，也不改变Bean的调用方



~~~java
public interface SomeService {
    int doSth();
}
~~~

~~~java
public class SomeServiceImpl implements SomeService {
    @Override
    public int doSth() {
        log.info("original");
        return 1;
    }
}
~~~



当没办法修改`SomeServiceImpl`（比如这个类在别人负责的项目中）

也不希望修改@Autowired了`SomeService`的Bean时（因为不希望改动到后者内部的逻辑）

可以利用@Primary结合继承的方式来替换

~~~java
@Primary
@Component
public class SomeServiceImplEx extends SomeServiceImpl {
    
    @Override
    public int doSth() { // 修饰或是直接重写
        log.info("last");
        int superResult = super.doSth(); 
        return -superResult; 
    }

}
~~~



`SomeServiceImplEx`没有影响到Bean本身以及Bean的调用方

而且只要把`SomeServiceImplEx`删除，或是去掉它的`@Component`注解，替换行为就会消失

实现了解耦