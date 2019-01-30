---
title: macOS常用环境、软件

date: 2019-02-04 17:01

updated: 2019-02-04 17:48

tags:
- 总结
- MySQL
- Redis
- MongoDB
- Nginx
- Git
- macOS
- Python
- 7Zip

categories: macOS

permalink: macos-softwares
---



## 简介

事情的起因是Deolin买了人生中的第一台MacBook Pro...



## Homebrew

~~~shell
$ ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"
~~~

macOS的brew相当于centOS的yum，ubuntu的apt。



## 一般软件

### 7 Zip

~~~shell
$ brew cask install p7zip
~~~

*顺便写一下常用命令*



### 浏览器

google chrome



### Office相关

iwork或是mac版office

OneDrive

OneNote



### 思维导图

mindnode



### Markdown编辑

typora



### 视频播放

Iina 



### 音乐播放

网易云音乐



### 通讯

微信

QQ

钉钉



### PDF阅读器

Skim



### 截图

Snipaste



### 下载工具

Aria2



### 其他

Alfred 似乎比spotlight好用

[bilibili-mac-client](https://github.com/typcn/bilibili-mac-client/releases) B站用户开发的客户端，占用和发热比用chrome好很多   

Padbury 桌面屏保时钟



## 开发用软件

### 文档查询

dash



### 版本管理

git

sourcetree



### IDE

intellij-idea

visual-studio-code



### DB图形化

navicat

medis



### web开发

postman



## 开发用环境

### Oracle JDK

[下载地址](https://www.oracle.com/technetwork/java/javase/downloads/jdk8-downloads-2133151.html)

由于开源协议的缘故，需要个人用户去Oracle手动下载，并配置环境变量。



### Python 3

1. 安装

   ~~~shell
   $ xcode-select --install
   $ brew install python
   ~~~

2. 安装完毕

   ~~~shell
   $ python --version                     
   $ pip --version
   $ python2 --version
   $ python3 --version
   ~~~



### MySQL

~~~shell
$ brew install mysql
~~~



### NoSQL

1. Redis

   ~~~shell
   $ brew install redis
   ~~~

2. MongoDB

   ~~~shell
   $ brew install mongodb
   $ brew install mongodb --devel
   ~~~


