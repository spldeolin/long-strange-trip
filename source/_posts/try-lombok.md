---
title: 使用lombok

date: 2017-12-20 14:30:00

updated: 2017-12-20 14:30:00

tags:
- Lombok
- IntelliJ IDEA

categories: Java

permalink: try-lombok
---

## 为IDE集成Lombok插件

### Eclipse

从官网下在`lombok.jar`，拷贝到Eclipse的根目录，在`ecplise.ini`的最后追加

~~~ini
-Xbootclasspath/a:lombok.jar
-javaagent:lombok.jar
~~~

### IntelliJ IDEA

`Setting` - `Plugins` > `Browse repositories...` 搜索并安装`Lombok`

## 为项目整合Lombok

在pom.xml中追加

~~~xml
<dependency>
    <groupId>org.projectlombok</groupId>
    <artifactId>lombok</artifactId>
    <version>1.16.20</version>
    <scope>provided</scope>
</dependency>
~~~

## 使用Lombok

常用的注解有`@Data`、`@Log4j2`等，具体可以参考官方文档。

## 文件模板

对于一个DTO来说，它可能需要较多的声明和注解，使自己用起来比较舒服。基于这点，可以在IntelliJ IDEA为DTO类设置一套专门的`文件模板`。

*config/fileTemplates/DTO.java*

~~~java
#if (${PACKAGE_NAME} && ${PACKAGE_NAME} != "")package ${PACKAGE_NAME};#end
#parse("File Header.java")

import java.io.Serializable;
import lombok.AllArgsConstructor;
import lombok.Builder;
import lombok.Data;
import lombok.NoArgsConstructor;
import lombok.experimental.Accessors;

@Data
@NoArgsConstructor
@AllArgsConstructor
@Builder
@Accessors(chain = true)
public class ${NAME} implements Serializable {

    private static final long serialVersionUID = 1L;

}
~~~