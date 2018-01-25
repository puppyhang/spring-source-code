```
spring mvc 请求流程(基于spring 4.3.x 分支)
```

> ![](/assets/springmvc-process.jpg)图片引用自：[https://www.cnblogs.com/hujiapeng/p/5765636.html](https://www.cnblogs.com/hujiapeng/p/5765636.html) 借作者这张图阐述自己对spring mvc 的理解

## 核心概念

### _DispatcherServlet_

> _Central dispatcher dor HTTP request handlers/controllers    _**中央调度器**

首先纠正一个概念，大多数看网上文章"长大的"人都会把这个组件称之为前端控制器，但事实上更适合他的名称应该是中央调度器，在spring的源码中关于这个组件的文档的第一句话就写上以上的解释。下文就间的使用dispatcher来代表DispatcherServlet。

这个类见名知意，他是一个Servlet，基于tomcat-embed-core-8.5.23，实现的是servlet 3.1 标准。下面开始从源码中讲解其处理请求的流程

首先问一个问题：**spring可以接受用户发起的请求吗？**

答案是否定的，spring并不可以接受用户发起的请求，接受请求的是servlet容器，比如tomcat/jetty/undertow\['ʌndətəʊ\]

回答了这个问题我们就可以找到理解DispatcherServlet的切入点了，既然他是一个Servlet，那么他肯定可以接受到容器转发给他的请求，也就是会执行 **doGet/doPost/doPut/doDelete/doOptions/service\(\)  **等方法的其中一个，这样他才有机会处理用户的请求。故事就这样从这里开始了😁😂！让我一起开始这愉快的旅程吧。

![](/assets/dispatcher-servlet-uml.png)

```
protected void doService(HttpServletRequest request, HttpServletResponse response) throws Exception{.....}
```

请求的处理是从doService方法开始的

### _HandlerMapping_

### _HandlerAdapter_

### _ModelAndView_

### _HandlerExcutionChain_

### _HandlerInterceptor_

### _Handler_

### _ViewResolver_

## 以后不要再问我

* 为什么我的请求参数接收不到?

* 我改用哪种请求方式？

* 我该用那么Content-Type?

* 为什么我什么都没返回却收到404?



