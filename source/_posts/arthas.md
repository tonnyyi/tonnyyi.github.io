---
title: arthas
tags:
  - arthas
  - java
categories:
  - java
date: 2021-06-23 09:18:37
---



## 常用流程

1. 搜索类 `SC`

   ```bash
   sc *.UserController
   sc org.springframework.*
   
   # 打印详细信息 -d：详情详细  -f：打印属性
   sc -d -f com.test.UserController
   ```

2. 查看方法内部调用栈以及各节点上的耗时 `trace`  

   ```bash
   trace com.test.UserController login
   
   # 统计次数为1；包含JDK内的函数；耗时超过10ms
   trace --skipJDKMethod false com.test.userController login -n 1 '#cost > 10'
   ```

3. 查看方法被调用路径 `stack`

   ```bash
   stack com.test.UserService login
   
   # 根据条件过滤 统计1次
   stack com.test.UserService login 'params[0]=="admin"' -n 1
   ```

4. 查看方法出入参 `watch`

   ```bash
   # 方法调用前后入参和返回值 -x:属性展开深度
   watch demo.MathGame primeFactors "{params,returnObj}" -x 3 -b -f
   
   # 根据耗时过滤
   watch demo.MathGame primeFactors '{params, returnObj}' '#cost>200' -x 2
   # 观察异常信息
   watch demo.MathGame primeFactors "{params[0],throwExp}" -e -x 2
   ```

5. 保存、重播方法调用记录  `tt`

   ```bash
   tt -t demo.MathGame primeFactors
   
   # 根据参数过滤
   tt -t *Test print params[0].name=="admin"
   # 查看记录
   tt -l
   # 查看详细调用信息
   tt -i 1003
   # 观察表达式
   tt -w 'target.failCount' -x 1 -i 1003
   
   # 请求重放
   tt -i 1003 -p
   ```

   

### spring应用下调用Bean方法

1. 拦截请求

   ```bash
   tt -t org.springframework.web.servlet.mvc.method.annotation.RequestMappingHandlerAdapter invokeHandlerMethod
   ```

2. 获取spring context

   ```bash
   tt -i 1000 -w 'target.getApplicationContext()'
   ```

3. 调用bean方法

   ```bash
   tt -i 1000 -w 'target.getApplicationContext().getBean("helloWorldService").getHelloMessage()'
   ```

   

## 命令详细手册

Arthas的一些特殊用法 https://github.com/alibaba/arthas/issues/71
