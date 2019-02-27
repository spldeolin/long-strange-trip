---

title: Spring Boot 2集成Elasticsearch

date: 2019-02-27 17:16

updated: 2019-02-27 17:16

tags:
- Spring Boot
- Elasticsearch

categories: Java

permalink: springboot-elasticsearch

---

## 简介

Deolin最近打算学习Elasticsearch，这里将记录一下Spring Boot集成Elasticsearch的流程。



## 版本介绍

Spring Boot版本为`2.1.3.RELEASE`

Elasticsearch版本为`6.6.1`



## 安装并运行ES

~~~shell
$ brew install elasticsearch
~~~

~~~shell
$ brew services start elasticsearch

 Successfully started `elasticsearch`...
~~~



## POM

~~~xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-elasticsearch</artifactId>
</dependency>
~~~



## application.yml

~~~yaml
spring:
  data.elasticsearch:
    # Elasticsearch cluster name.
    clusterName: elasticsearch
    # Comma-separated list of cluster node addresses.
    clusterNodes: 127.0.0.1:9300
~~~



这些配置会映射到`org.springframework.boot.autoconfigure.data.elasticsearch.ElasticsearchProperties`的对象上



至此，集成完毕。下面演示一下CURD



## CURD

1. 实体

   ~~~java
   @Document(indexName = "oneDatabase", type = "book")
   @Data
   public class Book {
       
       @org.springframework.data.annotation.Id
       private String id;
       
       private String name;
   
   }
   ~~~

3. 保存测试

   ~~~java
   @RunWith(SpringRunner.class)
   @SpringBootTest
   public class BookTest {
       
       @Autowired
       private ElasticsearchTemplate template;
   
       @Test
       public void create() {
           Book book = new Book();
           book.setId("112");
           book.setName("啊啊啊啊啊");
           
           IndexQuery indexQuery = new IndexQuery();
           indexQuery.setObject(book);
           
           template.index(indexQuery);
           
           log.info("结束");
       }
       
   }
   ~~~

   稍微解释一下

   1. 这里选用`ElasticsearchTemplate`，而不是通过定义一个继承了`ElasticsearchRepository`的DAO来操作ES，原因是Deolin觉得前者的API更底层、更灵活。
   2. `IndexQuery`内部有很多字段，但由于给`Book`指定了`@Document`和`@Id`注解，所以`IndexQuery.indexName`、`IndexQuery.type`、`IndexQuery.id`不再需要显式地set了。

4. 查询测试

   ~~~i
       @Test
       public void create() {
           SearchQuery query = new NativeSearchQueryBuilder().build();
           
           elasticsearchTemplate.queryForList(query, Book.class).forEach(log::info);
           
           log.info("结束");
       }
   ~~~

   这里只演示最简单的查询



## 源码

[spldeolin](https://github.com/spldeolin) / [springboot-elasticsearch-with-template](https://github.com/spldeolin/springboot-elasticsearch-with-template) 