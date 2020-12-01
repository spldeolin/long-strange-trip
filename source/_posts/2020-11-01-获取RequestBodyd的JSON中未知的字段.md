---

title: 获取RequestBody的JSON中未知的字段

excerpt: 总会出现恶劣的联调环境

date: 2020-11-01 09:26

updated: 2020-11-01 09:26

tags:
- Jackson
- Spring MVC

categories: Java

permalink: warn-unknown-properties

---

## 简介

Spring MVC默认的使用Jackson作为将JSON序列化@RequestBody对象的实现。

而在实际的web开发时，往往会对Jackson进行这样的配置

~~~java
new ObjectMapper().configure(DeserializationFeature.FAIL_ON_UNKNOWN_PROPERTIES, false)
~~~



这样做的目的为了确保接口的API适配性，即API只需要自己所需要的字段，对于**<u>不认识的字段</u>**，也不快速失败，仅仅是无视

这样做的好处是一个标准的API可以被多个地方所复用，只要调用方至少能提供API所需的参数即可。



## 引发的问题

这样做会产生一个问题，如果字段名搞错了，就会出现——一个选填项，明明在页面上面填了，但是没有保存到数据库的情况，因为前端提供的字段名与后端所需的字段名不一致。

这是一个简单的BUG，但很难快速被第一时间发现



## 解决思路

现在，提供一个“快速警告”的方式。能够在**<u>每次请求时，获取到所有后端服务不认识的字段</u>**

1. 定义一个线程上下文，用于为请求保存请求报文中不认识的字段

   ~~~java
   public class RequestTrack {
   
       private static final ThreadLocal<RequestTrack> context = ThreadLocal.withInitial(RequestTrack::new);
       
       private RequestTrack() {
       }
   
       public static RequestTrack getCurrent() {
           return context.get();
       }
   
       public static void removeCurrent() {
           context.remove();
       }    
   
       private transient Map<String, String> unrecognizedRequestBodyProperties = Maps.newHashMap();
       
   }
   ~~~



2. 让所有ReqDto都继承2个方法

   ~~~java
   public abstract class ReqDtoAncestor {
   
       @JsonAnyGetter
       public Map<String, String> any() {
           return RequestTrack.getCurrent().getUnrecognizedRequestBodyProperties();
       }
   
       @JsonAnySetter
       public void set(String name, String value) {
           RequestTrack.getCurrent().getUnrecognizedRequestBodyProperties().put(name, value);
       }
   
   }
   ~~~

   

3. 在BaseRespDto中新增一个方法，方法必须是get开头，这个做法会使这个方法在序列化成JSON被视为一个字段

   ~~~java
   public class BaseRespDto<T> {
       private Boolean successOrNot;
       private T data;
       private String errorMessage;
   
       public Map<String, String> getUnrecognizedRequestBodyProperties() {
   		    RequestTrack currentRequestTrack = RequestTrack.getCurrent();
   		    if (currentRequestTrack != null) {
   			    return currentRequestTrack.getUnrecognizedRequestBodyProperties();
   	    	}
   		    return Maps.newHashMap();
   	  }
   
       // ignore getter and setters
   }
   ~~~



4. 至此，每次请求时，所有后端API不认识的字段，都会在返回值的`unrecognizedRequestBodyProperties`中显示出来，已作为对前端开发者的一种“快速警告”



整个方案的核心是`com.fasterxml.jackson.annotation.JsonAnySetter`和`com.fasterxml.jackson.annotation.JsonAnyGetter`注解。