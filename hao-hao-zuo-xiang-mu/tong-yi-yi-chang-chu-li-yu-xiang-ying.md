---
description: 基础组件工程，提供整个项目的可复用类、功能
---

# 统一异常处理与响应

SpringMVC为自定义异常处理，提供两种嵌入策略。

1. HandlerExceptionResolve 接口实现
2. @ExceptionHandler  添加注解



DispatcherServlet，轮询 HandlerExceptionResolve的实现类，执行 resolveException方法。其中一个实现类，ExceptionHandlerExceptionResolve 是实现 @ExceptionHandler注解的异常处理逻辑。



为什么需要 @ControllerAdvice

@RestControllerAdvice 与 @ControllerAdvice 的关系







