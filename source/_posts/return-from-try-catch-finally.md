---
title: try-catch-finally都有return时的返回顺序

date: 2017-05-16 20:12:00

updated: 2017-05-16 20:12:00

tags:
- 

categories: Java

permalink: return-from-try-catch-finally
---



## 简介

这是一道面试题，记录一下



## 示例

写一个简单的示例就很能说明问题了

~~~java
public class Demo {
    
    public static void main(String[] args) {
		System.out.println(mustThrow());
        System.out.println(neverThrow());
    }

    public static int mustThrow() {
        try {
			System.out.println("[mustThrow] try block");
            Integer.valueOf("a");
            return 1;
        } catch (Exception e) {
			System.out.println("[mustThrow] catch block");
            return 2;
        } finally {
			System.out.println("[mustThrow] finally block");
            return 3;
        }
    }

    public static int neverThrow() {
        try {
			System.out.println("[neverThrow] try block");
            Integer.valueOf("1");
            return 1;
        } catch (Exception e) {
			System.out.println("[neverThrow] catch block");
            return 2;
        } finally {
			System.out.println("[neverThrow] finally block");
            return 3;
        }
    }
    
}
~~~



## 演示结果

~~~
[mustThrow] try block
[mustThrow] catch block
[mustThrow] finally block
3
[neverThrow] try block
[neverThrow] finally block
3
~~~



## 总结

1. `finally`代码块中的内容无论如何都会被执行
2. `finally`中的`return`有点过河拆桥的意思，直接不给`try`或是`catch`中`return`语句的执行机会了
3. 日常开发的时候，`finally`最好不写`return`语句，很有可能把预想的返回结果覆盖掉。