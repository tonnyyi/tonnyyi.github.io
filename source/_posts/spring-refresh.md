---
title: spring-refresh方法分析
tags:
  - null
categories:
  - null
date: 2019-02-22 14:58:15
---

> spring源码版本: 4.3.1-RELEASE

`AbstractApplicationContext#refresh`
```java
public void refresh() throws BeansException, IllegalStateException {
    synchronized (this.startupShutdownMonitor) {
        // 设置启动时间和启动标识active为true
        // 如果是web应用, 则会替换servletContextInitParams和servletConfigInitParams配置
        prepareRefresh();

        // 创建DefaultListableBeanFactory, 加载BeanDefinition
        ConfigurableListableBeanFactory beanFactory = obtainFreshBeanFactory();

        // 设置BeanClassLoader, PropertyEditor注册器, 一些AwareProcessor
        // 把一些Aware接口排除出autowire
        prepareBeanFactory(beanFactory);

        try {
            // 给子类处理BeanFactory的机会, web容器时会增加一个BeanPostProcessor和排除一些Aware接口
            postProcessBeanFactory(beanFactory);

            // 初始化并且执行已经注册的BeanFactoryPostProcessor
            invokeBeanFactoryPostProcessors(beanFactory);

            // 初始化并执行已经注册的BeanPostProcessor
            registerBeanPostProcessors(beanFactory);

            // 初始化上下文中的资源文件，如国际化文件的处理等
            initMessageSource();

            // 初始化容器事件广播器
            initApplicationEventMulticaster();

            // 给子类参与refresh的机会, web容器会在这里初始化主题资源
            onRefresh();

            // 注册容器事件监听器
            registerListeners();

            // 初始化剩余的非懒加载的单例bean, 开发人员定义的bean基本上是在这里实例化的
            finishBeanFactoryInitialization(beanFactory);

            // 初始化LifecycleProcessor, 发布容器刷新事件
            finishRefresh();
        }

        catch (BeansException ex) {
            if (logger.isWarnEnabled()) {
                logger.warn("Exception encountered during context initialization - " +
                        "cancelling refresh attempt: " + ex);
            }

            // 销毁所有已经创建的单例
            destroyBeans();

            // 重置active标识
            cancelRefresh(ex);

            // Propagate exception to caller.
            throw ex;
        }

        finally {
            // Reset common introspection caches in Spring's core, since we
            // might not ever need metadata for singleton beans anymore...
            resetCommonCaches();
        }
    }
}
```

## prepareRefresh()

主要是设置当前`ApplicationContext`的启动时间, `active`和`closed`标志位; 执行留给子类的扩展方法`initPropertySources()`; 验证系统必填属性(Spring默认没有配置必填属性)

```java
protected void prepareRefresh() {
    this.startupDate = System.currentTimeMillis();
    this.closed.set(false);
    this.active.set(true);

    if (logger.isInfoEnabled()) {
        logger.info("Refreshing " + this);
    }

    // 留给子类初始化属性资源
    initPropertySources();

    // 验证必填属性
    getEnvironment().validateRequiredProperties();

    // Allow for the collection of early ApplicationEvents,
    // to be published once the multicaster is available...
    this.earlyApplicationEvents = new LinkedHashSet<ApplicationEvent>();
}
```
* **`initPropertySources()`**在`AbstractApplicationContext`中实现是空的, 这是留给子类覆盖的, 这样就给了用户扩展的能力. 在web应用中, 子类`AbstractRefreshableWebApplicationContext`就利用覆盖整个方法将Servlet属性`ServletContext`和`servletConfig`注册到了系统属性资源中.
  
* **`getEnvironment().validateRequiredProperties()`**的实现是在`AbstractEnvironment`中, 整个方法用来验证一些必填属性, 而Spring默认是没有设置必填属性的, 所以这个方法的意义是什么呢? 扩展性! 如果我们自己有校验必填属性的需求, 比如我们要求应用启动时必须设置日志目录参数`logDir`, 就可以创建自己的ApplicationContext, 继承`XmlWebApplication`, 然后通过重写`initPropertySources()`方法设置该属性必填. 
    ```java
    public class MyApplicationContext extends XmlWebApplicationContext {
        @Override
        protected void initPropertySources() {
            super.initPropertySources();
            getEnvironment().setRequiredProperties("logDir");
        }
    }
    ```
    当我们运行项目时可以通过JVM参数`-DlogDir=/tmp/log`传递属性值, 如果不传递应用就会抛出`MissingRequiredPropertiesException`异常. 当然我们需要在`web.xml`中通过如下配置指定使用我们的自定义的`MyApplicationContext`.
    ```xml
    <context-param>
        <param-name>contextClass</param-name>
        <param-value>com.tonny.study.MyApplicationContext</param-value>
    </context-param>
    ```
    

