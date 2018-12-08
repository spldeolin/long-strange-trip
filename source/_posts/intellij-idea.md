---
title: IntelliJ IDEA

date: 2018-02-01 13:07:44

updated: 2018-02-01 13:07:44

tags: IntelliJ IDEA

categories: 常用软件

permalink: intellij-idea
---

## 简介

[IntelliJ IDEA](https://www.jetbrains.com/idea/ "IntelliJ IDEA")是功能十分强大的Java IDE，在Deolin看来，它比Eclipse更加好用。

这篇POST主要介绍了IntelliJ IDEA这款软件的配置方式，记录下自己在反复配置IDEA过程中，综合下来比较利于开发的配置方式，以及各种的最佳实践。这其中，主要涵盖了`Code Style`，`Inspection`，`Keymap`，`Template`几个方面。

Deolin已经不记得自己重复配置过多少次IDEA了，十分希望这是Deolin最后一次从头整理IDEA的配置了。

## 快速开始

[spldeolin/intellij-idea-config](https://github.com/spldeolin/intellij-idea-config "快速开始")

## 下载安装与第一次启动IDEA

- 下载地址

    https://www.jetbrains.com/idea/download/#section=windows

- 推荐下载便携的zip文件

- 下载完毕后，解压zip文件，本文将解压目录称为`{idea-client}`

- 打开`{idea-client}\bin\idea.properties`，可以看到`idea.config.path`和`idea.system.path`。前者是IDEA所有配置的存放目录，后者是IDEA的缓存目录，本文将它们分别称为`{idea-config}`和`{idea-system}`。

- 与Eclipse不同的是，IDEA的配置文件、软件缓存、代码文件是分离的，而不是一起放在一个“workspace”中。

- 为`{idea-config}`和`{idea-system}`指定合适的路径，不指定的话，在Windows环境下，它们的位置缺省为系统盘的用户文件夹。

  > 指定路径的分割符必须是`/`。`\`的话无法被识别到

- 指定完毕后，启动`{idea-client}\bin\idea.exe`或`{idea-client}\bin\idea64.exe`

- 由于是第一次启动，打算一步步配置IDEA，所以选择`Do not import settings`，进入下一个页面

- 这个页面比较微妙​​，选择`License server`，并输入[一个地址](http://www.cnblogs.com/deolin/p/7647324.html "一个地址")，进入下一个页面。

- 随意选择，因为后续会专门配置样式，进入下一个页面

- Deolin是一个普通的后端开发者，对于下面的页面，这里列出了他的选择。

  大致原则是用不到的、无法大幅度提升效率的不启用。

  > 其实可以不去理会这个页面，因为这里所有插件都可以在工作界面的`File` - `Settings` - `Plugin`中选择是否启用。

  | 项目                | 选择                    |
  | ------------------- | ----------------------- |
  | Java Frameworks     | 只选择 Java EE          |
  | Build Tools         | 只选择 Maven            |
  | Web Development     | 不变动                  |
  | Version Controls    | 只选择 Git              |
  | Test Tools          | 只选择 JUnit            |
  | Application Servers | 只选择 Tomcat and TomEE |
  | Clouds              | 禁用所有                |
  | Swing               | 禁用                    |
  | Android             | 禁用                    |
  | Database Tools      | 不变动                  |
  | Other Tools         | 不变动                  |
  | Plugin Development  | 禁用                    |

  选择完毕后进入下一个页面。

- 什么都不做，Deolin用不到它们，进入下一页面。

- 这个阶段结束了。

## 配置共享与备份

对于配置的共享与备份，IntelliJ IDEA提供了两种方案。

- 使用`Export Settings`，导出成jar文件。

- 使用`Settings Repository`，结合Git进程云同步配置。

可是，这样两者都一个相同缺点——无法将IDEA插件备份下来。想要将配置和插件同时备份下来，最合适的方法可能是将`{idea-config}`文件夹整个上传到Git，每个全新的IDEA都只要将`{idea-config}`指向Git本地仓库就可以了。[快速开始](https://github.com/spldeolin/intellij-idea-config/blob/master/README.md "README")

## 几乎必须的配置

这部分配置的都是现代Java开发必不可少的东西。包括**JDK**、**编码**、**Git**、**Maven**。

- JDK

  `Configure` - `Project Defaults` - `Project Structure` > `Project` > `New` > ...

- 编码

  `Configure` - `Settings...` - `Editor` - `File Encodings` > 三个选择框全都选择 `UTF-8`

- Git

  `Configure` - `Settings...` - `Version Control` - `Git` > … - ...

- Maven

   - Maven客户端

     Deolin不习惯用各种Java IDE自带的Maven，一般都是都从[官网](https://maven.apache.org/download.cgi "官网")下载以后，引入到IDE中的。

      `Configure` - `Settings...` - `Build, Execution, Deployment` - `Build Tools` - `Maven` > 指定 `Maven home directory:`和 `User settings file:`

  - 自动下载依赖的代码和文档

     `Configure` - `Settings...` - `Build, Execution, Deployment` - `Build Tools` - `Maven` - `Importing` > `Automatically download:` - `Import Maven projects automatically`、 `Sources`和 `Documentation`勾选

  - Maven的JDK

    `Configure` - `Settings...` - `Build, Execution, Deployment` - `Build Tools` - `Maven` -  `Importing` > `JDK for importer:`选择之前设置的JDK

  - 打包（mvn install）时不再做单元测试

     `Configure` - `Settings...` - `Build, Execution, Deployment` - `Build Tools` - `Maven` - `Runner` > `Properties` - 勾选 `Skip tests`

## 改善开发体验的习惯配置

这部分配置不是必须的，但基本上都能大幅度提升开发时的体验，它们全部在`Configure` - `Settings...`中进行。

- 启动IDEA后需要选择项目，而不是直接进上次关闭时的项目

  `Appearance & Behavior` - `System Settings` > `Reopen last project on startup`取消勾选

- 直接在新窗口打开新的项目，不再询问

  上一条配置的窗口 > `Open project in new window`选择

- 支持Ctrl+滚轮改变字体大小，投影演示代码时这个功能很有用

  `Editor` - `General` > `Change font size (Zoom) with Ctrl+Mouse Wheel`勾选

- 代码编辑区竖直滚动条上的警告、错误标记加粗，更加明显

  上一条配置的窗口 > `Error stripe mark min height (pixels)`设置为20

- 鼠标移动到类上自动显示JavaDoc

  上一条配置的窗口 > `Show quick documentation on mouse move`勾选

- 自动import

  `Editor` - `General` - `Auto Import` > `Insert imports on paste:`选择 `All`，`Add unambiguous imports on the fly`和 `Optimize imports on the fly (for current project)`勾选

- 消除魔法值前面的“S:”

  `Editor` - `General` - `Appearance` > `Show parameter name hints`取消勾选

- 不再折叠代码（如getter和setter方法）

  `Editor` - `General` - `Code Folding` > `HTML 'style' attribute`和 `One-line methods`取消勾选

- 字体

  `Editor` - `Font` > `Show only monospaced fonts`取消勾选，然后选择需要的字体，强烈推荐 `YaHei Monaco Hybird`和 `Microsoft YaHei Mono`

- 左侧项目树中忽略项目在IDEA、Eclipse下的配置文件、文件夹

  `Editor` - `File Types` > `Ignore files and folders`追加

  ~~~
  .idea;.settings;.classpath;.project;*.iml;
  ~~~

- Debug时，不在代码后面显示值

  `Build, Execution, Deployment` - `Debugger` - `Data Views` > `Show values inline`取消勾选

- 浏览HTML时，右上角不再显示浏览器图标

  `Tools` - `Web Browsers` > `Show browser popup in the editor`取消勾选

## 插件

插件很有必要。这部分的内容，Deolin会不断扩充。
插件的下载页面通过`Configure` - `Plugin` - `Browse repository...`进入

推荐的插件

 - Lombok
 - Mongo Plugin
 - RestfulToolkit
 - ...

------------

**Deolin为至此为止的配置做了备份**
**链接：https://pan.baidu.com/s/1mjwP73U 密码：o77b**

**这此之前的配置，都是分散在`{idea-config}`的各种xml文件中的，这里配置如果日后改坏了，可能会使整个`{idea-config}`都不干净，导致整个IDEA重新配置，所以做个备份；**

**而在此之后的配置，全都会独立地形成一个专门的文件，比如一套Code Styles配置，会在`{idea-config}\codestyles`生成一个xml文件，一套Color Scheme配置生成一个icls文件等等，即便是日后里面的配置被改坏了，也只需要覆盖一个文件，不会影响到`{idea-config}`其他地方**

------------

## Code Style

这部分配置负责**代码格式化策略**，关于Code Style，Deolin同样也推荐使用Google提供的配置[google / styleguide](https://github.com/google/styleguide/blob/gh-pages/intellij-java-google-style.xml)

> 无论哪种，都强烈建议你的开发团队成员使用一套相同Code Style配置。

它们全都在中进行配置

- Java侧（`Settings` > `Editor` - `Code Style` - `Java`）

   - import声明中不出现*号

      `Imports` > `class count to use import with '*'`和 `Names count to use static import with '*'`变更为99

   - import分组排序

      `Imports` > `Import Layout`变更为以下顺序

     ~~~
     import static all other imports
     <blank line>
     import java.*
     import javax.*
     import org.*
     import com.*
     import all other imports
     ~~~

   - 空格和换行

     配置条目实在是太多了，不一一列举出来了。

- 如何引入Code Style配置

  - 下载

    https://github.com/spldeolin/intellij-idea-config/blob/master/config/codestyles/Code%20Style%20By%20Deolin.xml

  - 使用

   - 将Code Style By Deolin.xml移动到`{idea-config}\codestyles`，如果没有这个文件夹，创建它。

   - 启动IDEA

   - 在`Settings` > `Editor` - `Code Style`中选择它。

## Color Scheme

这部分配置负责**IDEA整体的字体/背景颜色、字体粗细等样式**
可以去各种IDEA主题站下载完整的样式策略。
或者，使用Deolin提供的两个样式——**spllight**和**spldark**。
前者是Deolin设计的仿Eclipse样式，后者是Deolin在Lady Night 2基础上做了一些调整的样式

- 如何引入Color Scheme配置
  - 下载

    https://github.com/spldeolin/intellij-idea-config/blob/master/config/colors/spldark.icls

    https://github.com/spldeolin/intellij-idea-config/blob/master/config/colors/spllight.icls

   - 将spllight.icls和spldark.icls移动到`{idea-config}\colors`，如果没有这个文件夹，创建它。

   - 启动IDEA

   - 在`Settings` > `Editor` - `Color Scheme`中选择它们。

## Inspection

这部分配置负责**代码校验策略**，所有条目都在`Settings` - `Editor` - `Inspection`中

- 配置一览

   - 关闭拼写检查

      `Spelling` - `Typo`取消勾选

   - 放宽Unused检查（比如public的类属性，即便现在未被用到，也有很大可能是为将来预留的，没有特别的必要警告它）。

      `Java` - `Declaration redundancy` - `Unused declaration` >

      `Classes`降为`Package-private`

      `Fields`降为`private`

      `Method`降为`private`

   - 不再作类似“Acess can be pageage-private”的警告

      `Java` - `Declaration redundancy` - `Declaration access can be weaker`

   - Mybatis的xml中SQL的背景颜色

      `SQL` - `No data sources configured`和 `SQL dialect detection`取消勾选

   - 不再警告int i = 0; return i;这样的多余声明做法

      `Java` - `Data Flow issues` - `Redundant local variable`取消勾选

   - 实现了Serializable的类没有serialVersionUID时，警告

      `Java` - `Serialization issues` - `Serializable without 'serialVersionUID'`勾选

   - 不再警告@Autowird域未赋值

      `Java` - `Declaration redundancy` - `Unused declaration` > `Entry Points` > `Annotations...` > 在 `Mark field as implicitly written if annotationed by`中追加Spring的 `Autowired`类

- 如何引入Keymap配置
  - 下载

    https://github.com/spldeolin/intellij-idea-config/blob/master/config/inspection/Inspection%20By%20Deolin.xml

   - 将Inspection By Deolin.xml移动到`{idea-config}\inspection`，如果没有这个文件夹，创建它。

   - 启动IDEA

   - 在`Settings` > `Editor` - `Inspection`中选择将Inspection By Deolin。

## Keymap

这部分配置维护的是**快捷键**
Deolin在IDEA提供的默认模板和Ecplise模板上，追加了几个常用快捷键。

- 追加的快捷键一览

   - `alt+F5` Rebuild Project

     重新编译项目

   - `alt+I` Evaluate Expression...

     在DEBUG模式进程被断点阻塞时，计算自定义表达式的值

   - `alt+A` Compare with Lastest Repository Version

     当前文件与Git上最近的版本进行比较

   - `ctrl+S` Rerun

     重新运行/DEBUG项目

   - `ctrl+alt+F5` Pull... 

     Pull代码

   - `ctrl+alt+F6` Commit 

     Commit代码

   - `ctrl+alt+F7` Push... 

     Push代码

 - 如何引入Template配置

   - 下载

     https://github.com/spldeolin/intellij-idea-config/blob/master/config/keymaps/Default%20Keymap%20By%20Deolin.xml

     https://github.com/spldeolin/intellij-idea-config/blob/master/config/keymaps/Eclipse%20Keymap%20By%20Deolin.xml

    - 将Default Keymap By Deolin.xml和Eclipse Keymap By Deolin.xml移动到`{idea-config}\keymaps`，如果没有这个文件夹，创建它。

    - 启动IDEA

    - 在`Settings` > `Keymap`中选择Default Keymap By Deolin或是Eclipse Keymap By Deolin。

## Template

这部分配置维护的是**实时代码模板**

- 常用模板一览

   - `aa`  Spring的Autowired注解

        ~~~
        @Autowired
        private $VAR1$ $VAR2$;
        ~~~

   - `ar` Dubbo的Reference注解

        ~~~
        @Reference
        private $VAR1$ $VAR2$;
        ~~~

   - `av` Spring的Value注解

        ~~~
        @Value("${$VAR1$}")
        private String $VAR2$;
        ~~~

   - `dt` 默认年月日时分秒Pattern

        ~~~
        yyyy-MM-dd HH:mm:ss
        ~~~

   - `dtd` 默认年月日Pattern

        ~~~
        yyyy-MM-dd
        ~~~

   - `dtt` 默认时分秒Pattern

        ~~~
        HH:mm:ss
        ~~~

   - `i ` 打印一条日志

        ~~~
        log.info($VAR$);
        ~~~

   - `l` 快速new一个List

        ~~~
        List<$VAR$> $V$s = new ArrayList<>();
        ~~~

   - `m` 快速new一个Map

        ~~~
        Map<String, $VAR1$> $VAR2$ = new HashMap<>();
        ~~~

   - `main` 快速生成main方法

        ~~~
        public static void main(String[] args) {
            $END$
            System.out.println("临时测试");
        }
        ~~~

   - `o` 使用Java8的Optional类，抛异常

        ~~~
        Optional.ofNullable($END$).orElseThrow(() -> new ServiceException("不存在或是已被删除"));
        ~~~

   - `p` 快速生成System.out.println

        ~~~
        System.out.println($END$);
        ~~~

   - `sb` 快速new一个StringBuilder

        ~~~
        StringBuilder sb = new StringBuilder(400);
        ~~~

   - `t` 快速throw一个ServiceException

        ~~~
        throw new ServiceException($END$);
        ~~~

   - `ser`

        ~~~
         implements Serializable 
        ~~~

   - `ver`

        ~~~
        private static final long serialVersionUID = 1L;
        ~~~

- 如何引入Template配置

   - 下载

     https://github.com/spldeolin/intellij-idea-config/blob/master/config/templates/Template%20By%20Deolin.xml

   - 使用

    - 将Template By Deolin.xml移动到`{idea-config}\templates`，如果没有这个文件夹，创建它。直接可以使用

## 结束

现在，可以把最终的`{idea-config}`目录上传到Github上去了。