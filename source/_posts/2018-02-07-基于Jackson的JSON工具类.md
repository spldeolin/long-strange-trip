---
title: 基于Jackson的JSON工具类

excerpt: 拿来就能用

date: 2018-02-07 21:22

updated: 2022-11-22 22:42

tags:
- 工具类

categories: Java

permalink: json-util
---



一共有3个java文件，1个Exception，2个Util，抽取出ObjectMapperUtils是为了基于Jackson的CSV工具类也能用上

~~~java
/**
 * JSON工具类
 *
 * <pre>
 * 序列化与反序列化策略参考{@link ObjectMapperUtils#initDefault}
 * </pre>
 *
 * @author Deolin 2018-04-02
 */
@Slf4j
public class JsonUtils {

    private static final ObjectMapper om = ObjectMapperUtils.initDefault(new ObjectMapper());

    private JsonUtils() {
        throw new UnsupportedOperationException("Never instantiate me.");
    }

    /**
     * 将对象转化为JSON
     */
    public static String toJson(Object object) {
        return toJson(object, om);
    }

    /**
     * 将对象转化为JSON
     */
    public static String toJson(Object object, ObjectMapper om) {
        try {
            return om.writeValueAsString(object);
        } catch (JsonProcessingException e) {
            log.error("object={}", object, e);
            throw new JsonException(e);
        }
    }

    /**
     * 将对象转化为JSON，结果是美化的
     */
    public static String toJsonPrettily(Object object) {
        return toJsonPrettily(object, om);
    }

    /**
     * 将对象转化为JSON，结果是美化的
     */
    public static String toJsonPrettily(Object object, ObjectMapper om) {
        try {
            return om.writerWithDefaultPrettyPrinter().writeValueAsString(object);
        } catch (JsonProcessingException e) {
            log.error("object={}", object, e);
            throw new JsonException("转化JSON失败");
        }
    }

    /**
     * 将JSON转化为对象
     */
    public static <T> T toObject(String json, Class<T> clazz) throws JsonException {
        return toObject(json, clazz, om);
    }

    /**
     * 将JSON转化为对象
     */
    public static <T> T toObject(String json, Class<T> clazz, ObjectMapper om) throws JsonException {
        try {
            return om.readValue(json, clazz);
        } catch (IOException e) {
            log.error("json={}, clazz={}", json, clazz, e);
            throw new JsonException(e);
        }
    }

    /**
     * 将JSON转化为对象列表
     */
    public static <T> List<T> toListOfObject(String json, Class<T> clazz) throws JsonException {
        return toListOfObject(json, clazz, om);
    }

    /**
     * 将JSON转化为对象列表
     */
    public static <T> List<T> toListOfObject(String json, Class<T> clazz, ObjectMapper om) throws JsonException {
        try {
            @SuppressWarnings("unchecked") Class<T[]> arrayClass = (Class<T[]>) Class
                    .forName("[L" + clazz.getName() + ";");
            return Lists.newArrayList(om.readValue(json, arrayClass));
        } catch (IOException | ClassNotFoundException e) {
            log.error("json={}, clazz={}", json, clazz, e);
            throw new JsonException(e);
        }
    }

    /**
     * JSON -> 参数化的对象
     */
    public static <T> T toParameterizedObject(String json, TypeReference<T> typeReference) throws JsonException {
        return toParameterizedObject(json, typeReference, om);
    }

    /**
     * JSON -> 参数化的对象
     */
    public static <T> T toParameterizedObject(String json, TypeReference<T> typeReference, ObjectMapper om)
            throws JsonException {
        try {
            return om.readValue(json, typeReference);
        } catch (JsonProcessingException e) {
            log.error("json={}, typeReference={}", json, typeReference, e);
            throw new JsonException(e);
        }
    }

    /**
     * JSON -> JsonNode对象
     *
     * <strong>除非JSON对应数据结构在运行时是变化的，否则不建议使这个方法</strong>
     */
    public static JsonNode toTree(String json) throws JsonException {
        return toTree(json, om);
    }

    /**
     * JSON -> JsonNode对象
     *
     * <strong>除非JSON对应数据结构在运行时是变化的，否则不建议使这个方法</strong>
     */
    public static JsonNode toTree(String json, ObjectMapper om) throws JsonException {
        try {
            return om.readTree(json);
        } catch (JsonProcessingException e) {
            log.error("json={}", json, e);
            throw new JsonException(e);
        }
    }

}
~~~



~~~java
/**
 * 工具类Jsons内部抛出的异常，调用方可自行决定如何处理
 *
 * @author Deolin 2020-03-05
 * @see JsonUtils
 */
public class JsonException extends RuntimeException {

    private static final long serialVersionUID = 2506389302288058433L;

    public JsonException() {
        super();
    }

    public JsonException(String message) {
        super(message);
    }