## obtainFreshBeanFactory()

初始化BeanFactory, 加载BeanDefinition

```java
protected ConfigurableListableBeanFactory obtainFreshBeanFactory() {
    refreshBeanFactory();
    ConfigurableListableBeanFactory beanFactory = getBeanFactory();
    if (logger.isDebugEnabled()) {
        logger.debug("Bean factory for " + getDisplayName() + ": " + beanFactory);
    }
    return beanFactory;
}
```

核心方法在`refreshBeanFactory()`, 它完成了创建BeanFactory和加载BeanDefinition
```java
// AbstractRefreshableApplicationContext#refreshBeanFactory
protected final void refreshBeanFactory() throws BeansException {
    if (hasBeanFactory()) {
        destroyBeans();
        closeBeanFactory();
    }
    try {
        DefaultListableBeanFactory beanFactory = createBeanFactory();
        beanFactory.setSerializationId(getId());
        customizeBeanFactory(beanFactory);
        loadBeanDefinitions(beanFactory);
        synchronized (this.beanFactoryMonitor) {
            this.beanFactory = beanFactory;
        }
    }
    catch (IOException ex) {
        throw new ApplicationContextException("I/O error parsing bean definition source for " + getDisplayName(), ex);
    }
}
```
大概的逻辑如下:

1. 如果当前容器已经存在一个BeanFactory, 则将其销毁
2. 通过`createBeanFactory()`创建一个BeanFactory实例, 具体创建的是`DefaultListableBeanFactory`
3. 自定义BeanFactory, spring的默认实现做了两件事: 是否允许覆盖同名称的不同定义的对象; 是否允许bean之间存在循环依赖. 
4. 加载BeanDefinition
5. 将BeanFactory赋值给当前ApplicationContext

简单说下第3步, 先看方法实现
```java
protected void customizeBeanFactory(DefaultListableBeanFactory beanFactory) {
    // 如果属性allowBeanDefinitionOverriding不为空，设置给beanFactory对象相应属性
    // 此属性的含义：是否允许覆盖同名称的不同定义的对象
    if (this.allowBeanDefinitionOverriding != null) {
        beanFactory.setAllowBeanDefinitionOverriding(this.allowBeanDefinitionOverriding);
    }
    // 如果属性allowCircularReferences不为空，设置给beanFactory对象相应属性
    // 此属性的含义：是否允许bean之间存在循环依赖
    if (this.allowCircularReferences != null) {
        beanFactory.setAllowCircularReferences(this.allowCircularReferences);
    }
}
```
Spring默认是没有对这两个属性进行设置的, 所以其实两个`if`判断都没有进, 那这两个属性有什么意义呢? 还是那句话, 扩展性, 子类可以通过覆盖`customizeBeanFactory()`修改这两个属性的配置.

下面说下第5步, 先看实现
```java
protected void loadBeanDefinitions(DefaultListableBeanFactory beanFactory) throws BeansException, IOException {
    // 创建XmlBeanDefinitionReader并交给BeanFactory
    XmlBeanDefinitionReader beanDefinitionReader = new XmlBeanDefinitionReader(beanFactory);

    // 为beanDefinitionReader配置资源加载环境
    beanDefinitionReader.setEnvironment(getEnvironment());
    beanDefinitionReader.setResourceLoader(this);
    beanDefinitionReader.setEntityResolver(new ResourceEntityResolver(this));

    // 配置beanDefinitionReader内部的验证方式, 用于校验xml文件合法性(XSD/DTD)
    initBeanDefinitionReader(beanDefinitionReader);
    // 解析xml文件, 生成并注册BeanDefinition
    loadBeanDefinitions(beanDefinitionReader);
}

// 生成并注册BeanDefinition方法实现
protected void loadBeanDefinitions(XmlBeanDefinitionReader reader) throws BeansException, IOException {
    Resource[] configResources = getConfigResources();
    if (configResources != null) {
        reader.loadBeanDefinitions(configResources);
    }
    String[] configLocations = getConfigLocations();
    if (configLocations != null) {
        reader.loadBeanDefinitions(configLocations);
    }
}
```

