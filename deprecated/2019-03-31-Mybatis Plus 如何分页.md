---
title: Mybatis Plus 如何分页

date: 2019-03-31 10:24

updated: 2019-03-31 10:24

tags:
- Mybatis

categories: Java

permalink: mybatis-plus-page
---

过时原因：教学类

## 简介

Mybatis Plus提供了2种分页方式。



## 启用分页功能

在使用分页之前，需要先注册一个Bean以启动分页功能

~~~java
    @Bean
    public PaginationInterceptor paginationInterceptor() {
        return new PaginationInterceptor();
    }
~~~



## 方式一

利用各种Wrapper结合BaseMapper的方式

~~~java
// mapper interface
public interface UserMapper extends BaseMapper<UserEntity> {
}
~~~

~~~java
// service
    Page<UserEntity> pageParam = new Page<>(1, 2); // 当前页码，每页条数

    LambdaQueryWrapper<UserEntity> query = new LambdaQueryWrapper<>();
    query.gt(UserEntity::getMobile, "1");

    IPage<UserEntity> pageResult = userMapper.selectPage(pageParam, query);
    return pageResult;
~~~



## 方式二

开发者自写Statement的方式

~~~java
// mapper interface
public interface UserMapper {
    IPage<UserEntity> searchAsPageByMobile(Page<UserEntity> param, @Param("mobile") String mobile);
}
~~~

~~~xml
<!-- mapper xml -->
    <select id="searchAsPageByMobile" result="UserEntity">
        select * from user where is_deleted = false and mobile = #{mobile}
    </select>
~~~

~~~java
// service
    Page<UserEntity> pageParam = new Page<>(1, 2); // 当前页码，每页条数

    IPage<UserEntity> pageResult = userMapper.listAsPage(pageParam, "999");
    return pageResult;
~~~



> 代码中出现过的`IPage`类的全限定名是`com.baomidou.mybatisplus.core.metadata.IPage`
>
> `Page`类的全限定名是`com.baomidou.mybatisplus.extension.plugins.pagination.Page`