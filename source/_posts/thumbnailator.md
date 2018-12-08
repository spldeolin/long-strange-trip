---
title: 基于Google Thumbnailator类库的图片压缩

date: 2018-11-29 13:42:00

tags:
- 工具类

categories: Java

permalink: thumbnailator
---



## 简介

`Thumbnailator`是Google提供的，用于生成高品质压缩图的类库，具体介绍可以参照<http://code.google.com/p/thumbnailator>。这篇POST整理了`Thumbnailator`的常用API和生成出来的效果。



## 示例

原图 kaneki.png (1712*962)

![](images/thumbnailator-kaneki.png)



~~~java
// 缩小（如果原图长比宽大，则长缩小到150，宽根据长的缩小比例缩小；反之同理）
Thumbnails.of("kaneki.png").size(150, 150).toFile("smaller-size.png");
~~~

![](images/thumbnailator-smaller-size.png)



~~~java
// 放大（如果原图长比宽大，则长放大到2048，宽根据长的放大比例放大；反之同理）
Thumbnails.of("kaneki.png").size(2048, 1536).toFile("larger-size.png");
~~~

![](images/thumbnailator-larger-size.png)



~~~java
// 缩小（最高质量）
Thumbnails.of("kaneki.png").size(150, 150).outputQuality(1F)
                .toFile("smaller-size-with-high-quality.png");
~~~

![](images/thumbnailator-smaller-size-with-high-quality.png)



~~~java
// 拉伸到指定宽高
// 这个方法适用于 例如需要用户上传500*500的图片，但用户切割误差上传了498*500的图片，为了不让图片两边显示出白色背景细线，所以把图片拉伸到500*500
Thumbnails.of("kaneki.png").size(150, 300).keepAspectRatio(false).toFile("fixed-size.png");
~~~

![](images/thumbnailator-fixed-size.png)



~~~java
// 等比缩放
Thumbnails.of("kaneki.png").scale(0.5D).toFile("smaller-scaled.png");
~~~

![](images/thumbnailator-smaller-scaled.png)



~~~java
// 格式转换（缩小硬盘占用）
Thumbnails.of("lemon.bmp").scale(1D).outputFormat("jpg").toFile(lemon.jpg");
~~~

![](images/thumbnailator-lemon.jpg)