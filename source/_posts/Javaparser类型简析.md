---

title: Javaparser类型简析

date: 2019-11-01 21:46

updated: 2019-11-01 21:46

tags:
- Java

categories: Java

permalink: javaparser-type

---

##  BodyDeclaration “声明”

1. `AnnotationDeclaration`

   ~~~java
   public @interface JedisOperation {
   	...
   }
   ~~~

2. `AnnotationMemberDeclaration`

   ~~~java
   boolean paramInReturned() default false; // @interface里的属性
   ~~~

3. `ClassOrInterfaceDeclaration`

   ~~~java
   public class Student {
       ...
   }
   ~~~

   ~~~java
   public interface payable {
       ...
   }
   ~~~

4. `ConstructorDeclaration`

   ~~~java
       public BaseBizException() { // 构造方法
           ...
       }
   ~~~

5. `EnumConstantDeclaration`

   ~~~java
       SPRING(1, "spring"), // 枚举元素
   ~~~

6. `EnumDeclaration`

   ~~~java
   public enum FourSession {
       ...
   }
   ~~~

7. `FieldDeclaration`

   ~~~java
       private List<StudentDTO> students;
   ~~~

8. `InitializerDeclaration`

   ~~~java
       { // 构造代码块
            ...
       }
   ~~~

   ~~~java
       static { // 静态代码块
           ....
       }
   ~~~

9. `MethodDeclaration`

   ~~~java
       private void doSomething(Long userId) {
            ...
       }
   ~~~

   ~~~java
       public abstract onSuccess(Long userId);
   ~~~

10. `PackageDeclaration`

    ~~~java
    package com.spldeolin.demo.entity;
    ~~~

11. `VariableDeclarator`

    ~~~java
        userService // 处于 @Autowired private UserService userService;中
    ~~~

    ~~~
        i = 0 // 处于for (int i = 0; i < size; i ++)中
    ~~~

12. `ImportDeclaration`

    ~~~java
    import com.spldeolin.demo.entity.UserEntity;
    ~~~

13. `ModuleDeclaration`

    Java9的“模块”

    `ModuleDeclaration`不是`BodyDeclaration`的派生类



## Expression “表达式”

1. `ArrayAccessExpr`

   数组对象+中括号下标

2. `ArrayCreationExpr`

   通过指定个数或者指定所有元素的方式new一个数组

3. `ArrayInitializerExpr`

   指定数组元素时的整个大括号

4. `AssignExpr`

   整个赋值语句

5. `BinaryExpr`

   有运算符参与的运算语句

6. `CastExpr`

   类型强转

7. `ClassExpr`

   例如：`UserEntity.class`

8. `ConditionalExpr`

   三目表达式

9. `EnclosedExpr`

   整个小括号

10. `FieldAccessExpr`

    访问field（包含`this`关键字）

11. `InstanceOfExpr`

    例如：`userDTO instaceof BaseDTO`

12. `MarkerAnnotationExpr`

    例如：`@Override`

13. `MethodCallExpr`

    方法调用（链式调用一整句话算一个MethodCallExpr对象）

14. `NameExpr`

    - 最基础的`Expression`，代表变量名、属性名、package名等等等...

    - 很多`Expression`内部都是由若干个`NameExpr`组成，例如，一个非链式调用的`MethodCallExpr`对象，内部有一个`NameExpr scope`属性，代表点号左侧那个被调用方名称；再例如，``FieldAccessExpr``内部有个`NameExpr name`属性，代表field的名称

    - `NameExpr`内部只有个`SimpleName`类型的属性，后者是个最基础的`Node`

15. `NormalAnnotationExpr`

    例如：`@NotNull(message = "用户名不能为空")`

16. `ObjectCreationExpr`

    任何new东西的语句

17. `SingleMemberAnnotationExpr`

    例如：`@JsonProperty("started_at")`

18. `SuperExpr`

    就一个`super`关键字代表的表达式

19. `ThisExpr`

    就一个`this`关键字代表的表达式

20. `UnaryExpr`

    取反表达式，感叹号和负号都算

