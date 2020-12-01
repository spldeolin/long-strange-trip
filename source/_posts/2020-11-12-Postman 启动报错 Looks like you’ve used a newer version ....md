---

title: Postman 启动报错 Looks like you’ve used a newer version ...

excerpt: 记录一个软件的非自然问题

date: 2020-11-12 08:34

updated: 2020-11-12 08:34

tags:
- 报错

categories: Mac OS

permalink: postman-launch-error

---

## 简介

今天碰到了使用Homebrew升级Postman之后，再次启动发生报错的情况，详细错误信息如下

> Looks like you’ve used a newer version of the Postman app on this system. Please download the latest app and try again.



## 解决方式

1. 卸载Postman

   ~~~shell
   brew uninstall postman
   ~~~

2. 删除Library目录下的Postman目录

   ~~~shell
   cd ~/Library/Application\ Support
   rm -rf Postman
   ~~~

3. 重新安装Postman

   ~~~shell
   brew install postman --cask
   ~~~

问题解决，系统版本是`Mac OS 10.15.7`