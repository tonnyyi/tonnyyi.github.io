---
title: Spring启动过程分析
tags:
  - Spring
categories:
  - SourceCode
  - Spring
date: 2018-03-01 14:19:50
---

### AbstractApplicationContext#refresh
```java
public void refresh() throws BeansException, IllegalStateException {
    synchronized (this.startupShutdownMonitor) {
        prepareRefresh();
    
        ConfigurableListableBeanFactory beanFactory = obtainFreshBeanFactory();
        prepareBeanFactory(beanFactory);
        
        try {
            postProcessBeanFactory(beanFactory);
            invokeBeanFactoryPostProcessors(beanFactory);
            registerBeanPostProcessors(beanFactory);
            initMessageSource();
            initApplicationEventMulticaster();
            onRefresh();
            registerListeners();
            finishBeanFactoryInitialization(beanFactory);
            finishRefresh();
        }
        catch (BeansException ex) {
            destroyBeans();
            cancelRefresh(ex);
            throw ex;
        }
    }
}
```
<!-- more -->
### 1. prepareRefresh()
 - 设置启动时间`startupDate`为当前毫秒值 
 - 启动标志`active`为`true`
 - 执行子类重载方法:`AbstractRefreshableWebApplicationContext#initPropertySources`, 将当前应用的`servletContext`和`servletConfig`添加到属性源
 - 执行	`AbstractEnvironment#validateRequiredProperties`, 确保必要属性非空

### 2. obtainFreshBeanFactory()
主要作用: 创建`BeanFactory`, 加载`BeanDefinition`.
内部主要调用`AbstractRefreshableApplicationContext#refreshBeanFactory`, 定义如下

```java
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
`createBeanFactory()`通过`new DefaultListableBeanFactory`创建了Bean工厂, 也就是Spring默认的工厂类是`DefaultListableBeanFactory`
`customizeBeanFactory`实际是调用的子类重载的方法`AbstractRefreshableApplicationContext#customizeBeanFactory`, 该方法内是配置Bean工厂的`allowBeanDefinitionOverriding`和`allowCircularReferences`属性
`loadBeanDefinitions()`方法用来加载`BeanDefinition`, 实际调用的是子类重载的方法`AbstractXmlApplicationContext#loadBeanDefinitions(DefaultListableBeanFactory)`


### 3. prepareBeanFactory(beanFactory)
主要作用: 配置Bean工厂, 将内部的一些类ignore或注册为可依赖的, 主要调用`ConfigurableListableBeanFactory#ignoreDependencyInterface` 和 `ConfigurableListableBeanFactory#registerResolvableDependency`.

### 4. postProcessBeanFactory(beanFactory)
实际调用子类重载方法`AbstractRefreshableWebApplicationContext#postProcessBeanFactory`. 该方法主要: 
- 向Bean工厂添加`ServletContextAwareProcessor`Bean后置处理器
- 配置Bean工厂, 忽略`ServletConfigAware`和`ServletContextAware`接口依赖

### 5. invokeBeanFactoryPostProcessors(beanFactory)
调用所有的`BeanFactoryPostProcessor`, myBatis的`MapperScannerConfigurer`就是在此刻向Bean工厂注入了Mapper接口的`BeanDefinition`, 这个`BeanDefinition`都是工厂类, 最终调用`MapperFactoryBean`生成实现类.

### 6. registerBeanPostProcessors(beanFactory)
将Bean工厂里的`BeanPostProcessor`类型的Bean先实例化, 然后通过`ConfigurableBeanFactory#addBeanPostProcessor`将这些Bean存到`beanPostProcessors`属性里

### 7. initMessageSource()
初始化`MessageSource`, 该类主要是为系统提供国际化支持, 可以输出不同语言的提示信息

### 8. initApplicationEventMulticaster()
初始化Spring事件多播器, 默认创建一个`SimpleApplicationEventMulticaster`实例

### 9. onRefresh()
给`AbstractApplicationContext`的子类一个参与refresh的机会, 其中`AbstractRefreshableWebApplicationContext#onRefresh`初始化了主题资源

### 10. registerListeners()
将保存在容器中的事件监听器`ApplicationListener`对象注册到Spring事件多播器中

### 11. finishBeanFactoryInitialization(beanFactory)
实例化容器中剩余的单例Bean

### 12. finishRefresh()
初始化生命周期处理器, 默认实例化`DefaultLifecycleProcessor`, 并发布容器刷新时间`ContextRefreshedEvent`

### 13. destroyBeans()
清空容器中保存bean的相关属性

### 14. cancelRefresh(ex)
设置启动标志为false
