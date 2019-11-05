---

title: 如何校验枚举值？

date: 2019-11-05 09:43

updated: 2019-11-05 09:43

tags:
- Spring

categories: Java

permalink: validate-enum-value

---

## 源码

需要3个东西

- 一个接口，实现了该接口的枚举可以参与校验
- 一个校验注解，声明在需要被校验的field上
- 一个校验器，作为校验的具体实现

~~~java
/**
 * 这个接口的派生枚举类 需要实现对枚举值的是否有效的校验
 *
 * @author Deolin 2019-10-29
 */
public interface ValidityInterpretable {
    /**
     * 判断value对该enum而言是否是有效的枚举值
     */
    boolean isValid(Integer value);
}
~~~

~~~java
/**
 * “有效的枚举值”校验注解
 * <pre>
 * 支持类型：Integer
 * 规则：11位数字，以1开头
 * </pre>
 *
 * @author Deolin 2019-10-29
 */
@Documented
@Constraint(validatedBy = {ValidEnumValueValidator.class})
@Target({METHOD, FIELD, ANNOTATION_TYPE, CONSTRUCTOR, PARAMETER})
@Retention(RUNTIME)
public @interface ValidEnumValue {
    String message() default "不是有效的枚举值";
    Class<?>[] groups() default {};
    Class<? extends Payload>[] payload() default {};
    Class<? extends ValidityInterpretable> enumType();
}
~~~

~~~java
/**
 * “有效枚举值”校验器
 *
 * @author Deolin 2019-10-29
 */
public class ValidEnumValueValidator implements ConstraintValidator<ValidEnumValue, Integer> {
    private ValidityInterpretableEnum validityInterpretableEnum;
    private boolean isEmpty;
    @Override
    public void initialize(ValidEnumValue constraintAnnotation) {
        ValidityInterpretableEnum[] enumConstants = constraintAnnotation.enumType().getEnumConstants();
        if (enumConstants.length == 0) {
            isEmpty = true;
        }
        validityInterpretableEnum = enumConstants[0];
    }
    @Override
    public boolean isValid(Integer value, ConstraintValidatorContext context) {
        if (value == null) {
            return true;
        }
        if (isEmpty) {
            return false;
        }
        return validityInterpretableEnum.isValid(value);
    }
}
~~~



## 用法

~~~java
/**
 * VIP类型枚举
 */
@AllArgsConstructor
@Getter
public enum VipTypeEnum implements ValidityInterpretableEnum {

    NORMAL(1), SUPER(2);

    private Integer value;

    @Override
    public boolean isValid(Integer value) {
        // 由用户代码定义 “怎样算有效的？” 这件事
        return Arrays.stream(VipType.values()).anyMatch(one -> one.getValue().equals(value));
    }
}
~~~



~~~java
@Data
public class SimpleUserDTO {
    @NotNull
    private Long id;
        
    @NotBlank
    @Size(max = 6)
    private String name;

    @@ValidEnumValue(enumType = VipTypeEnum.class)
    private Integer vipType;
}
~~~

