---
title: 学习Python（9） JSON与CSV

date: 2019-04-14 08:45

updated: 2019-04-14 08:45

tags:
- Python

categories: Python

permalink: learning-python-json-csv
---

## 简介

这是系列的第9篇，记录一下如果将Python中的数据转化为JSON和CSV文件。

与Java需要引入一系列依赖不同，Python之需要引入一个名为`pandas`的类库，就可以实现各种常见数据格式的转化。



## 正文

示例代码如下

~~~python
#!/usr/bin/python3
import pandas as pd

demo_data = {'a': [1, 2, 3], 'b': [4, 5, '汉字']}

# help(pd)
df = pandas.DataFrame(demo_data)

# help(df)
print(df.to_json(force_ascii=False))
print(df.to_csv())
~~~



很简单，但是要特别注意汉字的编码问题。

