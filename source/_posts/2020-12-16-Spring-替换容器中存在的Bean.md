---

title: Spring 替换容器中存在的Bean

excerpt: 偶尔需要重写一些别人的提供的组件

date: 2020-12-16 18:59

updated: 2020-12-16 18:59

tags:
- Spring

categories: Java

permalink: spring-replace-bean

---



提供一个替换Spring容器中的已经存在的Bean的方法

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

也不希望修改@Autowired了`SomeService`的Bean时（因为不希望动到后者内部的代码）

就可以通过实现`BeanPostProcessor`，将`SomeServiceImpl`替换成其他实现类

~~~java
@Component
public class SomeServiceBpp implements BeanPostProcessor {

    @Autowired
    private SomeServiceImpl someServiceImpl;

    @Override
    public Object postProcessBeforeInitialization(Object bean, String beanName) throws BeansException {
        if (bean instanceof SomeService) {
            return new SomeService() {
                @Override
                public int doSth() {
                    int i = someServiceImpl.doSth();
                    return -i; // 演示一下 修饰被替换实现类的返回值
                }
            };
        }
        return bean;
    }

    @Override
    public Object postProcessAfterInitialization(Object bean, String beanName) throws BeansException {
        return bean;
    }
}
~~~



替换行为完全没有影响到组件本身以及组件的调用方

而且只要把`SomeServiceBpp`注释掉，或是去掉它的`@Component`注解，替换行为就会消失

实现了解耦