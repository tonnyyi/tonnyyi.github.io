---
title: uml
tags:
  - uml
categories:
  - uml
date: 2020-09-10 10:27:14
---



## 依赖

{% plantuml %}
Person ..> Car
{% endplantuml %}

java中体现为局部变量, 方法参数, 静态方法调用

```java
class Person {
    void buy(Car car){}
}
```



## 关联

{% plantuml %}

师傅 "1" --> "*" 徒弟

{% endplantuml %}



java中通过**成员变量**来体现

```java
class 师傅 {
    private List<徒弟> tdList;
}
```



## 聚合

{% plantuml %}

部门 "1" o-- "*" 员工

{% endplantuml %}

属于是关联的特殊情况, 体现部分-整体关系, 是一种弱拥有关系; **整体和部分可以有不一样的生命周期**



## 组合

{% plantuml %}

人 "1" *-- "4" 肢

{% endplantuml %}

合成关系是关联关系的一种，是比聚合关系还要强的关系; **整体的对象负责部分的对象的生命周期**


[UML类图符号 各种关系说明以及举例](https://www.cnblogs.com/duanxz/archive/2012/06/13/2547801.html)