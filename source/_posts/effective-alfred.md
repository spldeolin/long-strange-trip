---
title: 使Alfred更加高效

date: 2019-02-21 11:37

updated: 2019-02-21 11:37

tags:
- 配置

categories: Mac OS

permalink: effective-alfred
---

## 简介

Alfred是个非常高效的工具，用过Mac OS的都懂



## 改变默认搜索引擎

![](/images/effective-alfred-01.png)



##  网易云音乐控制器

<https://github.com/li-xinyang/AW_NeteaseAlfredController>

> 这里需要利用Alfred 的workflow功能



## 使用iTerm 2作为默认终端

1. alfred打开“alfred”
2. 选择“Terminal / Shell”标签卡
3. “Application: ”选择“Custom”

4. 在下方输入以下脚本

   ~~~
   on alfred_script(q)
   	if application "iTerm2" is running or application "iTerm" is running then
   		run script "
   			on run {q}
   				tell application \":Applications:iTerm.app\"
   					activate
   					try
   						select first window
   						set onlywindow to false
   					on error
   						create window with default profile
   						select first window
   						set onlywindow to true
   					end try
   					tell current session of the first window
   						if onlywindow is false then
   							tell split vertically with default profile
   								write text q
   							end tell
   						end if
   					end tell
   				end tell
   			end run
   		" with parameters {q}
   	else
   		run script "
   			on run {q}
   				tell application \":Applications:iTerm.app\"
   					activate
   					try
   						select first window
   					on error
   						create window with default profile
   						select first window
   					end try
   					tell the first window
   						tell current session to write text q
   					end tell
   				end tell
   			end run
   		" with parameters {q}
   	end if
   end alfred_script
   ~~~