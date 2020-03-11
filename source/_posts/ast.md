---

title: 抽象语法树

date: 2020-03-11 08:59

updated: 2020-03-11 08:59

tags:
- AST

categories: 未分类

permalink: ast

---

## 简介

这篇POST将以图文结合的方式直观的介绍一下抽象语法树



## 源码

~~~java
class X {
    int x;
}
~~~



## 语法结构（xml结构）

~~~xml
<root type='CompilationUnit'>
    <types>
        <type type='ClassOrInterfaceDeclaration' isInterface='false'>
            <name type='SimpleName' identifier='X'></name>
            <members>
                <member type='FieldDeclaration'>
                    <variables>
                        <variable type='VariableDeclarator'>
                            <name type='SimpleName' identifier='x'></name>
                            <type type='PrimitiveType' type='INT'></type>
                        </variable>
                    </variables>
                </member>
            </members>
        </type>
    </types>
</root>

~~~



## 树状图

![](/images/ast-01.png)