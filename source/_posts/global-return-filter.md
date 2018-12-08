---
title: 利用ResponseWrapper使请求接口能够返回统一结构

date: 2018-06-19 10:38:00

updated: 2018-06-19 10:38:00

tags:
- Spring

categories: Java

permalink: global-return-filter
---



## 源流

设计接口的时候一般都会有统一的包装结构，比如

~~~java
{
    "code":200,
    "data":{...}
    "message":null;
}
~~~



想要达到这样的效果，可以这样设计控制器——

~~~java
@GetMapping("/getSomething")
public ResultVO getSomething() {
    return ResultVO.success().setData(someService.getSomething());    
}
~~~



可以看到，如果想要统一的话，每个控制器都要写关于`ResultVO`的包装处理。



## 改善

我们可以借助**切面**，将关于`ResultVO`的处理封装起来——

~~~java
@Around("controllerMethod()")
public RequestBean around(ProceedingJoinPoint point) throws Throwable {
    // 其他的前置处理... 
    
    // 执行切点
    Object data = point.proceed(point.getArgs());
    
    // 其他的后置处理...
    
    if (data instanceof RequestBean) {
        return (RequestBean) data;
    }
    return ResultVO.success().setData(data);
}
~~~



这样做的话，控制器可以简练一点。

~~~java
@GetMapping("/getSomething")
public Object getSomething() {
    return someService.getSomething();    
}
~~~



这里有一个问题，控制器的返回类型，必须同时是`ResultVO`和`data`对象类型的父类。

如果返回类型是`Something`或是别的什么，最终返回的`ResultVO`对象会因为无法转化到返回类型而抛出异常。（你可以试试）



## 更好的方案

可以用**过滤器**，在最外层把Response的报文流替换掉。

想要替换报文，需要引入一个`ResponseWrapper`接口

~~~java
@Log4j2
@Component
public class RequestResultFilter implements Filter {

    @Override
    public void init(FilterConfig filterConfig) {
    }

    @Override
    @SneakyThrows
    public void doFilter(ServletRequest request, ServletResponse response, FilterChain chain) {
        /* 
        	这里放行一切请求（或是可以在这里过滤一下不需要统一返回数据结构的请求，如swagger）
         */
        
        ResponseWrapper wrapperResponse = new ResponseWrapper((HttpServletResponse) response);
        chain.doFilter(request, wrapperResponse);
        String content = new String(wrapperResponse.getContent());
        if (content.length() > 0) {
            // content是Response的json内容
            // 这里可以做一些处理，替换掉原json
        }
        ServletOutputStream out = response.getOutputStream();
        // 新的content覆盖掉原来的content
        out.write(content.getBytes());
        out.flush();
    }

    @Override
    public void destroy() {
    }

    private static class ResponseWrapper extends HttpServletResponseWrapper {

        private ByteArrayOutputStream buffer;

        private ServletOutputStream out;

        public ResponseWrapper(HttpServletResponse httpServletResponse) {
            super(httpServletResponse);
            buffer = new ByteArrayOutputStream();
            out = new WrapperOutputStream(buffer);
        }

        @Override
        public ServletOutputStream getOutputStream() {
            return out;
        }

        @Override
        public void flushBuffer() throws IOException {
            if (out != null) {
                out.flush();
            }
        }

        public byte[] getContent() throws IOException {
            flushBuffer();
            return buffer.toByteArray();
        }

        class WrapperOutputStream extends ServletOutputStream {

            private ByteArrayOutputStream bos;

            public WrapperOutputStream(ByteArrayOutputStream bos) {
                this.bos = bos;
            }

            @Override
            public void write(int b) {
                bos.write(b);
            }

            @Override
            public boolean isReady() {
                return false;

            }

            @Override
            public void setWriteListener(WriteListener arg0) {
            }
        }
    }

}
~~~



至此，控制层演变成了这样

~~~java
@GetMapping("/getSomething")
Something getSomething() {
    return someService.getSomething();    
}
~~~



返回data类型可以通过请求方法API直接得到反映，

非`public`方法也可以被Spring解析到，所以`public`也可以去掉。

可以说非常简练了。