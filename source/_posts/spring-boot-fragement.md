---
title: Spring Boot 零碎
tags:
  - springboot
  - java
categories:
  - java
  - framework
date: 2020-04-27 20:06:52
---

## spring boot 配置文件加载顺序

> https://docs.spring.io/spring-boot/docs/current/reference/htmlsingle/#boot-features-external-config-application-property-files

spring boot默认会按照以下顺序, 在下面的文件夹中查找配置文件:

1. **启动目录**下的`config`目录
2. **启动目录**
3. classpath下的`config`目录

4. classpath根目录下

- **启动目录**指的是你在哪个目录下启动的应用, 举例来说: 应用包`user-center-1.0.0.jar`在`/home/test/app/`下

  1. 如果在`/home/test/app/`下执行`java -jar user-center-1.0.0.jar`, 那么**启动目录**就是`/home/test/app/`

  2. 如果在`/home/test/`下执行`java -jar app/user-center-1.0.0.jar`, 那么**启动目录**就是`/home/test/`

- **classpath**可以简单理解为应用的`resource`目录

spring可以通过命令行参数**`spring.config.location`**自定义配置文件加载顺序, 如: `java -Dspring.config.location=classpath:/properties/,file:./properties/ -jar user-center-1.0.0.jar`, **spring会完全按照这个配置进行查找**, 配置文件加载顺序会变成下面这样:

1. `file:./properties/`
2. `classpath:properties/`

另外, spring还可以通过命令行参数**`spring.config.additional-location`**指定额外的配置文件路径, 如: `java -Dspring.config.additional-location=classpath:/custom-config/,file:./custom-config/ -jar user-center-1.0.0.jar`, 那么配置文件加载顺序就变成了:

1. `file:./custom-config/`
2. `classpath:custom-config/`
3. `file:./config/`
4. `file:./`
5. `classpath:/config/`
6. `classpath:/`

需要注意的是, 如果一个配置项在多个文件都定义了, 则**先定义的优先级更高**, 所以应用`resource`下的配置文件优先级最高

## profile配置文件加载

> https://docs.spring.io/spring-boot/docs/current/reference/htmlsingle/#boot-features-external-config-profile-specific-properties

spring中可以使用` application-{profile}.properties`配置作为`application.properties`配置的补充, 如果没有配置要激活的profile, 则spring会加载` application-default.properties`配置文件. 

**指定` application-{profile}.properties`的优先级高于`application.properties`**, 而且如果激活了多个profile, 则后面的profile会覆盖前者, 这条规则比上面文件目录规则优先级更高, 比如`java -jar -Dspring.profiles.active=prod user-center-1.0.0jar`启动应用. 结合目录加载顺序, 一个配置项的优先级如下:



1. `file:./config/spring-prod.properties`
2. `file:./spring-prod.properties`
3. `classpath:/config/spring-prod.properties`
4. `classpath:spring-prod.properties`
5. `file:./config/spring.properties`
6. `file:./spring.properties`
7. `classpath:/config/spring.properties`
8. `classpath:spring.properties`

**profile优先级最高, 其次先外部再内部, 然后config目录, 最后是默认配置**

## 配置参数加载顺序

> https://docs.spring.io/spring-boot/docs/current/reference/htmlsingle/#boot-features-external-config

spring中通过`${xxx}`加载变量时, 会从多个位置查找这个配置项, 具体顺序如下(不完整, 只保留运行时需要关心的):

1. 命令行
2. `ServletConfig`初始化参数
3. `ServletContext`初始化参数
4. `System.getProperties()`
5. 操作系统环境变量
6. 外部目录下的`application-{profile}.properties`文件
7. `classpath`下的`application-{profile}.properties`文件
8. 外部目录的`application.properties`
9. `classpath`下的`application.properties`文件
10. 应用内使用`@PropertySource`的配置类(`@Configuration`)
