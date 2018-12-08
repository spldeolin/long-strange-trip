---
title: 统一异常处理

date: 2018-05-21 18:46:64

tags:
- Spring

categories: Java

permalink: global-exception-handler
---

## 简介

这篇POST将全面的介绍Spring项目中有关统一异常处理的一切，以及如何为一个项目设计统一异常处理



## 事前准备

设计统一异常处理的初衷是——将服务端一切的情况，无论是成功、失败、BUG，还是别的什么情况，都以JSON的形式，将信息返回给客户端。

基于这点，我们需要一个数据结构，用于规范化这些信息。

~~~java
/**
 * 控制器通用返回类型
 *
 * @author Deolin
 */
@Data
@Accessors(chain = true)
@JsonInclude(JsonInclude.Include.NON_NULL)
public class RequestResult {

    /**
     * 请求结果
     */
    private Integer code;

    /**
     * 请求成功时，需要返回的数据
     */
    private Object data;

    /**
     * 请求失败时，需要返回的错误信息
     */
    private String message;

    private RequestResult() {}

    public static RequestResult success() {
        return success(null);
    }

    public static RequestResult success(Object data) {
        RequestResult instance = new RequestResult();
        instance.setCode(ResultCode.OK.getCode());
        instance.setData(data);
        return instance;
    }

    public static RequestResult failure(ResultCode code) {
        return failure(code, code.getDefaultMessage());
    }

    public static RequestResult failure(ResultCode code, String message) {
        RequestResult instance = new RequestResult();
        instance.setCode(code.getCode());
        instance.setMessage(message);
        return instance;
    }

}
~~~



`请求结果`会比较多，所以设计一个枚举。

~~~java
public enum ResultCode {

    OK(200, null),
    INTERNAL_ERROR(500, "内部错误");

    private Integer code;

    private String defaultMessage;
    
    ResultCode(Integer code, String defaultMessage) {
        this.code = code;
        this.defaultMessage = defaultMessage;
    }

    // Lombok不支持enum，省略Getter、Setter
    
}
~~~



`请求结果`不会仅仅只有2个，先保持这样，然后慢慢追加其他情况...



## 请求成功的场合

这种情况没有任何特殊的处理，直接用`RequestResult`包装一下控制层的每个返回值就可以了。

~~~java

	@GetMapping("user")
	public RequestResult getUser() {
        return RequestResult.success(userService.listAllUsers());
	}

~~~



## 基于注解的异常捕获

在Spring MVC中，控制层中未被处理的异常会被框架捕获，进行缺省处理，最终，客户端会获得一个Http Status不是200的一个回应，以及一大篇比较不友好的异常信息。

我们可以在每个控制层都捕获异常，然后处理掉。

~~~java
	@GetMapping("user")
	public RequestResult user() {
        try {
            RequestResult.success(userService.listAllUsers());
        } catch(Throwable t) {
            log.info("内部错误", t);
            RequestResult.failure(ResultCode.INTERNAL_ERROR);
        }
	}
~~~



我们显然不可能为每个请求都追加异常处理的代码，那样太不优雅了。

更合适的方法可能是用切面。

~~~java
    @Pointcut("@within(org.springframework.web.bind.annotation.RestController)")
    public void controllerMethod() {}

    @Around("controllerMethod()")
    public Object afterThrowing(ProceedingJoinPoint point) throws Throwable {
        try {
            return point.proceed(point.getArgs());
        } catch (Throwable t) {
            log.info("内部错误", t);
            return RequestResult.failure(ResultCode.INTERNAL_ERROR);
        }
    }
~~~



代码被抽取出来了，优雅了不少。

但似乎依然有一些不优雅的地方，如果我需要细分异常类型，不同的异常做不同的处理，那么`afterThrowing()`就需要追加大量的`catch`语句块，或是在`catch (Throwable)`分支中，通过丑陋的`instanceof`，来分区具体的异常。



所幸，Spring为我们提供了一种非常优雅的方式，`@RestControllerAdvice`和`@ExceptionHandler`的注解组合——

~~~java
/**
 * 控制层增强：统一异常处理
 *
 * @author Deolin
 */
@Component
@RestControllerAdvice
@Log4j2
public class GlobalExceptionAdvance {

	@ExceptionHandler(Throwable.class)
    public RequestResult handle(Throwable e) {
    	return RequestResult.failure(ResultCode.INTERNAL_ERROR);
    }

}
~~~



有了`GlobalExceptionAdvance`以后，控制层里可以放心的抛异常，所有异常最终会进入`GlobalExceptionAdvance.handle()`方法，这个方法的返回值，会代替掉控制层原来的返回值。



