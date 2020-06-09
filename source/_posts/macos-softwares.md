---
title: Mac OS常用环境、软件

date: 2019-02-14 13:39

updated: 2020-06-09 16:13

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



## Homebrew

~~~shell
$ ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"
~~~



## 一般软件

### Alfred （代替spotlight）

~~~shell
$ brew cask install alfred
~~~



### Magnet （快速调整当前窗口的大小和位置）



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



### Microsoft Office 2019



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



### iStat Menus （硬件指标实时监控）



### Irvue （壁纸）

通过App Store安装



### Word Clock （屏保）



###  Inpaint



### Bartender （管理Mac OS顶部菜单栏的图标）



## 开发用软件

### iTerm （代替Mac OS自带的终端）

~~~shell
$ brew cask install iterm2
~~~



## Zsh、Oh my zsh

~~~shell
$ zsh --version
$ chsh -s /bin/zsh
$ sh -c "$(curl -fsSL https://raw.github.com/robbyrussell/oh-my-zsh/master/tools/install.sh)"
~~~



###  Git

~~~shell
$ brew install git 
~~~



### Maven

~~~shell
$ brew install maven
~~~



### VisualVM

~~~shell
$ brew cask install visualvm
~~~



### Sourcetree

~~~shell
$ brew cask install sourcetree
~~~



### Dash

~~~shell
$ brew cask install dash
~~~



### Jetbrains

~~~shell
$ brew cask install jetbrains-toolbox
~~~



### Visual Studio Code

~~~shell
$ brew cask install visual-studio-code
~~~



### Postman

~~~shell
$ brew cask install postman
~~~



### Medis



### TextLab



### Oracle JDK

https://www.oracle.com/java/technologies/javase-downloads.html

~~~shell
$ vi ~/.zshrc

source ~/.bash_profile;

$ source ~/.bash_profile
~~~



### Python 3、pip

https://www.python.org/downloads/



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

