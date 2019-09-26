---
title: spring启动过程分析
tags:
  - spring ioc
  - spring
categories:
  - spring
  - source code
  - java
date: 2019-08-26 18:18:05
---

> spring 版本 4.3.1

入口`org.springframework.web.context.ContextLoader#initWebApplicationContext`, 这里创建应用上下文, 并绑定当前 web 环境中.
精简后的方法体如下:
```java
if (this.context == null) {
    // 创建应用上下文
	this.context = createWebApplicationContext(servletContext);
}
if (this.context instanceof ConfigurableWebApplicationContext) {
	ConfigurableWebApplicationContext cwac = (ConfigurableWebApplicationContext) this.context;
	if (!cwac.isActive()) {
    	// 设置父容器
		if (cwac.getParent() == null) {
			ApplicationContext parent = loadParentContext(servletContext);
			cwac.setParent(parent);
		}
		configureAndRefreshWebApplicationContext(cwac, servletContext);
	}
}
servletContext.setAttribute(WebApplicationContext.ROOT_WEB_APPLICATION_CONTEXT_ATTRIBUTE, this.context);

return this.context;
```

1. `createWebApplicationContext`方法会先确定要实例化的上下文类名. 读取顺序是: `contextClass`参数 -> `spring-web` jar包中的`ContextLoader.properties`文件, 后者配置的默认值是`XmlWebApplicationContext`.
2. `configureAndRefreshWebApplicationContext`方法中最终调用`org.springframework.context.support.AbstractApplicationContext#refresh`进行容器的初始化.

## `refresh` 方法
完整的方法体如下:
```java
@Override
public void refresh() throws BeansException, IllegalStateException {
	synchronized (this.startupShutdownMonitor) {
		// 设置启动时间, 运行状态标识, 初始化Environment(System.props/env/web属性)
		prepareRefresh();

		// 销毁已有BeanFactory, 创建ConfigurableListableBeanFactory, 加载 BeanDefinition
		ConfigurableListableBeanFactory beanFactory = obtainFreshBeanFactory();

		// 创建ResourceEditorRegistrar, SpelExpressionParser
		// 设置要忽略的自动注入接口, 配置当前类作为一些自动注入接口的实现类
		// 注册单例的environment, systemProperties, systemEnvironment
		prepareBeanFactory(beanFactory);

		try {
			// Allows post-processing of the bean factory in context subclasses.
			postProcessBeanFactory(beanFactory);

			// Invoke factory processors registered as beans in the context.
			invokeBeanFactoryPostProcessors(beanFactory);

			// Register bean processors that intercept bean creation.
			registerBeanPostProcessors(beanFactory);

			// Initialize message source for this context.
			initMessageSource();

			// Initialize event multicaster for this context.
			initApplicationEventMulticaster();

			// Initialize other special beans in specific context subclasses.
			onRefresh();

			// Check for listener beans and register them.
			registerListeners();

			// Instantiate all remaining (non-lazy-init) singletons.
			finishBeanFactoryInitialization(beanFactory);

			// Last step: publish corresponding event.
			finishRefresh();
		}

		catch (BeansException ex) {
			// Destroy already created singletons to avoid dangling resources.
			destroyBeans();

			// Reset 'active' flag.
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

## 加载资源

## 解析资源

## 注册 BeanDefinition