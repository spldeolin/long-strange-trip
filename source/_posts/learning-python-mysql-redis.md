---
title: 学习Python（8） MySQL与Redis

date: 2019-04-12 17:34:15

updated: 2019-04-12 17:34:15

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
#!/usr/bin/python3
import pymysql


# 建立数据库连接
def connect_mysql():
    """
    连接MySQL，返回数据库连接对象
    """
    # help(pymysql.connect)
    help(pymysql.connections.Connection.__init__)
    connection = pymysql.connect(host='127.0.0.1',
                                 user='root',
                                 password='root_r0oT',
                                 database='beginning_mind',
                                 charset='utf8')
    return connection


def write(sql, *param_tuple):
    """
    入库操作
    """
    db = connect_mysql()
    cur = db.cursor()
    try:
        help(cur.execute)
        cur.execute(sql, param_tuple)
        db.commit()
    except:
        db.rollback()
    finally:
        db.close()


def read(sql, *param_tuple):
    """
    查询数据库
    """
    db = connect_mysql()
    cur = db.cursor()
    try:
        help(cur.execute)
        cur.execute(sql, param_tuple)
        result = cur.fetchmany(2)

        # print(type(result))   # 是个tuple，取值的时候会有点难受
        for one in result:
            print(one[1])
    finally:
        db.close()


if __name__ == '__main__':
    write('insert into biz_demo (id,name) values (%s, %s)', 2, 'a')
    read("select * from biz_demo")
~~~



`pymysql`库类似与Java中的`apache-dbutils`，简单业务场景下使用起来没问题，业务复杂时应该还是要上ORM框架



### Redis

示例代码如下

~~~python
#!/usr/bin/python3
import redis


# 建立数据库连接
def connect_redis():
    """
    连接Redis，返回连接对象
    """
    # help(redis.Redis)
    connection = redis.Redis(host='127.0.0.1',
                             password='root_r0oT',
                             decode_responses=True) # decode_responses=True用来确保取值时中文不乱码
    return connection


if __name__ == '__main__':
    conn = connect_redis()
    # help(conn)
    conn.hset('a', 'b', '汉字')
    print(conn.hget('a', 'b'))
~~~



`redis`是Redis官方提供的Python类库