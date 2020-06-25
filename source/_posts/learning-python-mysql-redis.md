---
title: 学习Python（8） MySQL与Redis

date: 2019-04-12 17:34

updated: 2020-06-25 13:14

tags:
- Python

categories: Python

permalink: learning-python-mysql-redis
---

## 简介

这是系列的第8篇，从这篇POST开始，会逐渐记录一些Python的三方库



## 正文

### MySQL

示例代码如下

~~~python
# coding=utf-8

import pymysql


# 建立MySQL连接
connection = pymysql.connect(host='127.0.0.1',
                             port=3306,
                             database='beginning_mind',
                             user='root',
                             password='root_r0oT',
                             charset='utf8')

cur = connection.cursor(pymysql.cursors.DictCursor)

try:
    cur.execute('insert into biz_demo (id,name) values (%s, %s)', (2, 'a'))
    connection.commit()
except:
    db.rollback()

cur.execute('select * from biz_demo')
for one in cur.fetchAll():
    print(one)
    
db.close()
~~~

`pymysql`库类似与Java中的`apache-dbutils`，简单业务场景下使用起来没问题，业务复杂时应该还是要上ORM框架



### Redis

示例代码如下

~~~python
# coding=utf-8

import redis

connection = redis.Redis(host='127.0.0.1',
                         password='root_r0oT',
                         decode_responses=True) # decode_responses=True用来确保取值时中文不乱码

connection.hset('a', 'b', '汉字')
print(connection.hget('a', 'b'))
~~~

`redis`是Redis官方提供的Python类库