---
title: 在CentOS中部署Hexo

date: 2018-03-27 18:09:00

updated: 2018-12-27 17:09:58

tags:
- Hexo
- CentOS

categories: Linux

permalink: hexo-deploy-centos
---

## 事前准备

确保服务器安装了`nginx`、`nodejs`和`git`

~~~shell
$ nginx -v
$ node -v
$ npm -v
$ git --version
~~~



它们安装方式可以参考[这篇POST](https://spldeolin.com/posts/centos-softwares/)



## 安装Hexo

~~~shell
$ sudo npm install hexo-cli -g
$ hexo init your-blog-directory-name
$ cd your-blog-directory-name
$ sudo npm install
~~~



`your-blog-directory-name`是博客的根目录



## 生成静态页面

~~~shell
$ cd your-blog-directory-name
$ hexo g
~~~



至此，静态资源（指`index.html`、css文件和js文件等）生成完毕了，他们都在根目录的`public`目录下



**注意，每次发布新的POST，上传完md文件后都需要重新生成静态页面**



## Nginx配置

静态页面已经生成，可以修改nginx配置映射静态目录了

~~~shell
$ sudo vi /etc/nginx/nginx.conf
~~~

~~~nginx
# 80 server下
root /your-blog-directory-name/public; # 此处填写刚才hexo生成的路径
~~~

~~~shell
$ sudo service nginx reload
~~~



## 安装搜索功能

~~~shell
$ sudo npm install hexo-generator-search --save
$ sudo npm install hexo-generator-searchdb --save
$ hexo g
~~~



## 安装评论功能

评论的实现方式有多种，Deolin参考了一篇网上的文章，并选用`Valine`作为评论功能的实现。

https://www.bluelzy.com/articles/use_valine_for_your_blog.html



## 发布新的POST

每次输入`hexo g`命令后，Hexo都会将`source\_posts`目录下的所有**声明了YAML Front Matter**的md文件，转化为静态页面。

也就就是说，发布一篇新的POST只需要3个步骤

1. 创建一个md文件，声明了`YAML Front Matter`，编写正文，正文支持`Markdown`语法。
2. 将编写完毕的md文件上传到服务器，可以通过github同步，也可以通过sftp。

3. 服务器执行`hexo g`



## YAML Front Matter

`YAML Front Matter`语句块的作用类似于`html`中的`header`标签。

以现在这篇POST为例，它的格式是这样的

~~~markdown
---
title: 在CentOS中部署Hexo

date: 2018-03-27 18:09:00

updated: 2018-12-27 16:54:00

tags:
- Hexo
- CentOS

categories: Linux

permalink: hexo-deploy-centos
---
~~~



其中，

- title 用来声明POST的标题
- date 用来声明POST的初次发布时间
- updated 用来声明POST最近一次的修改时间
- tags 用来声明POST的标签是什么，它支持数组，数组中元素都是等价的

- categories 用来声明POST属于哪个分类，它支持数组，数组中前面的元素代表上级分类

  ~~~
  categories:
  - Java
  - 语法
  ~~~

  代表POST属于“Java”分类的“语法”子分类

  ~~~
  categories:
  - 语法
  - Java
  ~~~

  代表POST属于“语法”分类的“Java”子分类

- permalink 用来声明用于访问这篇POST的URI。示例中的POST需要通过`host:port/posts/hexo-deploy-centos`才能访问到






