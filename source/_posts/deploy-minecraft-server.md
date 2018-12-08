---
title: 搭建Minecraft服务器

date: 2018-06-10 06:55:00

updated: 2018-06-10 06:55:00

tags:
- Minecraft

categories: Minecraft

permalink: deploy-minecraft-server
---



## 简介

这篇POST介绍了如何部署一个Minecraft服务器并与好友一起玩耍。

由于Deolin选用阿里云的CentOS作为部署服务器，所以在开始这篇POST之前，最好能对Linux有一些了解。



## 安装Java

首先要为服务器安装Java，可以参照[这里](http://spldeolin.com/posts/centos-softwares/#JDK)。



## Minecraft版本的选择

这里选择了1.8.8，原因是1.8有了粘液块，pureDB材质包也从1.8开始支持3D方块和3D物品。

其他版本也是可以的，只要有对应版本的服务端能够选择即可。



## 客户端与启动器

想要启动Minecraft客户端需要`.minecraft`和启动器，

从[这篇帖子](http://www.mcbbs.net/forum.php?mod=viewthread&tid=38297&page=49&ordertype=1#pid547863)可以下载到这个版本的`.minecraft`，

从[这个GitHub](https://github.com/huanghongxun/HMCL/releases)上可以下载到最新的HMCL启动器。



## 材质包

[Sphax pureDB & 3D item & 3D block](http://bdcraft.net/purebdcraft-minecraft)

>3D item 和 3D block的贴图会变成紫色，非常奇怪
>
>后来发现256x的pureDB不会发生这个问题。
>
>64x也不会发生这个问题，似乎只有128x会



## 1.8.8中文输入补丁

1. 确保客户端集成了`forge`
2. [这个GitHub](https://github.com/zlainsama/InputFix/releases)上可以下载到最新的中文输入补丁。
3. 将补丁放入客户端的`mod`目录下



## 服务端的选择

对于好友之间玩耍的服务器而言，很多认证、防作弊插件都是不需要的，

所以简单来说，如果需要集成`mod`，则选择集成了`forge`的官方`minecraft_server`，

如果不需要集成`mod`，则选择`spigot`。



## spigot服务端

https://getbukkit.org/download/spigot



## 服务端安装forge

spigot暂时没有找到集成forge的方法，所以选择官方的`minecraft_server.jar`

https://mcversions.net/

https://files.minecraftforge.net/maven/net/minecraftforge/forge/index_1.8.8.html

选择`Windows Installer`

运行后选择`Install client`并指定服务端`minecraft_server.jar`所在的目录



## 部署流程

1. 最好在开发环境下`java -jar`一次，生成一下配置文件

2. 第一启动会失败，需要把`eula.txt`的`eula=false`改为`true`

3. `server.properties`常用的配置

   1. 正版验证

      ~~~properties
      online-mode=false
      ~~~

   2. PVP

      ~~~properties
      pvp=false
      ~~~

4. 配置完毕后打包成`tar.gz`上传到服务器

5. 服务器解压后执行命令启动

   ~~~shell
   $ nohup java -jar -Xmx350m /opt/mc-1.8.8-server/spigot-1.8.8-R0.1.jar > /opt/mc-1.8.8-server/server-consolo.log 2>&1&
   ~~~

6. 查看日志

   ~~~shell
   $ tail -f /opt/mc-1.8.8-server/logs/latest.log
   ~~~



## 客户端安装forge

运行`Windows Installer`后选择`Install server`并指定客户端`.minecraft`目录



## mods集成

集成了`forge`之后，绝大多数的`mod`直接放到`mod`目录下就可以了，区别在于有些`mod`服务端也需要集成，有些则不需要，下面是一些简单的经验——



1. 影响地图生成或是服务器行为`mod`需要服务端和客户端都集成，如“更多生物群落”、“精英怪“之类的；
2. 只影响玩家自身的`mod`只需要客户端集成，如”R键整理“、”小地图“之类的。