## 业务异常的场合 

有了`GlobalExceptionAdvance`兜底，我们可以直接将业务异常放心抛出去。

~~~java
	public School getSchoolByTeacherId(Long teacherId) {
        Long schoolId = Optional.ofNullable(teacherService.getOne(teacherId)).orElseThrow(()
        		-> new ServiceException("教师不存在或是已被删除。")).getSchoolId();
        return schoolService.getOne(schoolId);
    }
~~~

> Optional和Lambda优雅到窒息。



开发者主动抛出的`ServiceException`是想告诉客户端为什么流程进行不下去了，所以不能再让`ServiceException`被`@ExceptionHandler(Throwable.class)`捕获了，我们需要追加一个处理器

~~~java
    /**
     * 1001 业务异常
     */
    @ExceptionHandler(ServiceException.class)
    public RequestResult handle(ServiceException e) {
        return RequestResult.failure(ResultCode.SERVICE_ERROR, e.getMessage());
    }
~~~



现在，如果业务层想告诉客户端流程无法执行下去了，以及为什么执行不下去，仅仅只需要抛出一个`ServiceException`，而不需要用`if-else`控制流程，返回一个异常信息了。

同时，`@ExceptionHandler`比起切面的优势也体现出来了，每当需要多考虑一种异常时，开发者只需要追加一个`@ExceptionHandler`方法就可以了。



## 交互失败的场合

客户端没能正确的调用服务端提供的接口，也是异常的一种。

虽然发生这种情况基本上是客户端开发者的问题，但是，服务端方面如果能返回一个标准格式的错误信息，会显得服务端开发者十分绅士。

为了成为充满绅士风度的服务端开发者，我们应该追加一下应对交互失败的异常处理器。

~~~java
    /**
     * 400 请求动词错误
     */
    @ExceptionHandler(HttpRequestMethodNotSupportedException.class)
    public RequestResult handle(HttpRequestMethodNotSupportedException e) {
        return RequestResult.failure(ResultCode.BAD_REQEUST,
                "请求动词不受支持，当前为[" + e.getMethod() + "]，正确为" + Arrays.toString(e.getSupportedMethods()));
    }

    /**
     * 400 请求Content-Type错误
     * 往往是因为后端的@RequestBody和前端的application/json没有同时指定或同时不指定。
     */
    @ExceptionHandler(HttpMediaTypeNotSupportedException.class)
    public RequestResult handle(HttpMediaTypeNotSupportedException e) {
        return RequestResult.failure(ResultCode.BAD_REQEUST,
                "Content-Type错误，当前为[" + e.getContentType().toString().replace(";charset=UTF-8", "") +
                        "]，正确为[application/json]");
    }

    /**
     * 400 缺少RequestParam参数
     */
    @ExceptionHandler(MissingServletRequestParameterException.class)
    public RequestResult handle(MissingServletRequestParameterException e) {
        return RequestResult.failure(ResultCode.BAD_REQEUST, "缺少请求参数" + e.getParameterName());
    }

    /**
     * 400 RequestParam参数类型错误
     */
    @ExceptionHandler(MethodArgumentTypeMismatchException.class)
    public RequestResult handle(MethodArgumentTypeMismatchException e) {
        return RequestResult.failure(ResultCode.BAD_REQEUST, e.getName() + "类型错误");
    }

    /**
     * 400 RequestParam参数没有通过注解校验（控制器声明@Validated时）
     */
    @ExceptionHandler(ConstraintViolationException.class)
    public RequestResult handle(ConstraintViolationException e) {
        Set<ConstraintViolation<?>> cvs = e.getConstraintViolations();
        ControllerInfo controllerInfo = RequestContextUtils.getControllerInfo();
        String[] paramNames = controllerInfo.getParameterNames();
        Object[] paramValues = controllerInfo.getParameterValues();
        List<Invalid> invalids = new ArrayList<>();
        for (ConstraintViolation<?> cv : cvs) {
            // 参数路径
            PathImpl pathImpl = (PathImpl) cv.getPropertyPath();
            // 参数下标
            int paramIndex = pathImpl.getLeafNode().getParameterIndex();
            invalids.add(Invalid.builder().name(paramNames[paramIndex]).value(paramValues[paramIndex]).cause(cv.getMessage()).build());
        }
        return RequestResult.failure(ResultCode.BAD_REQEUST, "数据校验未通过").setData(invalids);
    }

    /**
     * 400 请求Body格式错误
     * <pre>
     * 以下情况时，会被捕获：
     * alkdjfaldfjlalkajkdklf
     * (空)
     * {"userAge"="notNumberValue"}
     * </pre>
     */
    @ExceptionHandler(HttpMessageNotReadableException.class)
    public RequestResult httpMessageNotReadable() {
        return RequestResult.failure(ResultCode.BAD_REQEUST, "请求Body不可读。可能是JSON格式错误，或JSON不存在，或类型错误");
    }

    /**
     * 400 请求Body内字段没有通过注解校验（形参声明@Valid时）
     */
    @ExceptionHandler(MethodArgumentNotValidException.class)
    public RequestResult handle(MethodArgumentNotValidException e) {
        BindingResult bindingResult = e.getBindingResult();
        List<Invalid> invalids = new ArrayList<>();
        for (FieldError fieldError : bindingResult.getFieldErrors()) {
            invalids.add(Invalid.builder().name(fieldError.getField()).value(fieldError.getRejectedValue()).cause(
                    fieldError.getDefaultMessage()).build());
        }
        return RequestResult.failure(ResultCode.BAD_REQEUST, "数据校验未通过").setData(invalids);
    }

	