21. `VariableDeclarationExpr`

    - 一个声明局部变量的表达式
    - 内部由n个`VariableDeclarator`对象构成，区别如下
      - `int a=0, b, c=-111`是`VariableDeclarationExpr`对象
      - `a=0`是`VariableDeclarator`对象，`b`和`c=-111`也是

22. `LambdaExpr`

    Java8的Lambda表达式，标志性运算符是`->`

23. `MethodReferenceExpr`

    Java8的方法指针，标志性运算符是`::`

24. `TypeExpr`

    与`NameExpr`地位相似，最基础的`Expression`，代表变量类型、属性类型等等等...

    很多`Expression`内部都有`TypeExpr`

25. `SwitchExpr`

    Java12的Switch表达式

###  LiteralExpr “字面量”

1. BooleanLiteralExpr
2. CharLiteralExpr
3. DoubleLiteralExpr
4. IntegerLiteralExpr
5. LongLiteralExpr
6. NullLiteralExpr
7. StringLiteralExpr
8. TextBlockLiteralExpr



##Statement “语句/代码块”

难以找到一个中文词语来描述`Statement`，但可以看看它们的一些共性

- 有些`Statement`代表一个以`;`结束的语句，这类`Statement`对象内部有个`Expression`属性

- 有些`Statement`代表一个大括号代码块，这类`Statement`对象内部有个`Statement`列表

1. `AssertStmt`

   `assert`开头的断言

2. `BlockStmt`

   一整个大括号，内部有个`Statement`列表

3. `BreakStmt`

   就是`break;`

4. `ContinueStmt`

   就是`continueStmt;`或者`continue myLabel;`

5. `DoStmt`

   do-while代码块，内部有`condition`、`body`两个属性

6. `EmptyStmt`

   就是`;`

7. `ExplicitConstructorInvocationStmt`

   调用父构造方法的语句

8. `ExpressionStmt`

   `;`结尾的表达式语句

9. `ForEachStmt`

   for-each代码块，内部有`variable`、`iterable`、`body`三个属性

10. `ForStmt`

    for-index代码块，内部有`initialization`、`compare`、`update`、`body`四个属性

11. `IfStmt`

    if-else代码块，内部有`condition`、`thenStmt`、`elseStmt`三个属性

    其中elseStmt还可以是个`IfStmt`，代表elseif

12. `LabeledStmt`

    label代码块

13. `ReturnStmt`

    return语句

14. `SwitchStmt`

    Switch代码块

15. `SynchronizedStmt`

    同步代码块

16. `ThrowStmt`

    throw语句

17. `TryStmt`

    try-catch语句，内部有`tryBlock`、`catchClauses`、`finallyBlock`，以及代表Java7的auto-close的`resources`

18. `LocalClassDeclarationStmt`

    Java11的方法内类声明

19. `WhileStmt`

    while代码块，内部有`condition`和`body`两个代码块

20. `UnparsableStmt`

    无法解析的语句或者代码块，一般编译能通过的代码，解析不出这种Statement

21. `YieldStmt`

    Java12的yield语句



## Type “类型”

`Type`的派生类也是个大家族，但它们的命名往往比较直观

类内部的属性也很简单，一般只有`name`需要关注，最多是个泛型类型，还有个`Type typeArguments`属性

所以这里不列举了



## Comment “注释”

`Comment`只有3个派生类

- `LineComment` 行注释
- `BlockComment` 块状注释
- `JavadocComment`  `/**`开头的Javadoc风格的注释

没有特别需要说明的



## Javadoc “Java Doc”

`Javadoc`是一个与`JavadocComment`的不同的类，前者不是`Comment`的派生类，甚至不是`Node`的派生类。

两者的区别，可以以下面两个例子说明——

~~~java
/**
 * JavadocComment对象内部只有一个String content属性，所以它只能表示简单的JavaDoc
 */
~~~

~~~java
/**
 * 这是<code>Javadoc</code>对象所有代表的注释
 * <pre>
 *  1. Javadoc内部属性的类型和层次比较丰富，
 *  2. 所以可以表示像这个这样的
 *  3. 复杂JavaDoc
 * </pre>
 *
 * @author Deolin
 * @see Javadoc
 */
~~~



![](/images/javaparser-type-01.png)





