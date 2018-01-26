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