加载BeanDefinition时, 先通过`AbstractBeanDefinitionReader`的`loadBeanDefinitions`方法读取配置文件, 将其转换成`Resource`资源对象. 接着使用`XmlBeanDefinitionReader`的`doLoadBeanDefinitions`方法解析xml, 生成对应的`Document`对象. 然后遍历`Document`对象中的`Element`, 委托给`BeanDefinitionParserDelegate`解析, 主要包括: 

1. 读取id/name/alias
2. 校验name/alias的唯一性
3. 读取class/parent属性 
4. 调用`BeanDefinitionReaderUtils.createBeanDefinition`生成`GenericBeanDefinition`实例
5. 解析并设置`singleton`/`lazy-init`/`autowire`/`depends-on`/`primary`/`init-method`/`destroy-method`/`Factory-bean`/`factory-method`等属性值
6. 解析并设置构造函数参数配置
7. 解析并设置实例属性(`property`)设置
8. 创建并返回`BeanDefinitionHolder`, 将刚刚创建的`BeanDefinition`进行包装(`BeanDefinition`+`beanName`+`aliases`)

最后使用`BeanDefinitionReaderUtils`调用`DefaultListableBeanFactory`的`registerBeanDefinition`方法, 使用`beanName`和`alias`将`BeanDefinition`进行注册

## prepareBeanFacotry(beanFactory)

- 配置`classloader`

- 配置SPEL表达式解析器(`#{...}`)`StandardBeanExpressionResolver`

- 配置一个属性编辑器`ResourceEditorRegistrar`, 该类实现`PropertyEditor`, 此为java标准, 主要服务GUI, 实现String与Object互转.与之搭配的有`TypeConverter`接口进行统一封装, 3.0之后可能交由`ConversionService`处理

  > 从spring3.0开始, SPring重新设计了一套类型转换接口, 核心接口有: `Converter` `ConverterFactory` 和 `GenericConverter`. 在此基础上定义了`ConversionService`整合他们三个, 统一话接口操作, 可以称其为**门面接口**. 除此之外还有`Formatter<T>`接口能将Object转换成类型T也可以实现类型转换, 但它主要侧重与格式化, 一般用于时间/日期, 数字, 货币等场景.

- 添加`ApplicationContextAware`接口后置处理器

- 配置几个忽略自动装配的`Aware`接口(`EnvironmentAware` `ResourceLoaderAware` `ApplicationEventPublisherAware` `MessageSourceAware` `ApplicationContextAware`)

- 配置几个自动装配的特殊类(`BeanFactory` `ResourceLoader` `ApplicationEventPublisher` `ApplicationContext`)

- 配置一个Bean后置处理器, 用来检测实现了`ApplicationListener`接口的类

- 注册单例(`Environment` `systemProperties` `systemEnvironment`)

## postProcessBeanFactory(ConfigurableListableBeanFactory)

该方法在`AbstractApplicationContext`中是空方法, 子类中`AbstractRefreshableWebApplicationContext`重写该方法:

- 添加了`ServletContextAwareProcessor` 
- 配置忽略自动注入接口`ServletContextAware`/`ServletConfigAware`
- 注册单例对象`servletContext`
- 注册环境对象属性`servletContext`和`servletConfig`

## invokeBeanFactoryPostProcessors(ConfigurableListableBeanFactory)

调用`BeanFactoryPostProcessor`

暂时为空

## registerBeanPostProcessors(ConfigurableListableBeanFactory)

从`BeanFactory`中获取`BeanPostProcessor`列表并调用

暂时为空

## initMessageSource

消息提示内容国际化

## initApplicationEventMulticaster

初始化应用事件广播器`SimpleApplicationEventMulticaster`, 并作为单例注册到`BeanFactory`

## onRefresh

空, 留个容器子类扩展

## registerListeners

注册`ApplicationListener`实例

## finishBeanFactoryInitialization(ConfigurableListableBeanFactory)

实例化剩余单例对象





参考资料:
[【死磕 Spring】—– 深入分析 ApplicationContext 的 refresh()](http://cmsblogs.com/?p=4043)
[Spring源码分析（二十）准备环境](https://www.cnblogs.com/warehouse/p/9384735.html)
[Spring源码分析（二十一）加载BeanFactory](https://www.cnblogs.com/warehouse/p/9385110.html)

