---
title: MySQL常用字段类型与对应的Java类型

date: 2018-01-05 15:09:00

updated: 2018-01-05 15:09:00

tags: 

categories: MySQL

permalink: mysql-frequent-fields
---

| MySQL字段类型          | 描述         | 适用场景                                                     | 对应的Java类型     |
| ---------------------- | ------------ | ------------------------------------------------------------ | ------------------ |
| varchar                | 不定长字符串 | 普通文本或是没有合适类型时，可以选择它作为字段类型           | String             |
| int和bigint            | 整数数值     | 一般以int作为数字的默认选择，数值很大时使用bigint            | Integer和Long      |
| char                   | 定长字符串   | 适用于盐、md5加密后的密码等情况                              | String             |
| float和double          | 小数数值     | 适用于各种小数，除非金额等情况，小数推荐使用double           | Float和Double      |
| decimal                | 精确浮点数   | 适用于金额                                                   | BigDecimal         |
| tinyint                | 逻辑型       | 适用于是/否的情况，ORM框架一般会将其映射为true/false         | Boolean            |
| date / time / datetime | “时间”       | 各自适用于年月日，时分秒，年月日时分秒三种情况               | 全是java.util.Date |
| text                   | 文本         | 适用于富文本、文章正文等                                     | String             |
| enum                   | 枚举         | 适用于性别、订单状态等（不过感觉用int代替enum更为常见）      | String             |
| blob                   | 二进制       | 适用于小图片、小音频等。（不过更常用的做法是上传到服务器，在DB中存储url） | String             |

