---
title: Velocity模板引擎

date: 2018-12-08 16:37:00

updated: 2018-12-08 16:37:00

tags:
- 

categories: Java

permalink: 
---



源码可以参照[spldeolin](https://github.com/spldeolin) / [beginning-mind](https://github.com/spldeolin/beginning-mind)





~~~
@Log4j2
public class VelocityDemo {

    public static void main(String[] args) {
        // 初始化引擎
        VelocityEngine engine = new VelocityEngine();
        engine.init(classpathResourceLoaderProp());

        // 添加参数
        VelocityContext context = new VelocityContext();
        context.put("name", "Deolin");

        // 渲染
        StringWriter writer = new StringWriter();
        engine.getTemplate("tem/helloworld.vm").merge(context, writer);

        // 获取渲染完毕的数据
        log.info(writer.toString());
    }

    private static Properties classpathResourceLoaderProp() {
        Properties prop = new Properties();
        prop.setProperty(RuntimeConstants.RESOURCE_LOADER, "classpath");
        prop.setProperty("classpath.resource.loader.class", ClasspathResourceLoader.class.getName());
        return prop;
    }

}
~~~

