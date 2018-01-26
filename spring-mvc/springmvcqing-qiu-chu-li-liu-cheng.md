# spring mvc 请求流程\(基于spring 4.3.x 分支\)

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
protected void doService(HttpServletRequest request, HttpServletResponse response) 
                                throws Exception{.....}
```

请求的处理是从doService方法开始的，我们在web.xml中如下方式注册了DispatcherServlet之后容器会根据url匹配，

```
    <servlet>
        <servlet-name>dispatcherServlet</servlet-name>
        <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
        <init-param>
            <param-name>contextConfigLocation</param-name>
            <param-value>classpath:/spring-mvc.xml</param-value>
        </init-param>
        <load-on-startup>1</load-on-startup>
    </servlet>
    <servlet-mapping>
        <servlet-name>dispatcherServlet</servlet-name>
        <url-pattern>/</url-pattern>
    </servlet-mapping>
```

如果匹配上，首先会调用FrameworkServlet中重写的

```
service(HttpServletRequest request, HttpServletResponse response)

这个方法的实现很简单，如果HTTP请求方法是PATCH,或者为null，就直接执行processRequest方法，
否则执行父类的service方法，那么父类的service方法就会根据请求的类型来对应调用do$HTTP_REQUEST_METHOD()方法,这do$HTTP_REQUEST_METHOD()方法的实现都是调用下面的
processRequest方法。
```

方法。然后这个方法中调用如下方法

```
processRequest(HttpServletRequest request, HttpServletResponse response)

这个方法做了一系列的初始化之后就会调用doService()方法，这样就到了DispatcherServlet处理求情的方法了，
核心的逻辑从这里开始了，如果想看懂核心逻辑是如何实现，请先继续看完一下核心概念,然后我们在继续讨论Dispatcher是如何工作的
```
### _HandlerExecutionChain_
Handler执行链，由一系列的Handler和Handler Intercptor组成，由HandlerMapping的getHandler方法返回。这样在spring执行handler的请求处理方法的时候会先执行一系列的Handler Intercptor。

### _Handler_
spring中有很多种Handler，常见的是HandlerMethod,用HandlerMethod表示一个Handler,并不是我们猜想的一个Controller或者一个Controller里面使用@RequestMapping注解的方法，请看下图，我们将理解HandlerExecutionChain和Handler，HandlerExecution之间的联系。
![](/assets/handler-chain.png)

mappedHandler是一个HandlerExecutionChain类型的对象,HandlerExecutionChain类型的对象包括了一个handler和一些列handler interceptor


### _HandlerMapping_

> Interface to be implemented by objects that define a mapping between  
>  requests and handler objects.  
>  这个接口会被那些定义了请求和处理器之间的映射关系的对象实现,他的实现类有BeanNameUrlHandlerMapping，DefaultAnnotationHandlerMapping，RequestMappingHandlerMapping等

**RequestMappingHandlerMapping**
    
    平时我们在controller中使用@RequestMapping定义的handler方法就是被这个类发现的，
    如果这个类不能正常工作，那么我们定义的接口八九不离十的会出现404了，看看他是怎么工作的。
**BeanNameUrlHandlerMapping**
    
    使用定义的Bean的名字去映射URLS

**DefaultAnnotationHandlerMapping**

    这个实现已经过时了，所以不做过多的讲解。

### _HandlerAdapter_
> MVC framework SPI, allowing parameterization of the core MVC workflow.
### _ModelAndView_

### _HandlerExcutionChain_

### _HandlerInterceptor_

### _Handler_

### _ViewResolver_

### _Aware_
>Marker superinterface indicating that a bean is eligible to be
notified by the Spring container of a particular framework object
through a callback-style method. Actual method signature is
determined by individual subinterfaces, but should typically
consist of just one void-returning method that accepts a single
argument.

**表示一个bean符合通过回调方法接受spring容器————一个特殊的框架对象的通知的条件。**，实际的方法签名是由某些特别的子接口提供定义的，不过典型的方法签名应该是一个只接受一个参数，void返回值的方法签名。

不过仅仅实现Aware接口并没有什么作用，因为他是一个标记接口，不提供任何的默认功能，如果要获得Aware表示的能力，那么应该显式的进行回调方法的处理，例如`BeanPostProcessor`，`AbstractAutowireCapableBeanFactory`，`ApplicationContextAwareProcessor` 等组件的实现方式。

### _ApplicationContextAware_

## 以后不要再问我

* 为什么我的请求参数接收不到?

* 我改用哪种请求方式？

* 我该用那么Content-Type?

* 为什么我什么都没返回却收到404?



