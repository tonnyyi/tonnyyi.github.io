---
title: spring-tmp
tags:
  - null
categories:
  - null
date: 2018-12-28 15:21:50
---

容器启动过程
1. `ConfigurableWebApplicationContext.setConfigLocation()` : 解析配置的资源文件路径上的占位符
    一般在`web.xml`中使用形如:`classpath:spring-context.xml`指定spring的配置文件位置, 但其实该文件路径中可以包含占位符, 比如: `classpath:${configDir}/spring-context.xml`, 然后通过系统属性(JVM参数传递)设置该占位符的实际内容. 而spring会在设置配置文件路径前会解析占位符, 得到完整的配置文件路径. 解析方式是创建`StandardServletEnvironment` 或 `StandardEnvironment`, 前者是Web容器中使用, 后者是父容器使用. 这两个类都继承了`AbstractEnvironment`, 该父类中有一个`resolvePlaceholders`方法用来解析占位符. 但`resolvePlaceholders`也是将解析工作委托给`PropertySourcesPropertyResolver.resolvePlaceholders()`方法. `resolvePlaceholders`的定义以及实现是在`PropertySourcesPropertyResolver`的父类`AbstractPropertyResolver`, 该父类中有几个比较重要的属性: `placeholderPrefix`: 标识占位符前缀, 即`${`; `placeholderSuffix`: 标识占位符后缀, 即`}`; `conversionService`: 类型转换类, 用于将属性在不同类型进行转换, 比如: string -> array, array -> collection. 解析过程就是一个自我递归的过程, 因为`${configDir}`解析出来的值中可能也包含占位符, 所以需要递归解析. 
    `conversionService`还有一个用处就是在实例化bean时(`AbstractAutowireCapableBeanFactory.populateBean()`), 如果bean的属性是通过`@Value`注入的, 还需要解析其中的占位符并进行类型转后, 赋值给bean的属性. 但这是通过`PropertySourcesPlaceholderConfigurer`完成的. `conversionService`中保存了大量的`Converters`实现类, 每个负责一种转换策略, 比如:`StringToCollectionConverter`实现类负责将String 转成 Collection
2. `ConfigurableApplicationContext.refresh()`

# spring mvc
`DefaultHandlerExceptionResolver`: 默认的异常处理类, 处理的异常包括: 找不到对应的处理方法(405); 响应类型不支持(415); 响应类型不接受(406); PathVariable缺失(500); RequestParameter缺失(400); 

`RequestMapping`负责找到请求对应的处理者, `HandlerAdapter`负责在调用实际处理者之前, 处理参数转换

