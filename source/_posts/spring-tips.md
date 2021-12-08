---
title: Spring零碎知识点
tags:
  - Spring
categories:
  - SourceCode
  - Spring
date: 2018-03-01 12:17:44
---

- Environment对象初始化时间点
容器初始化时, 设置配置文件路径AbstractRefreshableConfigApplicationContext#setConfigLocations, 内部调用了resolvePath(), 该方法首先通过AbstractApplicationContext#getEnvironment()获取ConfigurableEnvironment对象, 因为是第一次获取, 所以会首先createEnvironment()
<!-- more -->

### Spring扩展点
- `BeanFactoryPostProcessor`
- `BeanDefinitionRegistryPostProcessor`
- `BeanPostProcessor`
- `InitializingBean`
- `DisposableBean`
- `Aware`子类, 如:`ApplicationContextAware`, `BeanFactoryAware`, `ResourceLoaderAware`, `ServletContextAware`, `BeanNameAware`, `EnvironmentAware`等
- `ApplicationListener`

获取本机IP地址: `org.springframework.cloud.commons.util.InetUtils`

