---
title: Spring源码 - IOC(2) - ApplicationContext
tags:
- 源码
- IOC
- ApplicationContext
categories:
- Java
- Spring
---
> spring源码版本: 4.3.1-RELEASE

接口继承结构
![hierarchy](extends.png)

实现类继承结构
![hierarchy](impls.png)
比较常用的是如下三个类:
* `XmlWebApplicationContext` : 当`web.xml`中没有指定`contextClass`时, 使用该容器, 即这个是默认容器, 该容器时使用xml格式配置文件
* `AnnotationConfigWebApplicationContext` : 基于注解的 spring 容器, 可以配合`WebApplicationInitializer`和`AbstractAnnotationConfigDispatcherServletInitializer`对容器进行配置
* `AnnotationConfigServletWebServerApplicationContext` :

## 继承结构

- **功能丰富** ：支持高亮代码块、插入 *LaTex* 公式，工作学习好帮手
- **得心应手** ：支持插入图片，无论是本地上传/图片URL/拖放图片/直接截图粘贴，随心所欲
- **深度整合** ：支持选择笔记本和添加标签，支持从印象笔记跳转编辑，轻松管理

