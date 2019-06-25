---

title: 7Zip命令

date: 2019-02-25 15:08

updated: 2019-03-01 13:09

tags:
- 7Zip

categories: Mac OS

permalink: 7zip-command

---

## 简介

这篇POST将会介绍一些7Zip的命令



## 解压

~~~shell
$ 7z x your-achievement.7z
~~~

这个命令演示了将`your-achievement.7z`解压到当前目录



## 压缩

~~~shell
$ 7z a -pPASSWORD folder-to-achieve/ your-achievement.7z   
~~~

这个命令演示了将`folder-to-achieve`文件夹压缩到`your-achievement.7z`，并将解压密码设置为`PASSWORD`



## 分卷压缩

~~~shell
$ 7z a -v512m large-folder/ large-achievement.7z
~~~

这个命令演示了将`large-folder`文件夹进行分卷压缩，生成`large-achievement.7z.001`、`large-achievement.7z.002`……每个分卷最大`512M`
