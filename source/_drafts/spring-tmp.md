---
title: spring-tmp
tags:
  - null
categories:
  - null
date: 2018-12-28 15:21:50
---

# spring mvc
`DefaultHandlerExceptionResolver`: 默认的异常处理类, 处理的异常包括: 找不到对应的处理方法(405); 响应类型不支持(415); 响应类型不接受(406); PathVariable缺失(500); RequestParameter缺失(400); 

`RequestMapping`负责找到请求对应的处理者, `HandlerAdapter`负责在调用实际处理者之前, 处理参数转换

