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

### _Handler_  _HandlerInterceptor_

spring中有很多种Handler，常见的是HandlerMethod,用HandlerMethod表示一个Handler,并不是我们猜想的一个Controller或者一个Controller里面使用@RequestMapping注解的方法，请看下图，我们将理解HandlerExecutionChain和Handler，HandlerExecution之间的联系。  
![](/assets/handler-chain.png)

mappedHandler是一个HandlerExecutionChain类型的对象,HandlerExecutionChain类型的对象包括了一个handler和一些列handler interceptor。handler是用于保存处理请求器信息的对象，interceptor是拦截器，会在handler中处理请求的方法被执行之前被执行。  
handler:常用的handler的类型是HandlerMethod，被@Controller和@RequestMapping标记的类型或方法才可以被称之为处理器，handler里面包括了bean：处理这个用户请求类的bean，beanFactory：创建这个bean的工厂，beanType：这个bean的类型，method：处理用户请求的bean中执行请求的方法，paramters：方法执行需要的参数。所以现在我们理解了Handler不是我们理解的Controller或者Controller中的方法，这种理解是狭义的。

### _HandlerMapping_

> Interface to be implemented by objects that define a mapping between  
>  requests and handler objects.  
>  这个接口会被那些定义了请求和处理器之间的映射关系的对象实现,他的实现类有BeanNameUrlHandlerMapping，DefaultAnnotationHandlerMapping，RequestMappingHandlerMapping等

**RequestMappingHandlerMapping**

```
平时我们在controller中使用@RequestMapping定义的handler方法就是被这个类发现的，
如果这个类不能正常工作，那么我们定义的接口八九不离十的会出现404了，看看他是怎么工作的。
```

**BeanNameUrlHandlerMapping**

```
使用定义的Bean的名字去映射URLS
```

**DefaultAnnotationHandlerMapping**

```
这个实现已经过时了，所以不做过多的讲解。
```

### _HandlerAdapter_

> MVC framework SPI, MVC 框架 “服务提供接口”\(SPI\)
>
> SPI的全名是Service Provider Interface，在java.util.ServiceLoader里面有比较详细的介绍。  
> 不必过于执着于这些细杂的概念，只需要知道mvc框架提供的服务都是通过这个接口来实现的，我们可以通过实现这个接口实现我们自己的HandlerAdapter，他的作用之一也是使得DispatcherServlet能够独立，灵活可扩展。

上文我们讲完了[Hadnler ](#Handler)[HandlerIntercptor](#Handler)和 [HandlerExcutionChain](#HandlerExecutionChain), HandlerAdapter会根据HandlerMapping找到支持这种mapping规则的HandlerAdapter，然后调用handle方法执行用户定义的请求处理方法。然后返回ModelAndView。

### _ModelAndView_

> model 表示数据，view表示视图，合起来就是数据和视图的封装对象，controller中的@RequestMapping修饰的方法返回这个对象，spring就回去使用ViewResoler解析页面，并使用模板渲染数据。

### _ViewResolver_

> 视图解析器，根据我们在XML中的配置，他会找到放在项目中的页面

# Dispatcher Servlet 请求处理流程（也就是springmvc请求处理流程）

1：根据请求的信息找到HandlerExecutionChain  
2：根据HandlerExecutionChain中的handler找到能够支持这种handler的handlerAdapter  
3：由HandlerAdapter执行拦截器和请求的处理器方法  
4：清楚了以上流程我们就可以自定义自己的mapping和adapter了，只要注册给springmvc就可以了。

# 读完本文之后以后不要再问我

* 为什么我的请求参数接收不到?

* 我该用哪种请求方式？

* 为什么我什么都没返回却收到404?



[^1]: Enter footnote here.