~~~

> 其中`MethodArgumentNotValidException`和`ConstraintViolationException`的处理器涉及到**数据校验**的设计，Deolin将会在后续的POST中详细介绍。



到目前为止，统一异常处理覆盖了几乎所有的交互失败情况，客户端开发者基本上可以根据返回值，自行判断自己错哪了。



## Spring Boot自带的统一异常处理

Spring Boot自带了一套默认的统一异常处理，他的流程大致是将异常捕获，存入request对象，然后将请求转发到`org.springframework.boot.autoconfigure.web.BasicErrorController`下的请求方法，方法内部通过解析request对象的attribute，包装出一个`ModelAndView对象`对象或是`ResponseEntity` （取决于请求是不是`text/html`）。



但是，我们自己的统一异常处理已经捕获`Throwable`了，难道还会有什么异常能够达到Spring Boot自带的统一异常处理吗？



实际上是有的，情况有3种

1. 当项目有过滤器（filter），而且过滤器中抛出任何异常时；
2. 最特殊的交互异常，当客户端调用一个不存在的请求时；（即http404）
3. 当项目集成actuator，没有指定`security=false`，调用`actuator`提供的接口时；



这3种情况统一异常处理无法直接捕获，因为它们都发生在控制层的外侧。

想要将这3种情况也用我们的统一异常处理捕获掉，可以从默认的请求转发那里开始重写。



~~~java
/**
 * 控制器：提供URL转发到统一异常处理的映射
 */
@Controller
@RequestMapping
@ApiIgnore
public class UrlForwardToExceptionController implements ErrorController {

    public static final String ERROR_PATH = "/error";

    @Override
    public String getErrorPath() {
        return ERROR_PATH;
    }

    /**
     * 结合被重写的getErrorPath方法，原先所有的“Whitelabel Error Page”都会转发给这个请求方法，
     * 这个请求方法会参照BasicErrorController，解析request后，转发给统一异常处理。
     *
     * @see DefaultErrorAttributes#addStatus(java.util.Map, org.springframework.web.context.request.RequestAttributes)
     * @see DefaultErrorAttributes#getError(org.springframework.web.context.request.RequestAttributes)
     */
    @RequestMapping(ERROR_PATH)
    public void error(HttpServletRequest request) throws Throwable {
        int status = (Integer) request.getAttribute("javax.servlet.error.status_code");
        if (status == HttpStatus.NOT_FOUND.value()) {
            throw new RequestNotFoundException();
        } else {
            Throwable throwable = (Throwable) request.getAttribute(
                    "org.springframework.boot.autoconfigure.web.DefaultErrorAttributes.ERROR");
            if (throwable == null) {
                throwable = (Throwable) request.getAttribute("javax.servlet.error.exception");
            }
            throw throwable;
        }
    }

}
~~~



这样一来，默认的异常处理将会转发到这个控制层，方法内部解析好request以后，根据情况抛出异常。

`RequestNotFoundException`是一个新追加的类，所以统一异常处理那里也要追加一个处理器

~~~java
    @ExceptionHandler(RequestNotFoundException.class)
    public RequestResult handleRequestNotFoundException() {
        return RequestResult.failure(ResultCode.NOT_FOUND);
    }
~~~



## 其他异常场合

如果项目集成了安全框架，如Shiro，那还需要追加一些异常处理器，比如`AuthorizationException`之类的....









源码可以参照[spldeolin](https://github.com/spldeolin) / [beginning-mind](https://github.com/spldeolin/beginning-mind)