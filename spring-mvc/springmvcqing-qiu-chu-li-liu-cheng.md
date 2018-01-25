# spring mvc 请求流程\(基于spring 4.3.x 分支\)

> ![](/assets/springmvc-process.jpg)图片引用自：[https://www.cnblogs.com/hujiapeng/p/5765636.html](https://www.cnblogs.com/hujiapeng/p/5765636.html) 借作者这张图阐述自己对spring mvc 的理解

## 核心概念

* DispatcherServlet

    中央调度器

* HandlerMapping

* HandlerAdapter

* ModelAndView
* HandlerExcutionChain
* HandlerInterceptor
* Handler
* ViewResolver

## 以后不要再问我

* 为什么我的请求参数接收不到?

* 我改用哪种请求方式？

* 我该用那么Content-Type?

* 为什么我什么都没返回却收到404?



