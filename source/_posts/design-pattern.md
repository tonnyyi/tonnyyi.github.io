---
title: 设计模式
tags:
  - design
categories:
  - design
date: 2020-06-30 20:20:27
---



## Builder与Factory模式区别

- Builder模式专注与构建一个复杂的对象. Factory则关注在对象家族的创建
- 通常, 开始会使用Factory方法创建对象, 但随着对象的创建过程变得复杂, 会转为使用抽象工厂, 原型或者Builder模式, 以满足程序的灵活性
- 有时候创建型模式之间是互补的, 你调我, 我调你. 

[stackoverflow](https://stackoverflow.com/a/757773/3351286)