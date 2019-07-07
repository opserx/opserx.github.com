---
title: Spring Boot MVC 400异常不打印
---
Spring Boot2对于异常的统一处理有所升级，这篇文章的介绍很详细：

[Spring5 REST Controller的异常控制](https://www.baeldung.com/exception-handling-for-rest-with-spring)

[Spring 的*ResponseStatusException*介绍](https://www.baeldung.com/spring-response-status-exception)

Spring boot异常控制的知识在第一篇中已经讲的比较系统了，其中的重点是：

> **we can implement a @ControllerAdvice globally, but also ResponseStatusExceptions locally.**

除此之外，和异常控制有关的几个配置如下：
```
spring.mvc.log-resolved-exception=false
# Whether to enable warn logging of exceptions resolved by a "HandlerExceptionResolver", except for"DefaultHandlerExceptionResolver".
```

```
spring.mvc.throw-exception-if-no-handler-found=false
# Whether a "NoHandlerFoundException" should be thrown if no Handler was found to process a request.
```
有时候发现即便配置了，仍然不能打印异常，原因是Spring会默认给你加上ResourceHttpRequestHandler这个handler，也就不会出现noHandler的情况了，该handler是用来处理资源使用的。此时需要修改以下配置：
```
spring.resources.add-mappings=false
# Whether to enable default resource handling.
```
其实事实并不完全如此，400异常仍然不会打印。
其实400、405这些异常常遇到，并且很容易被忽视导致调试困难，因为这样的问题是在前端发出请求后、但在后端处理请求之前发生。

## 终极解决方案

```java
@ControllerAdvice
@Slf4j
public class RestResponseEntityExceptionHandler
        extends ResponseEntityExceptionHandler {
  @Override
    protected ResponseEntity<Object> handleExceptionInternal(
            Exception ex, @Nullable Object body, HttpHeaders headers, HttpStatus status, WebRequest request) {
        log.debug("API调用异常:{}", ex.getMessage(), ex);
        return super.handleExceptionInternal(ex, body, headers, status, request);
    }
}
```

如果是Spring mvc在组装参数阶段出错或者RequestMapping错误，都会打印出异常，对于程序调试很有帮助。
