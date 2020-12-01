---

title: Elasticsearch常用查询备忘

date: 2019-03-06 11:36

updated: 2019-03-07 09:20

tags:
- Elasticsearch

categories: Java

permalink: elasticsearch-query
---

过时原因：总结、备忘类文章，非高深类技术，保存到Notion即可



## 简介

整理、备忘。

Elasticsearch版本为`6.6.1`



## 演示数据

演示数据来自

https://github.com/elastic/elasticsearch/blob/master/docs/src/test/resources/accounts.json?raw=true

一共1k条整，导入方式为

~~~shell
$ curl -H "Content-Type: application/json" -XPOST "localhost:9200/bank/_doc/_bulk?pretty&refresh" --data-binary "@accounts.json"
$ curl "localhost:9200/_cat/indices?v"
~~~



## “字符串”类型说明

elasticsearch中有2种字符串，一类叫text，一类叫keyword。
keyword类似与一般关系型数据库中的”字符串”，可以精准匹配(match)，左右模糊查询(wildcard)，排序(sort)等等，而且区别大小写。
text是全文类型，es会将全文类型的字段分成一个个词条，然后统一化，以提高可搜索型，查询时，会按照字条来匹配



## 查询语句

### 查询所有

~~~json
"query": {"match_all": {}}
~~~



### 根据字段排序

~~~json
"sort": [
    {
        "age": {
            "order": "desc"
        }
    },
    {
        "account_number": {
            "order": "asc"
        }
    }
]
~~~

可以理解为MySQL中的`ORDER BY age DESC, order`



### 限定结果条数

~~~json
"size": 1
~~~

可以理解为MySQL中的`LIMIT 1`
size缺省为10条



### 限定结果的起点和条数

~~~json
"from": 2,
"size": 15
~~~

可以理解为MySQL中的`LIMIT 2, 15`



### 限定结果的字段

~~~json
"_source": [
    "account_number",
    "balance"
]
~~~

可以理解为 SELECT account_number, balance
_source缺省为全字段



### 匹配查询（type不为text的字段，会进行精准匹配）

~~~json
"query": { "match": { "account_number": 20 } }
~~~



### 匹配查询（type为text的字段，会进行全文匹配）

~~~json
"query": { "match": { "address": "mill lane" } }
~~~

这个语句会查询出address中出现`mill`或`lane`的数据，不区分大小写



### 短语匹配查询（type为text的字段，会进行分析匹配）

~~~json
"query": { "match_phrase": { "address": "mill lane" } }
~~~

这个语句会查询出address中出现`mill lane`的数据



### 且，非且

~~~json
"query": {
    "bool": {
        "must": [
            {
                "match": {
                    "age": "20"
                }
            },
            {
                "match": {
                    "gender": "M"
                }
            }
        ]
        , "must_not": [
            {
                "match": {
                    "state": "AK"
                }
            }
        ]
    }
}
~~~

age为20，且gender中包含了M，且state中不包含AK（gender和state是text类型）



### 或

~~~json
"query": {
    "bool": {
        "should": [
            {
                "match": {
                    "age": "20"
                }
            },
            {
                "match": {
                    "gender": "M"
                }
            }
        ]            
    }
~~~



### 范围匹配

~~~json
"query": {
    "range": {
        "balance": {
            "gte": 20000,
            "lte": 30000
        }
    }
}
~~~

可以理解为MySQL中的`WHERE balance >= 20000 AND balance <= 30000`