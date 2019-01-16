---
title: 基于Jackson的JSON工具类

date: 2018-02-07 21:22

updated: 2018-02-07 21:22

tags:
- 工具类

categories: Java

permalink: json-util-based-jackson
---

## 源码

~~~java
/**
 * JSON工具类
 * <pre>
 * 支持JSON与对象间的互相转换，支持取得JSON结构中的具体的值。
 * p.s.: 本工具类不建议使用Java Collection中的类作为toObject()的返回值类型，
 * 如果仅仅需要临时取某个属性，推荐使用getValue()。
 * </pre>
 *
 * @author Deolin
 */
@UtilityClass
@Log4j2
public class JsonUtil {

    private static ObjectMapper defaultObjectMapper = new ObjectMapper();

    static {
        commonConfig(defaultObjectMapper);
        timeConfig(defaultObjectMapper);
        defaultObjectMapper.setPropertyNamingStrategy(PropertyNamingStrategy.SNAKE_CASE);
    }

    public static void commonConfig(ObjectMapper om) {
        om.configure(DeserializationFeature.FAIL_ON_UNKNOWN_PROPERTIES, false);
        om.setTimeZone(TimeZone.getTimeZone("GMT+8:00"));
    }

    public static void timeConfig(ObjectMapper om) {
        DateTimeFormatter date = DateTimeFormatter.ofPattern("yyyy-MM-dd");
        DateTimeFormatter time = DateTimeFormatter.ofPattern("HH:mm:ss");
        DateTimeFormatter dateTime = DateTimeFormatter.ofPattern("yyyy-MM-dd HH:mm:ss");
        JavaTimeModule javaTimeModule = new JavaTimeModule();
        // LocalDate Json化和反Json化
        javaTimeModule.addSerializer(LocalDate.class, new LocalDateSerializer(date));
        javaTimeModule.addDeserializer(LocalDate.class, new LocalDateDeserializer(date));
        // LocalTime  Json化和反Json化
        javaTimeModule.addSerializer(LocalTime.class, new LocalTimeSerializer(time));
        javaTimeModule.addDeserializer(LocalTime.class, new LocalTimeDeserializer(time));
        // LocalDateTime Json化和反Json化
        javaTimeModule.addSerializer(LocalDateTime.class, new LocalDateTimeSerializer(dateTime));
        javaTimeModule.addDeserializer(LocalDateTime.class, new LocalDateTimeDeserializer(dateTime));
        om.registerModule(javaTimeModule);
    }

    /**
     * 将对象转化为JSON字符串，采用默认SNAKE-CASE转化方式
     */
    public static String toJson(Object object) {
        return toJson(object, defaultObjectMapper);
    }

    /**
     * 将对象转化为JSON字符串，支持自定义ObjectMapper
     */
    public static String toJson(Object object, ObjectMapper om) {
        try {
            return om.writeValueAsString(object);
        } catch (JsonProcessingException e) {
            throw new RuntimeException("JsonUtil.toJson()", e);
        }
    }

    /**
     * 将JSON字符串转化为对象，采用默认SNAKE-CASE转化方式
     */
    public static <T> T toObject(String json, Class<T> clazz) {
        return toObject(json, clazz, defaultObjectMapper);
    }

    /**
     * 将JSON字符串转化为对象，支持自定义ObjectMapper
     */
    public static <T> T toObject(String json, Class<T> clazz, ObjectMapper om) {
        try {
            return om.readValue(json, clazz);
        } catch (IOException e) {
            log.info(e.getMessage());
            throw new RuntimeException("JsonUtil.toObject()", e);
        }
    }

    /**
     * 读取JSON中的某一个值
     * <pre>
     * 这个方法返回的是String格式。
     * 建议使用这个方法来方便地取得目标值，而不是转化为临时的Map对象后通过map.get("key")取得。
     * 例如，想要从下面的JSON从取到第二个user的id，可以调用
     * JsonUtil.getValue(Integer.class, json, "users", "1", "user_id");
     *
     * {
     *      "organization_id": "112",
     *      "users": [
     *          {
     *              "user_id": "1",
     *              "name": "Light Deolin"
     *          },
     *          {
     *              "user_id": "2",
     *              "name": "Dark Deolin"
     *          }
     *      ]
     *  }
     * </pre>
     *
     * @param json 目标JSON字符串
     * @param nodeKeys 抵达目标节点所有的节点key或数组下标
     * @return 目标值
     */
    public static String getValue(String json, String... nodeKeys) {
        try {
            JsonNode node = defaultObjectMapper.readTree(json);
            for (String nodeKey : nodeKeys) {
                if (node == null) {
                    return null;
                }
                if (NumberUtils.isCreatable(nodeKey)) {
                    Integer index = Integer.valueOf(nodeKey);
                    node = node.get(index);
                } else {
                    node = node.get(nodeKey);
                }
            }
            return node.asText();
        } catch (IOException e) {
            throw new RuntimeException("JsonUtil.getValue()", e);
        }
    }

}
~~~

## 演示

~~~java
	@Test
    public void testJsonUtil() {
        ObjectMapper om = new ObjectMapper();
        JsonUtil.commonConfig(om);
        JsonUtil.timeConfig(om);
        User user = User.builder().name("aaa_bbb").updatedAt(LocalDateTime.MAX).build();
        log.info(JsonUtil.toJson(user));
        log.info(JsonUtil.toJson(user, om));

        String json = "{\"id\":null,\"updatedAt\":\"+999999999-12-31 23:59:59\",\"name\":\"aaa_bbb\",\"salt\":null,\"sex\":null,\"age\":null,\"flag\":null,\"ymd\":null,\"hms\":null,\"ymdhms\":null,\"money\":null,\"serialNumber\":null,\"percent\":null,\"richText\":null}";
        String snakeJson = "{\"id\":null,\"updated_at\":\"+999999999-12-31 23:59:59\",\"name\":\"aaa_bbb\",\"salt\":null,\"sex\":null,\"age\":null,\"flag\":null,\"ymd\":null,\"hms\":null,\"ymdhms\":null,\"money\":null,\"serial_number\":null,\"percent\":null,\"rich_text\":null}";
        log.info(JsonUtil.toObject(json, User.class));
        log.info(JsonUtil.toObject(json, User.class, om));
        log.info(JsonUtil.toObject(snakeJson, User.class));
        log.info(JsonUtil.toObject(snakeJson, User.class, om));
    }
~~~

![](/images/20180409171253.png)

## 说明

工具类提供了默认**蛇形**风格的转化方式，也通过重载方法提供了自定义风格的转化方式。其中私有方法和静态代码块用于构建