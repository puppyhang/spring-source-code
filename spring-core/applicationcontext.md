# spring 的容器\(俗称父容器\)

## 核心概念
**BeanFactory**
这是访问Spring容器的root接口，是spring容器对客户端的基础视图。更进一步的扩展他的接口具有更多功能的接口有比如`ListableBeanFactory`，`ConfigurableBeanFactory`等等

**DefaultListableBeanFactory:**
> DefaultListableBeanFactory 是整个spring容器最核心的组件，bean加载的核心组件，是spring bean注册和加载的默认实现。

**XmlBeanFactory**
> spring容器的基础，从xml配置文件中加载bean的定义。

**XmlBeanDefinitionReader**
从XML中读取bean的定义，然后将bean注册到BeanDefinitionRegistry中，这个接口试专门用来给BeanFactory实现的，为了持有bean的所有定义。

什么是spring的容器？