---
title: Windows 常用软件

date: 2018-02-14 10:00:39

updated: 2019-01-14 10:36:00

tags:
- 总结

categories: Windows

permalink: windows-softwares
---

## 简介

Deolin把自己在Windows下常用软件全部整理出来，作成了这份清单，用于分享与备忘。

这篇POST将会不断更新。



## 普通软件

- 压缩工具

  [7-zip](http://www.7-zip.org/ "7-zip")

- VPN

  [Lantern](https://github.com/getlantern/forum "Lantern")

- 多tab资源管理器

  [XYplorer](https://www.xyplorer.com/)

- 文件搜索

  [Everything](https://www.voidtools.com/downloads/ "Everything")

- 办公软件

  [Office 2013](ed2k://|file|SW_DVD5_Office_Professional_Plus_2013_64Bit_ChnSimp_MLF_X18-55285.ISO|958879744|678EF5DD83F825E97FB710996E0BA597|/ "Microsoft Office 2013 Pro Plus（VOL版）简体中文 64位")

- 图片查看

  [XnView](https://www.xnview.com/en/xnview/#downloads "XnView")

- 下载工具

  [百度网盘](https://pan.baidu.com/download "百度网盘")

  [迅雷极速版366](https://pan.baidu.com/s/1dGqYiLN "迅雷极速版366")　　　　　密码：s9uq

- 云同步

  [OneDrive](https://onedrive.live.com/about/zh-cn/download/ "OneDrive")

- 垃圾清理

  [CCleaner](https://www.ccleaner.com/ccleaner/download "CCleaner")

- 网页浏览器

  [Google Chrome](https://www.google.com/chrome/?system=true&standalone=1 "Google Chrome")

- 文本编辑

  [Notepad++](https://notepad-plus-plus.org/download/ "Notepad++")

- 音乐播放器

  [网易云音乐](https://music.163.com/#/download "网易云音乐")

- 视频播放器

  [PotPlayerSetup](https://potplayer.daum.net/ "PotPlayerSetup") （建议都用32bit）

- 即时通讯

  [QQ Light](http://dldir1.qq.com/qqfile/qq/QQ6.7Light/13466/QQ6.7Light.exe)

  [微信](https://weixin.qq.com/cgi-bin/readtemplate?uin=&stype=&promote=&fr=&lang=zh_CN&ADTAG=&check=false&nav=download&t=weixin_download_list&loc=readtemplate,weixin,body,6 "微信")

- PDF阅读器

  [SumatraPDF](https://www.sumatrapdfreader.org/download-free-pdf-viewer.html "SumatraPDF")

- 拼音输入法

  [RIME](http://rime.im/download/ "RIME")

- 截图软件

  [Snipaste](https://zh.snipaste.com/) （类似QQ截图，功能比QQ丰富、好用，支持拾色，不支持滚动截图和GIF录制。）

  [GifCam](http://blog.bahraniapps.com/gifcam/#download)

- 词典

  [有道词典](http://cidian.youdao.com/multi.html "有道词典")

- 思维导图

  [XMind ZEN](https://www.xmind.cn/)

- 小工具

  [Hash](http://www.keir.net/hash.html "Hash") 文件校验



## 开发用软件

- SDK

  [Oracle JDK 8](http://www.oracle.com/technetwork/java/javase/downloads/jdk8-downloads-2133151.html "Oracle JDK 8")

  [OpenJDK 8](https://developers.redhat.com/products/openjdk/download/ "OpenJDK 8")

  [Python 3](https://www.python.org/downloads/windows/)

- IDE

  [IntelliJ IDEA 2018.3](https://www.jetbrains.com/idea/download/#section=windows)

  [Rover12421](https://plus.google.com/+Rover12421)

- VCS

  [Git](https://github.com/git-for-windows/git/releases "Git") (MinGit)

  [TortoiseGit](https://tortoisegit.org/download/ "TortoiseGit")

- 数据库图形化工具

  [Navicat 12 Premium](https://www.navicat.com/en/download/navicat-premium)

  [Navicat Keygen](https://github.com/Deltafox79/Navicat_Keygen/releases)

  [Medis](https://github.com/x2jia/medis/releases/tag/win "Medis")

- Linux

  [Xshell 和 Xftp](https://www.netsarang.com/en/free-for-home-school/)

- 文本比较

  [WinMerge](http://winmerge.org/downloads/ "WinMerge")

- Markdown

  [Typora](https://typora.io/)

- Web

  [Postman](https://www.getpostman.com/ "Postman")

- 虚拟环境

  [VirtualBox](https://www.virtualbox.org/wiki/Downloads)

  [Vagrant](https://www.vagrantup.com/downloads.html)



## 开发配置

https://github.com/spldeolin/java-development-config



## 系统安装相关软件

- PE、驱动、Windows补丁、.net库、vc++库、DX9.0

  [IT天空](https://www.itsk.com/)

- Win10、Win7官方镜像

  [MSDN, 我告诉你](https://msdn.itellyou.cn/)



## Win7

- 下载工具

  [迅雷极速版256](https://pan.baidu.com/s/1jKaPmdS "迅雷极速版256")　　　　　	密码：fl4o

- Win7杀毒软件

  [MSE](https://support.microsoft.com/zh-cn/help/14210/security-essentials-download "MSE")




## 其他技巧

- Win10 卸载自带软件

  - 管理员运行powershell

  - 执行命令`Get-AppxPackage -allusers | Remove-AppxPackage`

- 迅雷边下边播

  - 在安装目录的program目录下新建xmp.ini

  - ~~~ini
    [global]
    Path=C:\Program Files (x86)\DAUM\PotPlayer\PotPlayerMini.exe
    ~~~
