---
title: Mac OS常用环境、软件

date: 2019-02-14 13:39

updated: 2019-07-05 17:54

tags:
- 总结
- MySQL
- Redis
- MongoDB
- Nginx
- Git
- Mac OS
- Python
- 7Zip

categories: Mac OS

permalink: macos-softwares
---



## 简介

事情的起因，是Deolin买了人生中的第一台MacBook Pro...



## 说明

名字前面出现*****的软件，代表以下情况的其中一种

- 这个软件在App Store需要付费购买，如Microsoft Office

- 这个软件在购买正版前只能试用若干天，如iStat Menus
- 这个软件本身是免费的，但购买正版或拓展包后能大幅度提高效率，如Alfred 3



## Homebrew

~~~shell
$ ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"
~~~

Mac OS一般通过以下3个方式安装软件

- Homebrew

- App Store
- dmg文件

Homebrew相当于centOS的yum，ubuntu的apt。



## 一般软件

### * Alfred 3 （代替spotlight）

~~~shell
$ brew cask install alfred
~~~



### * Magnet （快速调整当前窗口的大小和位置）

通过App Store安装



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



### * Microsoft Office 2019

通过App Store安装



### Xmind ZEN

~~~shell
$ brew cask install xmind-zen
~~~



### Typora

~~~shell
$ brew cask install typora
~~~



### IINA （视频播放器）

~~~shell
$ brew cask install iina
~~~

 

### 网易云音乐

~~~shell
$ brew cask install neteasemusic
~~~



### 有道词典

~~~shell
$ brew cask install youdaodict
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



### Skim （PDF阅读器）

~~~shell
$ brew cask install skim
~~~



### Snipaste （截图工具）

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



### * iStat Menus （硬件指标实时监控）

~~~shell
$ brew cask install istat-menus
~~~



### * Timing 2 （统计使用每种软件的时间，分析工作效率）

~~~shell
$ brew cask install timing
~~~



### Irvue （壁纸）

通过App Store安装



### * Word Clock （屏保）

通过从[官网](https://www.simonheys.com/wordclock/)下载dmg安装



###  * Inpaint （消除图片水印）

通过从[官网](https://www.theinpaint.com/)下载dmg安装



### Bartender （管理Mac OS顶部菜单栏的图标）

~~~shell
$ brew cask install bartender
~~~



### Kawa （为每个输入法设置快捷键，方便切换）

~~~shell
$ brew cask install kawa
~~~



## 开发用软件

### iTerm 2 （代替Mac OS自带的终端）

~~~
$ brew cask install iterm2
~~~



###  Git

~~~shell
$ brew install git 
~~~



### GitKraken （Git客户端）

~~~shell
$ brew cask install gitkraken
~~~



### Dash

~~~shell
$ brew cask install dash
~~~



### Jetbrains

~~~shell
$ brew cask install jetbrains-toolbox
$ brew cask install intellij-idea
$ brew cask install pycharm
$ brew cask install datagrip
~~~



### Visual Studio Code

~~~shell
$ brew cask install visual-studio-code
~~~



### * Navicat

~~~shell
$ brew cask install navicat-premium
~~~



### * Medis

通过App Store安装



### Postman

~~~shell
$ brew cask install postman
~~~



### OpenJDK

~~~shell
$ brew cask install adoptopenjdk
~~~



### Python 3、pip

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



### 各种服务与中间件

~~~bash
$ brew install elasticsearch
$ brew install kibana
$ brew install logstash
$ brew install mongodb
$ brew install mysql
$ brew install rabbitmq
$ brew install redis
$ brew install zookeeper
$ brew install docker
~~~



### sshpass

~~~shell
$ brew install https://raw.githubusercontent.com/kadwanev/bigboybrew/master/Library/Formula/sshpass.rb
~~~

> 直接使用`brew install sshpass`会报错，因为这个软件会令人养成不好的安全习惯，所以需要通过github地址安装



## 字体

[Microsoft YaHei Mono](https://www.onlinewebfonts.com/download/9798f64007ae3426b2336e57dae4149c)



## 备份

Deolin将清单中通过brew命令导出成了[Brewfile](https://github.com/spldeolin/mac-os-settings/blob/master/Brewfile)了，方便使用。

