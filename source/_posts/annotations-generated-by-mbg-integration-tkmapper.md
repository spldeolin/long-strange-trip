title: 关于集成通用mapper的Mybatis代码生成器产生的model类注解
date: 2018-01-11 14:43:00

tags: Mybatis

categories: Java

permalink: annotations-generated-by-mbg-integration-tkmapper
---

主要是@Table、@Id、@GeneratedValue、@Column这4个注解。

这四个注解都来自javax.persistence包，是Java持久层规范，单纯的Mybatis并不认识这四个注解。

> @Table("basic_user") 代表db表的表名会映射到这个Java类名，即便类名与表名不一致

> @Id 代表所映射的db字段是主键

> @GeneratedValue(strategy = GenerationType.IDENTITY) 代表所映射的db字段是自增的，且每次insert操作完毕后，自增值都会绑定到这个属性上

> @Column(name="xx") 代表db中的xx字段会映射到这个Java属性上，即便属性名与字段名不一致

Mybatis集成tk.mapper之后，这4个注解会发挥各自的作用。

换句话说，不需要去管它们，它们都是由mybatis代码生成器自动生成，为通用Mapper接口提供支持的，不影响编写其他的`statement`。