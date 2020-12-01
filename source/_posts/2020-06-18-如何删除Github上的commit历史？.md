---

title: 如何删除Github上的commit历史？

excerpt: 最好永远没机会用上这个

date: 2020-06-18 17:42

updated: 2020-06-18 17:42

tags:
- Git

categories: Git

permalink: how-to-delete-commit-log

---

## 简介

如果你遇到了以下描述的问题，可以参考这篇POST来解决



## 问题重现

使用Github做版本控制时，不慎将不希望公开的信息commit & push到Github上了，之后虽然马上删了这部分信息并重新commit & push了，但这部分信息一直存在在Github的commit历史中了

现在希望把这次commit的历史记录也删除



## 解决方式

1. 定位到那次不慎操作的commit id，并记录下这个commit id的上一次commit的id

2. 使用git reset命令重置版本

   ~~~shell
   $ git reset --hard [记录下的commit id]
   ~~~

3. 强制push

   ~~~shell
   $ git push origin HEAD --force
   ~~~

4. 完毕。至此，版本被回退到了记录下的那次commit，之后的commit历史全部没有了。



## 需要注意的地方

1. 被“删除”的commit历史不是真正地被删除，真正被删除的是commit与仓库的关联关系，即通过仓库再也无法找到那次commit，但如果保留了commit id，依然能通过地址来访问到那次commit

   ~~~http
   https://github.com/用户名/仓库名/commit/被删除的commit id/
   ~~~

2. 越早发现不慎的commit & push越好，越晚发现代表越多的改动会被回退，期间的commit历史也全部会被删除

