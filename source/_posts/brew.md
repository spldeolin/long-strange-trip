---
title: Homebrew 常用命令

date: 2019-02-14 15:16

updated: 2019-02-15 10:41

tags:
- Homebrew

categories: macOS

permalink: brew

---

## 简介

经历了若干次尝试，Deolin已经掌握了如何在macOS安装软件了。

通过Homebrew大概是安装软件最方便的方式了，这篇POST将会介绍Brew相关的命令。



## brew相关命令

1. 搜索 受Homebrew支持的软件

   ~~~shell
   $ brew search keyword-to-search
   ~~~

   

   如果返回`No formula or cask found for …`，可能代表关键词错误，例如，阿里的钉钉在Homebrew的软件名为`dingtalk`，而不是`dingding`

   ~~~shell
   $ brew search dingding
   No formula or cask found for "dingding".
   
   $ brew search dingtalk
   ==> Casks
   dingtalk ✔
   
   $ brew search ding
   ==> Formulae
   glbinding                               schroedinger
   ==> Casks
   dingtalk ✔                folding-at-home           loading
   findings                  foldingtext
   ~~~



​	**`brew search`的检索范围包含了cask模块**



2. 安装软件

   ~~~shell
   $ brew install software-name
   ~~~

   `brew install`相比`brew search`的不同之处在于，`brew install`必须提供完整的软件名，如果软件不受Homebrew支持，则报错。



3. 查看所有通过`brew install`安装的软件

   ~~~ shell
   $ brew list
   ~~~



4. 查看通过`brew install`安装的软件的详细信息

   ~~~shell
   $ brew info software-name
   ~~~

   能够查看版本号、依赖的软件、与自身冲突的软件等信息



5. 更新通过`brew install`安装的软件

   ~~~shell
   $ brew upgrade software-name
   ~~~

   

6. 卸载通过`brew install`安装的软件

   ~~~shell
   $ brew uninstall software-name
   ~~~



7. 更新Homebrew自身

   ~~~shell
   $ brew update
   ~~~



8. 清除缓存

   ~~~shell
   $ brew cache clean
   ~~~

   

## brew cask 相关命令

cask可以认为是Homebrew的一个模块

`brew cask`开头的命令一般管理的是带有GUI的软件（如Navicat、Sourcetree等）

而`brew`开头的命令一般管理的是无GUI软件或是各种环境（如MySQL、Git等）



1. 安装软件

   ~~~
   $ brew cask install software-name
   ~~~



2. 查看所有通过`brew cask install`安装的软件

   ~~~shell
   $ brew cask list
   ~~~



3. 查看通过`brew cask install`安装的软件的详细信息

   ~~~shell
   $ brew cask info software-name
   ~~~

   

4. 更新通过`brew cask install`安装的软件的详细信息

   ~~~shell
   $ brew cask upgrade software-name
   ~~~

   

5. 卸载通过`brew cask install`安装的软件的详细信息

   ~~~shell
   $ brew cask uninstall software-name
   ~~~



## brew serivces 相关命令

`brew services`开头的命令一般是用于管理通过Homebrew安装的各类服务的。

1. 查看所有服务及其它们各自的运行状态

   ~~~sh
   $ brew services list
   ~~~



2. 启动服务

   ~~~shell
   $ brew services start redis
   ~~~

   ~~~shell`
   $ brew services start --all
   ~~~
   
   
   ~~~

3. 重启服务

   ~~~shell
   $ brew services restart redis
   ~~~

   ~~~shell
   $ brew services restart --all
   ~~~

   

4. 停止服务

   ~~~shell
   $ brew services stop redis
   ~~~

   ~~~shell
   $ brew services stop --all
   ~~~

   

