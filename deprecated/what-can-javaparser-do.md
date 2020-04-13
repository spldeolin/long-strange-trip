---

title: Javaparse能做的事情

date: 2019-11-08 11:03

updated: 2019-11-08 11:03

tags:
- Java

categories: Java

permalink: what-can-javaparser-do

---

1. 扫描具有一些共性的类/方法，使它们变得统一和规范

   例如：让每个@Data类中的field统一使用包装类型

2. 扫描符合一定“协议”的独立注释，生成代码

   例如：生成Spring MVC handler，顺便生成Service

   例如：基于Mybatis mapper方法上的特定注释，生成参数返回值DTO以及xml statement

3. 扫描出不符合项目规约的代码，报告出来

   例如：为项目规约中的具体条目写一个VoidVisitor，指定范围即插即用