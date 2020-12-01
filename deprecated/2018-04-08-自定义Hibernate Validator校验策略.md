---
title: 自定义Hibernate Validator校验策略

date: 2018-04-08 18:27

updated: 2018-04-08 18:27

tags: Spring

categories: Java

permalink: custom-hibernate-validator
---

过时原因：

## 简介

Spring结合了Hibernate Validator框架之后，可以做比较全面、方便的数据校验。框架本身提供的校验注解基本上已经面面俱到了，但是，我们依然可以二次开发一些较为细化的、更接近业务的**自定义校验注解**，来适当简化一下高复用性的校验策略。

下面，Deolin将以一个“可选项校验”为例，演示一下如何自定义校验注解

## 创建校验注解

~~~java
/**
 * “文本可选项”校验用注解
 * <pre>
 * 支持类型：CharSequence
 * 规则：必须是{value}的其中之一，才能通过校验。如果不指定{value}，则本注解不会产生任何作用
 * </pre>
 */
@Documented
@Constraint(validatedBy = {TextOptionValidator.class})
@Target({METHOD, FIELD, ANNOTATION_TYPE, CONSTRUCTOR, PARAMETER})
@Retention(RUNTIME)
public @interface TextOption {

    String message() default "不是可选项";

    Class<?>[] groups() default {};

    /**
     * 文本可选项数组
     */
    String[] value() default {};

    /**
     * 是否忽略单词大小写
     */
    boolean ignoreCase() default false;

    Class<? extends Payload>[] payload() default {};

}
~~~

1. `@Constraint`中指定了校验器
2. `groups()`是为了支持Hibernate Validator框架的分组校验功能

## 创建校验器

~~~java
/**
 * “文本可选项”校验器
 *
 * @author Deolin
 */
public class TextOptionValidator implements ConstraintValidator<TextOption, CharSequence> {

    private String[] options;

    private boolean ignoreCase;

    @Override
    public void initialize(TextOption constraintAnnotation) {
        options = constraintAnnotation.value();
        ignoreCase = constraintAnnotation.ignoreCase();
    }

    @Override
    public boolean isValid(CharSequence value, ConstraintValidatorContext context) {
        if (value == null || options == null || options.length == 0) {
            return true;
        }
        String string = value.toString();
        for (String option : options) {
            if (ignoreCase) {
                if (option.equalsIgnoreCase(string)) {
                    return true;
                }
            } else {
                if (option.equals(string)) {
                    return true;
                }
            }
        }
        return false;
    }

}
~~~

1. 第一个类级泛型用于指定校验注解，第二个类级泛型用于指定被校验属性的类型
2. `initialize()`用于提取校验注解中指定的属性
3. `isValid()`用于校验，返回`true`则代表校验通过了，否则代表未通过

## 使用校验注解

至此，自定义完成，可以使用TextOption了

自定义校验注解和自带的校验注解一样，都支持`@Valid`或`@Validate`方式的启用

### 方法级校验

~~~java
@Service
@Validate
public class TestServiceImpl implements TestService() {
    
    public void justCheck(@TestOption({"a", "b", "c"}) String s) {
        System.out.println(s);
    }

}

~~~

~~~java
	@Autowired
	private TestService testService;

	@Test
	public test() {
        testService.justCheck("d");
	}

~~~

### Spring参数绑定校验

~~~java
@Data
public class OneCookie {

    @TestOption(value = {"exciting cookie", "interesting cookie"}, ignoreCase = true)
    private String name;

}
~~~

~~~java
	@GetMapping("test")
	public String test(@RequestBody @Valid OneCookie oneCookie) {
        return "SUCCESS";
	}
~~~

~~~json
{"name": "IntEreStiNg cOokIe"}
~~~

