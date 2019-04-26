---
title: 学习Python（11） 发送HTTP请求

date: 2019-04-26 09:09

updated: 2019-04-26 09:09

tags:
- Python

categories: Python

permalink: learning-python-request
---

## 简介

这是系列的第11篇，介绍一个如果利用第三方库`requests`来发送HTTP请求。

演示发送请求需要一个Web服务，这里Deolin构建了一个Java的项目用于充当临时的Web服务。



## 正文

### 发送一个带query的GET请求

~~~python
# 示例：/test/getWithQuery?q=&type=

# help(requests.get)
resp = requests.get(url='http://127.0.0.1:2333/test/getWithQuery',
                    params={'q': '你好', 'type': '233'})
# help(resp)
if resp.status_code is 200:
    print(resp.text)
else:
    print('{}请求失败'.format(resp.status_code))
~~~



## 发送一个带请求body的POST请求

~~~python
# 示例：/test/postWithBodyAndQuery?q=
# {"name":"嗯","age":19}

resp = requests.post(url='http://127.0.0.1:2333/test/postWithBodyAndQuery',
                     params={'q': '啊'},
                     json={'name': '嗯', 'age': 19},
                     headers={'Content-Type': 'application/json'})
if resp.status_code is 200:
    print(resp.text)
else:
    print('{}请求失败'.format(resp.status_code))
~~~



## 发送一个带表单的POST请求

~~~python
# 示例：/test/postWithFormAndQuery?q=
# name 嗯
# age  19

resp = requests.post(url='http://127.0.0.1:2333/test/postWithFormAndQuery',
                     params={'q': '唉'},
                     data={'name': '哇', 'age': 0})
if resp.status_code is 200:
    print(resp.text)
else:
    print('{}请求失败'.format(resp.status_code))
~~~



### 其它

HTTP请求形式基本上是以上述三类情况为主，还有一种常见的可能是

~~~
/user/{userId}/address/{addressId}

如：/user/12/address/12312
~~~

这种情况一般就是手动拼接URL了



## 服务端代码

这里顺便把Web服务的关键代码也放出来

~~~java
@RestController
@RequestMapping("/test")
@Log4j2
public class TestController {
  
    @GetMapping("/getWithQuery")
    RequestResult getWithQuery(@RequestParam String q, @RequestParam Integer type) {
        log.info("收到请求 q={}, type={}", q, type);
        return RequestResult.success(q + type);
    }

    @PostMapping("/postWithBodyAndQuery")
    RequestResult postByBody(@RequestParam String q, @RequestBody NiceDTO dto) {
        log.info("收到请求 q={}, dto={}", q, dto);
        return RequestResult.success(dto);
    }

    @PostMapping("/postWithFormAndQuery")
    RequestResult postWithFormAndQuery(@RequestParam String q, NiceDTO dto) {
        log.info("收到请求 q={}, dto={}", q, dto);
        return RequestResult.success(dto);
    }

}
~~~

