---
title: 使用OkHttp3

date: 2018-04-13 15:26:53

tags: 工具类

categories: Java

permalink: okhttp3
---

## 简介

OkHttp3是一个Http客户端的类库，API的使用比Apache的HttpClient简单不少。

Apache的HttpClient由于一些原因，在Maven中央仓库有两个版本——[commons-httpclient](https://mvnrepository.com/artifact/commons-httpclient)和[Apache HttpClient](https://mvnrepository.com/artifact/org.apache.httpcomponents/httpclient)。虽然前者已经不再更新，但是很多老项目用的都是它，随着功能迭代，项目慢慢地就既依赖`commons-httpclient`又依赖`Apache HttpClient`，两个类库两套API，麻烦很多。

同时，OkHttp作为Android的默认网络框架，性能也是值得信赖的。

## 引入依赖

~~~xml
<dependency>
    <groupId>com.squareup.okhttp3</groupId>
    <artifactId>okhttp</artifactId>
    <version>3.10.0</version>
</dependency>
~~~



## 工具类

```java
/**
 * HTTP请求工具类
 * <pre>
 * 支持服务端发送HTTP请求，包括Get请求，Body为form的Post请求，Body为json的Post请求，
 * 请求的回应将被转化为String类型作为返回值。
 *
 * 这个类推荐与JsonUtil结合使用。
 * </pre>
 */
@UtilityClass
@Log4j2
public class HttpUtil {

    private static OkHttpClient client;

    static {
        client = new OkHttpClient();
    }

    /**
     * 发送一个GET请求
     * <pre>
     * e.g.: HttpUtil.get("http://spldeolin.com/reply/123?page=5");
     * </pre>
     */
    public static String get(String url) {
        try {
            Request request = new Request.Builder().url(url).build();
            Response response = client.newCall(request).execute();
            return response.body().string();
        } catch (IOException e) {
            log.info(e.getMessage());
            throw new RuntimeException("HttpUtil.get()发生异常。");
        }
    }

    /**
     * 发送一个POST请求，请求Body的格式是JSON
     * <pre>
     * e.g.: HttpUtil.post("http://spldeolin.com/post/start", JsonUtil.toJson(userDTO));
     * </pre>
     */
    public static String postJson(String url, String json) {
        try {
            okhttp3.RequestBody body = okhttp3.RequestBody.create(MediaType.parse("application/json"), json);
            Request request = new Request.Builder().url(url).post(body).build();
            Response response = client.newCall(request).execute();
            return response.body().string();
        } catch (IOException e) {
            log.info(e.getMessage());
            throw new RuntimeException("HttpUtil.postJson()发生异常。");
        }
    }

    /**
     * 发送一个POST请求，请求Body的格式是JSON
     * <pre>
     * e.g.: HttpUtil.post("http://spldeolin.com/post/start", userDTO);
     * </pre>
     */
    public static String postJson(String url, Object object) {
        return postJson(url, JsonUtil.toJson(object));
    }

    /**
     * 发送一个POST请求，请求Body的格式是Form表单
     * <pre>
     * e.g.: HttpUtil.post("http://spldeolin.com/post/like", userDTO);
     * </pre>
     */
    public static String postForm(String url, Object object) {
        try {
            FormBody.Builder form = new FormBody.Builder();
            if (object instanceof Map) {
                Map<?, ?> map = (Map) object;
                for (Map.Entry<?, ?> entry : map.entrySet()) {
                    form.add(entry.getKey().toString(), entry.getValue().toString());
                }
            } else {
                for (Field field : object.getClass().getDeclaredFields()) {
                    field.setAccessible(true);
                    form.add(field.getName(), field.get(object).toString());
                }
            }
            okhttp3.RequestBody body = form.build();
            Request request = new Request.Builder().url(url).post(body).build();
            Response response = client.newCall(request).execute();
            return response.body().string();
        } catch (IllegalAccessException | IOException e) {
            log.info(e.getMessage());
            throw new RuntimeException("HttpUtil.postForm()发生异常。");
        }
    }

}
```



