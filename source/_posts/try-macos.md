---
title: 初次使用macOS时遇到的各种问题

date: 2019-02-21 11:10

updated: 2018-02-21 11:10

tags:
- 配置

categories: macOS

permalink: try-macos
---

## 简介

这篇POST将会介绍Deolin使用macOS时的一些偏好。

这里将会不断追加新内容，请善用检索功能



## 改变macOS的桌面壁纸

1. alfred打开“桌面与屏幕保护程序“
2. 选择”桌面“选项卡
3. 点**+**添加壁纸文件或是文件夹

![](/images/try-macos-01.png)



##  改变用户头像

1. alfred打开“用户与群组”
2. 选择需要改变头像的用户，将头像图片从Finder拖移到原头像上

![](/images/try-macos-02.png)



## 切换输入法的快捷键

1. alfred打开“键盘”
2. 选择“快捷键”选项卡
3. “输入法”中的“选择上一个输入法”就是该功能

![](/images/try-macos-03.png)



## macOS下Google Chrome的快捷键

| 组合键   | 说明           |
| -------- | -------------- |
| ⌃ Tab    | 切换到左侧标签 |
| ⌃⇧   Tab | 切换到右侧标签 |
| ⌥⌘ J     | 调试模式       |



## macOS如何锁定屏幕

按键为`⌃⌘Q`

注意不要于QQ for mac的截图功能的全局快捷键冲突

可以在QQ的偏好设置中改变截图的快捷键

![](/images/try-macos-04.png)



## IntelliJ IDEA for mac 的各种路径

| 名称     | 说明                                             |
| -------- | ------------------------------------------------ |
| 配置目录 | ~/Library/Preferences/<PRODUCT><VERSION>         |
| 缓存目录 | ~/Library/Caches/<PRODUCT><VERSION>              |
| ⌥⌘ J     | ~/Library/Application Support/<PRODUCT><VERSION> |
| 日志目录 | ~/Library/Logs/<PRODUCT><VERSION>                |



## 启用触控板的拖移功能

1. alfred打开“辅助功能”
2. 选择“鼠标与触控板”选项卡
3. 打开“触控板选项”

![](/images/try-macos-05.png)



## iTerm 2 集成zsh和oh my zsh

1. macOS一般默认安装了zsh

   ~~~shell
   $ zsh --version
   ~~~

2. 修改iTerm 2 的终端为zsh

   ~~~shell
   $ chsh -s /bin/zsh
   ~~~

3. 安装oh my zsh

   ~~~shell
   $ sh -c "$(curl -fsSL https://raw.github.com/robbyrussell/oh-my-zsh/master/tools/install.sh)"
   ~~~



## 微信聊天记录的备份目录

~~~shell
~/Library/Containers/com.tencent.xinWeChat/Data/Library/Application\ Support/com.tencent.xinWeChat
~~~



## 一般macOS软件 到顶部、到底部 的快捷键

| 组合键 | 说明   |
| ------ | ------ |
| ⌘ ↑    | 到顶部 |
| ⌘ ↓    | 到底部 |



## 禁用rootless

rootless可以理解为macOS防止用户误操作的保护措施。

例如，开启rootless的情况下，用户即便是sudo，也无法删除系统根目录。

rootless默认是开启的。

关闭方式可以参考[这篇文章](https://blog.csdn.net/jcl314159/article/details/82710452)。