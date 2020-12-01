---

title: 在@RequestBody中使用enum

date: 2019-09-13 17:40

updated: 2019-09-13 17:40

tags:
- Java

categories: Java

permalink: enum-in-requestbody
---

过时原因：教学类文章，没有独特性，不适合放在个人网站

## 简介

演示一下如何在handler的@RequestBody中使用枚举类型field



## Enum VS Integer

~~~java
@Data
public class Dto1 {
    private Long id;
    private Integer dayEnumCode;
}
~~~

~~~java
@Data
public class Dto2 {
    private Long id;
    private DayEnum dayEnum;
}
~~~

显而易见，枚举比Integer更好，简单、直观，不需要在业务代码里多做一步Integer到枚举的转换。

另外在项目使用Enum来代替Integer是一种非常好的实践方式，《Effective Java》也好，各种各样的代码规范中也好，都提到过这点



## 如何实现在@RequestBody中使用枚举

- 枚举类

  ~~~java
  @AllArgsConstructor
  @Getter
  @ToString
  public enum DayEnum {
  
      MONDAY(1, "周一"), TUESDAY(2, "周二"), WEDNESDAY(3, "周三"), THURSDAY(4, "周四"), FRIDAY(5, "周五"), SATURDAY(6,
              "周六"), SUNDAY(7, "周日");
  
      private Integer code;
  
      private String display;
  
      @JsonCreator
      public static DayEnum fromText(String text) {
          for (DayEnum dayEnum : DayEnum.values()) {
              if (dayEnum.getCode().toString().equals(text)) {
                  return dayEnum;
              }
          }
          throw new IllegalArgumentException("unknown code");
      }
  
  }
  ~~~

  > 注意：
  >
  > 1. DayEnum属性作为@ResponseBody类的field时，这个方法会被回调。（JSON -> Object）
  > 2. DayEnum属性作为@RequestBody类的field时，这个方法会被回调。（Object -> Object）
  >
  > 

- RequestBody类

  ~~~java
  @Data
  public class SimpleDTO implements Serializable {
  
      private Long id;
  
      private DayEnum dayEnum;
  
      private static final long serialVersionUID = 1L;
  
  }
  ~~~

- handler

  ~~~java
      @PostMapping("/bb")
      @ResponseBody
      Object bb(@RequestBody SimpleDTO simple) {
          simple.setId(-1L);
          return simple;
      }
  ~~~

- 效果

  ![](/images/enum-in-requestbody-01.png)

  ![](/images/enum-in-requestbody-02.png)

  ![](/images/enum-in-requestbody-03.png)

  效果拔群



## Json类库

这篇POST是基于Spring MVC的默认Json转换器——Jackson类库。

也就是说，SimpleDTO对象通过ObjectMapper转化或者解析Json时，toString()和@JsonCreator方法也能生效。