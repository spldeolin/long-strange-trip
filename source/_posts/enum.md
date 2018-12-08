title: 枚举（enum）
date: 2017-11-23 09:05:00

categories: Java

permalink: enum
---

枚举的一大作用就是代替常量类。

~~~java
public class BasicsPersonConstant {
    
    public static final int TEACHER_CODE = "1";

    public static final String TEACHER_DISPLAY = "教师"; 

    public static final int STUDENT_CODE = "2";

    public static final String STUDENT_DISPLAY = "学生";

    public static final int PARENT_CODE = "3";

    public static final String PARENT_DISPLAY = "家长";

}
~~~

~~~java
public enum BasicsPerson {
    TEACHER(1, "教师"), STUDENT(2, "学生"), PARENT(3, "家长");

    private Integer code;

    private String display;

    private BasicsPerson(Integer code, String display) {
        this.code = code;
        this.display = display;
    }

    public Integer getCode() {
        return code;
    }

    public void setCode(Integer code) {
        this.code = code;
    }

    public String getDisplay() {
        return display;
    }

    public void setDisplay(String display) {
        this.display = display;
    }
}
~~~

原先的`BasicsPersonContant.STUDENT_CODE`可以用 `BasicsPerson.STUDENT.getCode()`代替。

同时`BasicsPerson.STUDENT`可以作为一个对象来作为参数传递。

两个枚举之间支持`==`直接判断是否相同。



如果枚举仅仅作为区分用，可以不需要那些属性。（这种情况在枚举作用域比较小的时候比较多见）

~~~
enum Judgment {
	EXECUTION, PARDON, DELAY;
}
~~~

