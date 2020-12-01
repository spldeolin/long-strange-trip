---

title: 在Python中使用数据传输类（DTO）

date: 2020-05-18 09:36

updated: 2020-05-18 09:36

tags:
- Python

categories: Python

permalink: python-dto

---

## 简介

最近写了一些复杂的脚本，编写过程中逐渐发现了Python中的`dict`虽然很灵活简便，但一旦脚本中的数据结构复杂了，`key`就需要统一管理起来了，否则需要不断地看代码的上下文来确定这个`dict`对象有哪些`key`

这个时候使用`DTO`比较好，因为声明`DTO`时需要直接声明类中有哪些`field`，后续使用dto时可以直接通过IDE的只能提示来使用属性



## namedtuple

Python提供了一种特殊的元组——`namedtuple`，用于快速声明一个DTO，`namedtuple`位于`collections`模块。

使用示例如下

```python
# coding=utf-8
from collections import namedtuple

UserDto = namedtuple("UserDto", "id name age address")
AddressDto = namedtuple("AddressDto", "provice city")
user = UserDto(1231231, '汉字', 17, AddressDto('浙江', city='杭州'))

print(user) # UserDto(id=1231231, name='汉字', age=17, address=AddressDto(provice='浙江', city='杭州'))
print(user.address.city) # 杭州
```



## namedtuple的优势

![](/images/python-dto-01.png)

![](/images/python-dto-02.png)

能够自动提示有哪些`field`，而不用像dict那样需要去想上文中使用了什么`key`



## JSON序列化

~~~python
serialized = json.dumps(user._asdict(), ensure_ascii=False)
print(serialized) # {"id": 1231231, "name": "汉字", "age": 17, "address": ["浙江", "杭州"]}
~~~

需要注意的是，`namedtuple`方法返回的是`a new subclass of tuple with named fields`，即它是一个元组，直接`json.dumps`会当作一个数组来数列化，如果需要序列化以后的现象与一个DTO一致，那么需要多做一步`_asdict()`操作