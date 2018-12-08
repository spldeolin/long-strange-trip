---
title: 在CentOS中部署Hexo

date: 2018-03-27 18:09:00

updated: 2018-03-27 18:09:00

tags:
- Hexo
- CentOS

categories: Linux

permalink: hexo-deploy-centos
---



## 环境

一个全新的阿里云ECS，操作系统是CentOS 7.4。



## 安装软件

- 通过Xshell进入终端。

- 更新yum

  ~~~shell
  $ sudo yum update
  ~~~

- 安装Nodejs与npm

  - 官网

    https://nodejs.org/en/download/current/

  - 解压

    ~~~shell
    $ tar xvJf node-v9.9.0-linux-x64.tar.xz
    ~~~

  - 创建快捷方式

    ~~~shell
    $ ln -s /opt/node-v8.10.0-linux-x64/bin/node /usr/local/bin/node
    $ ln -s /opt/node-v8.10.0-linux-x64/bin/npm /usr/local/bin/npm
    ~~~

  - 测试一下

    ~~~shell
    $ node -v
    ~~~

  ​

  ## 初始化Hexo

  ~~~shell
  $ npm install hexo-cli -g
  ~~~

  > 这一步会非常慢

  ~~~shell
  $ hexo init your-blog
  ~~~

  >your-blog可以替换为其他名称，它将作为博客的目录名

  ~~~shell
  $ cd your-blog
  $ npm install
  $ hexo s -p8080
  ~~~

  > 8080可以替换为其他端口号

  至此，博客可以通过域名+端口号访问了，但是比较不建议使用`$ hexo s`的方式启动服务，因为这个做法的性能比不上Nginx。

## 部署博客

- 生成静态文件

  ~~~shell
  $ hexo g
  ~~~

  至此，所有的静态文件都生成在`your-blog/public`中了，只要用Nginx反向代理这个目录就可以了。


- 安装Nginx

  ~~~shell
  $ sudo yum install nginx
  $ systemctl start nginx
  ~~~

- 部署Hexo

  ~~~shell
  $ vim /etc/nginx/nginx.conf
  ~~~

  把root后面的路径改为`public`的全路径后，重启Nginx。

  ~~~shell
  $ systemctl restart nginx
  ~~~

  至此，博客可以通过域名访问了。（Niginx默认监听80端口）

## 写文章

- 新建文章

    - 命令行

        ~~~shell
        $ hexo new nice-to-meet-u
        ~~~

        > nice-to-meet-u可以替换为其他名称，它将作为新文章的标题。

    - 非命令行方式

        可以在`your-blog/public/_posts`直接新建一个md文件，手动写上Front-matter。

- Front-matter

    新建文章的md文件中，往往会初始化以下这样的内容

    ~~~markdown
    ---
    title: nice-to-meet-u
    date: 2018-03-28 08:26:28
    tags:
    ---
    ~~~

    这些内容在Hexo中被称为文章的Front-matter。

    Front-matter用于声明文章的一些参数，官网预提供了以下的参数

| 参数       | 描述                             | 默认值                      |
| :--------- | :------------------------------- | :-------------------------- |
| layout     | 布局                             |                             |
| title      | 文章的标题                       |                             |
| date       | 创建日期（年月日时分秒）         | 敲下`$ hexo new` 命令的时间 |
| updated    | 更新日期（年月日时分秒）         |                             |
| comments   | 是否开启评论（true/false）       | true                        |
| tags       | 标签（可多个，顺序无意义）       |                             |
| categories | 标签（可多个，顺序代表层次先后） |                             |
| permalink  | 文章显示在URL上的标题            |                             |

- 发布文章

  重新生成静态文件

  ~~~shell
  $ hexo g
  ~~~

  至此，发布完成。

  如果生产环境显示的不是最新的内容，尝试先清理一下静态文件

  ~~~shell
  $ hexo clean
  $ hexo g
  ~~~

## 中文标题问题

以以下命令行为例

~~~shell
$ hexo new 你好啊
~~~

产生的md文件会以`你好啊`作为文件名，这在linux下会显示为乱码。

比较合适的做法是使用中文标题的英译作为`title`来新建文件，如下所示

~~~shell
$ hexo new nice-to-meet-u
~~~

并在Front-matter中，指定`title`为原先想定的中文标题，指定`permalink`为文件名，如下所示

~~~markdown
---
title: 你好啊
date: 2018-03-28 08:56:05
tags:
permalink: nice-to-meet-u
---
~~~

## 启用搜索功能

Deolin选择Hexo提供的local search来实现本地搜索功能。

~~~shell
$ npm install hexo-generator-search --save
$ npm install hexo-generator-searchdb --save
~~~

启用了之后重新生成一下hexo即可

~~~shell
$ hexo g
~~~

