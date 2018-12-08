title: 最优雅的方式解决@Valid List<E> 无法校验的问题
date: 2018-03-10 17:57:52

tags:

- Spring

categories: Java

permalink: valid-list
---

## 场景

如果外对API的参数涉及到数组时，参数校验可能会比较麻烦。

### 情况1

~~~java
@PostMapping
public String doIt(@RequestBody @Valid List<User> users) {
    return "SUCCESS";
}
~~~

如果API这样设计的话，经测试，@Valid无法发挥作用。

### 情况2

~~~java
@PostMapping
public String doItAgain(@RequestBody @Valid UserList userList) {
    return "SUCCESS";
}
~~~

~~~java
@Data
public class UserList() {
	
	@Valid
    private List<User> users;

}
~~~

这种情况是**情况1**的进阶，这样设计至少@Valid能够发挥作用了。但是太麻烦，需要专门写一个`UserList`类，简直不优雅。同时外接的JSON需要需要多套一层，如下

~~~json
{"users": [{"name": "Deolin", "age": 18}, {"name": "Rin", "age": 17}]}
~~~

多一层`"users"`对调用方来说也是麻烦。

## 分析

**情况1**那样子设计行不通的原因是，`java.util.List`（`ArrayList`）内部通过持有一个数组来保存对象们，而作为Java官方的类，内部肯定不会在数组上声明`@Valid`，所以内部的对象们没有得到应有的**递归校验**。

所以，考虑使用一种新的`java.util.List`实现，来变相的达到列表校验的效果。

## 解决方案

~~~java
/**
 * 可被校验的List
 *
 * @param <E> 元素类型
 * @author Deolin
 */
 public class ValidableList<E> implements List<E> {
 
    @Valid
    private List<E> list;
     
    public ValidableList() {
        this.list = new ArrayList<>();
    }
    
    public ValidableList(List<E> list) {
        this.list = list;
    }
    
    public List<E> getList() {
        return list;
    }

    public void setList(List<E> list) {
        this.list = list;
    }
    
    @Override
    public int size() {
        return list.size();
    }

    @Override
    public boolean isEmpty() {
        return list.isEmpty();
    }

    // 其他的Override方法省略。它们与size()和isEmpty()类似，直接调用private List list处理。 

 }
~~~

可以看到`ValidableList`具有*二象性*，`ValidableList` LIKE-A `java.util.List`，同时也是个“holder”，对外的方法全部会移交给持有的list处理。

**也就是说，ValidableList与java.util.List的对外功能完全一致。**

对于Spring参数绑定来说，JSON转换后也能够绑定到`ValidableList<E>对象上`（就像能绑定到`java.util.ArrayList<E>对象`上一样。）

实现了上述的效果后，直接在list上声明一个`@Valid`就解决所有问题了。

最终的解决方案是这样的——

API

~~~java
@PostMapping
public String noMore(@RequestBody @Valid ValidableList<User> users) {
    return "SUCCESS";
}
~~~

JSON

~~~json
[{"name": "Deolin", "age": 18}, {"name": "Rin", "age": 17}]
~~~





