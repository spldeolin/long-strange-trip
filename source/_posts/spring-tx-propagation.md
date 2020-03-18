---

title: Spring事务的传播级别

date: 2020-03-18 11:06

updated: 2020-03-18 11:25

tags:
- Spring

categories: Java

permalink: spring-tx-propagation

---

## 简介

**传播级别**是一个Spring在对事务管理的非常重要的特性。指定一个事务方法的传播级别可以对这个事务的作用范围进行控制。

Spring事务管理一共有7种传播级别，这篇POST将会详细地介绍它们。

而这篇POST将会介绍Spring事务管理中的7种传播级别。





## 1. REQUIRED级别

- 如果没有专门指定，`REQUIRED`会是默认的传播级别
- 如果外层存在一个事务，则加入到外层事务中；否则自己新建一个事务

~~~java
OuterService {

  @Tx
  method() {
      saveIntoDB(); // rollback
      innerService.method(); 
      int b = 1 / 0;
  }
}

InnerService {
  
    @Tx(REQUIRED)
    method() {
        saveIntoDB(); // rollback
    }
}
~~~





## 2. SUPPORTS级别

- 如果外层存在一个事务，则加入到外层事务中；否则非事务执行，就当没这个注解

~~~java
OuterService {

    method() {
        innerService.method();
    }

}

InnerService {

    @Tx(SUPPORTS)
    method() {
        saveIntoDB(); // not rollback
        int b = 1 / 0;
    }
}
~~~





## 3. MANDATORY级别

- 如果外层存在一个事务，则加入到外层事务中；否则一调用就抛出异常，执行不到内层方法

  `org.springframework.transaction.IllegalTransactionStateException: No existing transaction found for transaction marked with propagation 'mandatory'`





## 阶段性总结

`PROPAGATION_REQUIRED`、`PROPAGATION_SUPPORTS`、`PROPAGATION_MANDATORY`这三个传播级别碰到外层存在事务时处理方式是一致的，都是加入到外层事务；区别在于外层没事务时的处理方式。





## 4. REQUIRES_NEW级别

- 如果外层不存在事务，新建事务
- 如果外层存在事务，挂起外层事务，本层新建一个事务，执行完返回后直接提交，外层如果后续再抛异常，不回滚内层。
- 适用场景：使用一个方法记录操作日志，无论外层后续是否失败，操作日志都不应该回滚，因为这次操作客观发生过，只不过失败了。

~~~java
OuterService {

  @Tx
  method() {
      saveIntoDB(); // rollback
      innerService.method(); 
      int b = 1 / 0;
  }
}

InnerService {
  
    @Tx(REQUIRED_NEW)
    method() {
        saveIntoDB(); // not rollback
    }
}
~~~





## 5. NOT_SUPPORTED级别

- “如果外层不存在事务，不新建事务”版的`PROPAGATION_REQUIRES_NEW`





## 6. NEVER级别

- “如果外层不存在事务，不新建事务”版的`PROPAGATION_MANDATORY`





## 7. NESTED级别

- 如果外层不存在事务，新建事务
- 如果外层存在事务，也是单独提交单独回滚，不过内层会利用外层的`savepoint`来进行回滚，内层需要回滚时，只需回滚到指定`log`的位置，不需要新建连接，开销很低
- 适用场景：批量处理数据时，有一趟失败了，只回滚这一趟，不影响其他。这个场景不使用`PROPAGATION_REQUIRES_NEW`的原因是后者开销比较大

~~~java
OuterService {

  @Tx
  method() {
        for (
            try {
                innerService.method(); 
            } catch (e) {
                log.error(e);
            }
        )
    }
}

InnerService {
  
    @Tx(NESTED)
    method() {
        saveIntoDB();
        if (new Random().nextBoolean()) {
            int b = 1 / 0;
        }
    }
}
~~~





## 总结

`NEW`和`NESTED`比较特殊，都是外层异常不回滚内层的传播级别，区别在于开销的高低





## 写这篇POST时遇到的一个问题

事务方法必须是`public`的，否则事务将会失效

~~~java
//~ com.spldeolin.tx.TxController.java
    @GetMapping("/tx")
    Object tx() {
        txService.working("演示1");
        txService.notWorking("演示2");
        return null;
    }

//~ com.spldeolin.tx.TxService.java  
// 这个类没有实现接口，与TxController在同一包下，所以notWorking方法能被调用
    @Transactional
    public void working(String a) {
        BizDemoEntity bizDemo = new BizDemoEntity();
        bizDemo.setName(a);
        bizDemoMapper.insert(bizDemo);
        if (true) {
            throw new RuntimeException("something happened.");
        }
    }

    @Transactional
    void notWorking(String a) {
        BizDemoEntity bizDemo = new BizDemoEntity();
        bizDemo.setName(a);
        bizDemoMapper.insert(bizDemo);
        if (true) {
            throw new RuntimeException("something happened.");
        }
    }

~~~

在这个示例中，数据`演示1`被回滚了，而`演示2`没有被回滚，因为notWorking方法的事务注解没有生效。

#### 为什么会出现这个问题？

Spring的声明式事务是通过AOP实现，有一个名为`AbstractAutoProxyCreator`的`BeanPostProcessor`，是用来专门生成代理对象的组件，他会对所有声明了@Transactional的类进行动态代理。

Spring AOP有一个原则，如果被代理类实现接口，那么使用JDK Proxy的方式进行动态代理；否则使用CGlib的方式进行动态代理。

这个示例中的`TxService`没有实现接口，那么Spring AOP会使用CGlib，CGlib的原理是通过生成一个 Override了被代理类所有方法 的派生类，来实现对方法的拦截的。

CGlib想要能够Override一个package-private的方法，必须要将派生类生成在被代理类所在的包，这对CGlib而言是可以做到的，但它没有那么设计，因为代理模式的理念是“外部”调用内部才会被“拦截”，package-private算是一个内部的方法，所以protected方法不会被拦截，这样的设计与JDK Proxy代理时的结果也是一致的。

#### 怎么解决和避免这个问题？

永远把`@Transactional`声明在一个`public`方法上而不是其他访问权限的方法上。