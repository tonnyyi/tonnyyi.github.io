---
title: Spring源码 - IOC(1) - BeanFactory
tags:
- 源码
- IOC
- BeanFactory
categories:
- Java
- Spring
---

> spring源码版本: 4.3.1-RELEASE

![hierarchy](extends.png)

## spring 

bean的id属性与name属性
在3.1之前`id`属性的类型是`xsd:ID`,之后则改为`xsd:string`, 如果我们希望一个bean有多个标示符,在3.1之前只能在name里通过`,;空格`分隔方式实现.如果想在id属性上这么配是不符合xml规范的.3.1之后就没这个问题了,在id上也可以配置多个标示符.如果id不为空, 则第一个标识符会被解析为bean的`id`属性,其余的被赋值给bean的`alias`属性; 如果id为空而name不为空,则规则同上; 如果都为空则默认类的全限定名作为标识符. 标识符必须唯一.
