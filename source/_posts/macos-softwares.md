---
title: macOS常用环境、软件

date: 2019-02-14 13:39

updated: 2019-02-14 13:39

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

macOS一般通过以下3个方式安装软件

- Homebrew

- App Store
- dmg文件

brew相当于centOS的yum，ubuntu的apt。



## 一般软件

### Alfred

~~~shell
$ brew cask install alfred
~~~



### 7 Zip

~~~shell
$ brew cask install p7zip
~~~



### Google Chrome 

~~~shell
$ brew cask install google-chrome
~~~



### OneDrive

~~~shell
$ brew cask install onedrive
~~~



### OneNote

通过App Store安装



### Office

*TODO*



### 思维导图

~~~shell
$ brew cask install mindmaster
~~~



### Markdown

~~~shell
$ brew cask install typora
~~~



### 视频播放器

~~~shell
$ brew cask install iina
~~~

 

### 网易云音乐

~~~shell
$ brew cask install neteasemusic
~~~



### QQ

~~~shell
$ brew cask install qq
~~~



### 微信

~~~shell
$ brew cask install wechat
~~~



### 钉钉

~~~shell
$ brew cask install dingtalk
~~~



### PDF阅读器

~~~shell
$ brew cask install skim
~~~



### Snipaste

~~~shell
$ brew cask install snipaste
~~~



### 迅雷

~~~shell
$ brew cask install thunder
~~~



### 百度网盘

~~~shell
$ brew cask install baidunetdisk
~~~



### 小狼毫

~~~shell
$ brew cask install squirrel
~~~



## 开发用软件

### 文档查询

~~~shell
$ brew cask install dash
~~~



### 终端

~~~
$ brew cask install iterm2
~~~



###  Git

~~~shell
$ brew install git 
~~~



### Sourcetree

~~~shell
$ brew cask install sourcetree
~~~



### IntelliJ IDEA

~~~shell
$ brew cask install intellij-idea
~~~



### VS Code

~~~shell
$ brew cask install visual-studio-code
~~~



### Navicat

~~~shell
$ brew cask install navicat-premium
~~~



### Redis Desktop Manager

通过从[Github](https://github.com/uglide/RedisDesktopManager/releases?after=0.8.1)下载的dmg文件安装



### Postman

~~~shell
$ brew cask install postman
~~~



## 开发环境

### Oracle JDK

通过从[官网](https://www.oracle.com/technetwork/java/javase/downloads/jdk8-downloads-2133151.html)下载的dmg文件安装



### Python 3 和 pip

1. 安装

   ~~~shell
   $ brew install python3
   $ curl https://bootstrap.pypa.io/get-pip.py -o get-pip.py
   $ python3 get-pip.py
   ~~~

2. 安装完毕

   ~~~shell
   $ python --version
   $ pip --version
   $ python3 --version
   $ pip3 --version
   ~~~



### MySQL

~~~shell
$ brew install mysql
~~~



### Redis

~~~shell
$ brew install redis
~~~



### Mongo

~~~shell
$ brew install mongodb
~~~



## 其他

### 键盘映射

~~~shell
$ brew install karabiner-elements
~~~

使键盘默认不按下`fn`键