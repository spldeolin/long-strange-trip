title: PageHelper的问题
date: 2017-12-26 12:12:00

tags:

- 报错
- Mybatis

categories: Java

permalink: pagehelper-problem
---

当程序调用`PageHelper.startPage(pageNo, pageSize)`后，分页信息会存在**`ThreadLocal`**中，直到下一次查询statement，分页信息会变成SQL片段拼接到SQL文的后面，然后分页信息被消耗掉，不会影响后续的查询statement。

但是，如果因为程序流程或是异常等原因，导致分页信息没有正确的别消耗掉，那么分页SQL片段甚至会拼到该进程的下一次请求的查询statement中。

~~~
// 如果flag是false，分页语句就会去影响后续结果集了，如果后续结果集本身就有limit语句，那就SQL语法错误了。
PageHelper.startPage(pageNo, pageSize);
if (flag) {
    aaaMapper.listAll();
}
aaaMapper.getFirstOne();
~~~

所以，尽量使startPage()与目标查询语句紧贴在一起，确保分页信息能在希望的地方被使用掉。