    public JsonException(String message, Throwable cause) {
        super(message, cause);
    }

    public JsonException(Throwable cause) {
        super(cause);
    }

    protected JsonException(String message, Throwable cause, boolean enableSuppression, boolean writableStackTrace) {
        super(message, cause, enableSuppression, writableStackTrace);
    }

}
~~~



~~~java
/**
 * 对Jackson的核心 ObjectMapper 进行配置的工具类
 *
 * @author Deolin 2020-09-19
 */
public class ObjectMapperUtils {

    /**
     * 对 ObjectMapper 进行缺省化配置
     *
     * <pre>
     * 1. 时间相关的pattern缺省为yyyy-MM-dd HH:mm:ss、yyyy-MM-dd、HH:mm:ss，时区缺省为系统时区
     * 2. 发现并注册所有 jackson-datatype-* 依赖
     * 3. 反序列化时，忽略Javabean中不存在的属性，而不是抛出异常
     * 4. 反序列化时，忽略Javabean中Collection属性对应JSON Array中的为null的元素
     * </pre>
     *
     * @see ObjectMapperUtils#findAndRegister
     * @see ObjectMapperUtils#ignoreCollectionNullElement
     * @see ObjectMapperUtils#ignoreUnknownProperties
     * @see ObjectMapperUtils#setJavaUtilDateZone
     * @see ObjectMapperUtils#setJavaTimePattern
     * @see ObjectMapperUtils#setJavaUtilDatePattern
     */
    public static <T extends ObjectMapper> T initDefault(T om) {
        ObjectMapperUtils.findAndRegister(om);
        ObjectMapperUtils.ignoreCollectionNullElement(om);
        ObjectMapperUtils.ignoreUnknownProperties(om);
        ObjectMapperUtils.setJavaUtilDateZone(om, null);
        ObjectMapperUtils.setJavaTimePattern(om, "yyyy-MM-dd HH:mm:ss", "yyyy-MM-dd", "HH:mm:ss");
        ObjectMapperUtils.setJavaUtilDatePattern(om, "yyyy-MM-dd HH:mm:ss");
        return om;
    }

    /**
     * 使 ObjectMapper 自动发现和注册 jackson-datatype-* 所提供的Module
     */
    public static void findAndRegister(ObjectMapper om) {
        om.findAndRegisterModules();
    }

    /**
     * 使 ObjectMapper 在反序列化时，忽略Javabean中不存在的属性，而不是抛出异常
     */
    public static void ignoreUnknownProperties(ObjectMapper om) {
        om.configure(DeserializationFeature.FAIL_ON_UNKNOWN_PROPERTIES, false);
    }

    /**
     * 使 ObjectMapper 在反序列化时，忽略Javabean中Collection属性对应JSON Array中的为null的元素
     */
    public static void ignoreCollectionNullElement(ObjectMapper om) {
        om.registerModule(new IgnoreCollectionNullElementDeserializeModule());
    }

    /**
     * 为 ObjectMapper 配置时区
     *
     * @param timeZone 为null时缺省为TimeZone.getDefault()
     */
    public static void setJavaUtilDateZone(ObjectMapper om, TimeZone timeZone) {
        if (timeZone == null) {
            timeZone = TimeZone.getDefault();
        }
        om.setTimeZone(timeZone);
    }

    /**
     * 为 ObjectMapper 配置java.util.Date的全局pattern
     */
    public static void setJavaUtilDatePattern(ObjectMapper om, String pattern) {
        om.setDateFormat(new SimpleDateFormat(pattern));
    }

    /**
     * 为 ObjectMapper 分别配置（java.time.）LocalDateTime、LocalDate、LocalTime的全局pattern
     */
    public static void setJavaTimePattern(ObjectMapper om, String localDateTimePattern, String localDatePattern,
            String localTimePattern) {
        SimpleModule module = new JavaTimeModule();
        DateTimeFormatter ldtFormatter = DateTimeFormatter.ofPattern(localDateTimePattern);
        DateTimeFormatter ldFormatter = DateTimeFormatter.ofPattern(localDatePattern);
        DateTimeFormatter ltFormattter = DateTimeFormatter.ofPattern(localTimePattern);
        module.addSerializer(LocalDateTime.class, new LocalDateTimeSerializer(ldtFormatter))
                .addDeserializer(LocalDateTime.class, new LocalDateTimeDeserializer(ldtFormatter))
                .addSerializer(LocalDate.class, new LocalDateSerializer(ldFormatter))
                .addDeserializer(LocalDate.class, new LocalDateDeserializer(ldFormatter))
                .addSerializer(LocalTime.class, new LocalTimeSerializer(ltFormattter))
                .addDeserializer(LocalTime.class, new LocalTimeDeserializer(ltFormattter));
        om.registerModule(module);
    }

}
~~~

