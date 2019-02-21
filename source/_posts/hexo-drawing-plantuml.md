---
title: 在hexo中画图-PlantUml
tags:
  - hexo
  - uml
categories:
  - hexo
date: 2019-02-21 10:13:32
---
[PlantUml文档](http://plantuml.com/zh/)

# PlantUml
PlantUML 是一个画图脚本语言，用它可以快速地画出：时序图, 流程图, 用例图, 状态图, 组件图. 
## 安装
在博客目录下执行如下目录, 安装PlantUml插件
```bash
npm install hexo-tag-plantuml --save
```

## 使用
### 类图
#### 包
你可以通过关键词 package 声明包，同时可选的来声明对应的背景色（通过使用html色彩代码或名称）, 包也可嵌套
```
{% plantuml %}
package "Classic Collections" #DDDDDD {
  Object <|-- ArrayList
}

package net.sourceforge.plantuml {
  Object <|-- Demo1
  Demo1 *- Demo2
  
  package net.sourceforge.plantuml.demo {
    Demo2 <-- Demo3
  }
}
{% endplantuml %}
```

{% plantuml %}
package "Classic Collections" #DDDDDD {
  Object <|-- ArrayList
}

package net.sourceforge.plantuml {
  Object <|-- Demo1
  Demo1 *- Demo2
  
  package net.sourceforge.plantuml.demo {
    Demo2 <-- Demo3
  }
}
{% endplantuml %}

包样式
```
{% plantuml %}
scale 750 width
package foo1 <<Node>> {
  class Class1
}

package foo2 <<Rectangle>> {
  class Class2
}

package foo3 <<Folder>> {
  class Class3
}

package foo4 <<Frame>> {
  class Class4
}

package foo5 <<Cloud>> {
  class Class5
}

package foo6 <<Database>> {
  class Class6
}
{% endplantuml %}
```

{% plantuml %}
scale 750 width
package foo1 <<Node>> {
  class Class1
}

package foo2 <<Rectangle>> {
  class Class2
}

package foo3 <<Folder>> {
  class Class3
}

package foo4 <<Frame>> {
  class Class4
}

package foo5 <<Cloud>> {
  class Class5
}

package foo6 <<Database>> {
  class Class6
}
{% endplantuml %}

#### 抽象类与接口
用关键字abstract或abstract class来定义抽象类。抽象类用斜体显示。 也可以使用interface, annotation 和 enum关键字。
```
{% plantuml %}
abstract class AbstractList
abstract AbstractCollection
interface List
interface Collection

List <|-- AbstractList
Collection <|-- AbstractCollection

Collection <|- List
AbstractCollection <|- AbstractList
AbstractList <|-- ArrayList

class ArrayList {
  Object[] elementData
  size()
}

enum TimeUnit {
  DAYS
  HOURS
  MINUTES
}

annotation SuppressWarnings
{% endplantuml %}
```

{% plantuml %}
abstract class AbstractList
abstract AbstractCollection
interface List
interface Collection

List <|-- AbstractList
Collection <|-- AbstractCollection

Collection <|- List
AbstractCollection <|- AbstractList
AbstractList <|-- ArrayList

class ArrayList {
  Object[] elementData
  size()
}

enum TimeUnit {
  DAYS
  HOURS
  MINUTES
}

annotation SuppressWarnings
{% endplantuml %}
#### 抽象与静态
通过修饰符{static}或者{abstract}，可以定义静态或者抽象的方法或者属性
```
{% plantuml %}
class Dummy {
  {static} String id
  {abstract} void methods()
}
{% endplantuml %}
```

{% plantuml %}
class Dummy {
  {static} String id
  {abstract} void methods()
}
{% endplantuml %}

#### 关系
```
{% plantuml %}
Class01 <|-- Class02
Class03 *-- Class04
Class05 o-- Class06
Class07 .. Class08
Class09 -- Class10

Class11 <|.. Class12
Class13 --> Class14
Class15 ..> Class16
Class17 ..|> Class18
Class19 <--* Class20

Class21 #-- Class22
Class23 x-- Class24
Class25 }-- Class26
Class27 +-- Class28
Class29 ^-- Class30
{% endplantuml %}
```

{% plantuml %}
Class01 <|-- Class02
Class03 "1" *-- "many" Class04
Class05 o-- Class06 : aggregation
Class07 .. Class08
Class09 -- "1" Class10
{% endplantuml %}

{% plantuml %}
Class11 <|.. Class12
Class13 --> Class14
Class15 ..> Class16
Class17 ..|> Class18
Class19 <--* Class20
{% endplantuml %}

{% plantuml %}
Class21 #-- Class22
Class23 x-- Class24
Class25 }-- Class26
Class27 +-- Class28
Class29 ^-- Class30
{% endplantuml %}

类之间默认采用两个破折号 -- 显示出垂直 方向的线. 要得到水平方向的可以像这样使用单破折号 (或者点):
```
{% plantuml %}
Room o- Student
Room *-- Chair

School -o Teacher
Course --* Teacher
{% endplantuml %}
```

{% plantuml %}
Room o- Student
Room *-- Chair

School -o Teacher
Course --* Teacher
{% endplantuml %}

也可通过在箭头内部使用关键字， 例如left, right, up 或者 down，来改变方向
```
{% plantuml %}
foo -left-> dummyLeft 
foo -right-> dummyRight 
foo -up-> dummyUp 
foo -down-> dummyDown
{% endplantuml %}
```

{% plantuml %}
foo -left-> dummyLeft 
foo -right-> dummyRight 
foo -up-> dummyUp 
foo -down-> dummyDown
{% endplantuml %}

你可以在定义了两个类之间的关系后定义一个 关系类 association class 例如:
```
{% plantuml %}
class Student {
  Name
}
Student "0..*" - "1..*" Course
(Student, Course) .. Enrollment

class Enrollment {
  drop()
  cancel()
}
{% endplantuml %}
```

{% plantuml %}
class Student {
  Name
}
Student "0..*" - "1..*" Course
(Student, Course) .. Enrollment

class Enrollment {
  drop()
  cancel()
}
{% endplantuml %}

#### 类方法
```
{% plantuml %}
Dummy <|-- Flight

class Dummy {
  String data
  void methods()
}

class Flight {
   flightNumber : Integer
   departureTime : Date
}
{% endplantuml %}

```
{% plantuml %}
Dummy <|-- Flight

class Dummy {
  String data
  void methods()
}

class Flight {
   flightNumber : Integer
   departureTime : Date
}
{% endplantuml %}

#### 方法可见性
- \-			private
- \#			protected
- ~			    package private
- \+			public

```
{% plantuml %}
class Dummy {
 -field1
 #field2
 +field3
 ~method1()
 #method2()
 +method3()
}
{% endplantuml %}
```

{% plantuml %}
class Dummy {
 -field1
 #field2
 +field3
 ~method1()
 #method2()
 +method3()
}
{% endplantuml %}

可以采用以下命令停用这些特性 `skinparam classAttributeIconSize 0`, 这样方法可见性修饰符就不会变成icon
```
{% plantuml %}
skinparam classAttributeIconSize 0
class Dummy {
 -field1
 #field2
 +field3
 ~method1()
 #method2()
 +method3()
}
{% endplantuml %}
```

{% plantuml %}
skinparam classAttributeIconSize 0
class Dummy {
 -field1
 #field2
 +field3
 ~method1()
 #method2()
 +method3()
}
{% endplantuml %}

#### 方法属性分组
```
{% plantuml %}
class Foo1 {
  You can use
  several lines
  ..
  as you want
  and group
  ==
  things together.
  __
  You can have as many groups
  as you want
  --
  End of class
}

class User {
  .. Simple Getter ..
  + getName()
  + getAddress()
  .. Some setter ..
  + setName()
  __ private data __
  int age
  -- encrypted --
  String password
}
{% endplantuml %}
```

{% plantuml %}
class Foo1 {
  You can use
  several lines
  ..
  as you want
  and group
  ==
  things together.
  __
  You can have as many groups
  as you want
  --
  End of class
}

class User {
  .. Simple Getter ..
  + getName()
  + getAddress()
  .. Some setter ..
  + setName()
  __ private data __
  int age
  -- encrypted --
  String password
}
{% endplantuml %}

#### 备注
你可以使用`note left of` , `note right of` , `note top of` , `note bottom of`这些关键字来添加备注。
你还可以在类的声明末尾使用`note left`, `note right`, `note top`, `note bottom`来添加备注。
此外，单独用note这个关键字也是可以的，使用 .. 符号可以作出一条连接它与其它对象的虚线。
```
{% plantuml %}
class Object << general >>
Object <|--- ArrayList

note top of Object : In java, every class\nextends this one.

note "This is a floating note" as N1
note "This note is connected\nto several objects." as N2
Object .. N2
N2 .. ArrayList

class Foo
note left: On last defined class
{% endplantuml %}
```

{% plantuml %}
class Object << general >>
Object <|--- ArrayList

note top of Object : In java, every class\nextends this one.

note "This is a floating note" as N1
note "This note is connected\nto several objects." as N2
Object .. N2
N2 .. ArrayList

class Foo
note left: On last defined class
{% endplantuml %}

在定义链接之后，你可以用 note on link 给链接添加注释. 如果想要改变注释相对于标签的位置，你也可以用 note left on link， note right on link， note bottom on link。（对应位置分别在label的左边，右边，下边）
```
{% plantuml %}
class Dummy
Dummy --> Foo : A link
note on link #red: note that is red

Dummy --> Foo2 : Another link
note right on link #green
	this is my note on right link
	and in green
end note
{% endplantuml %}
```

{% plantuml %}
class Dummy
Dummy --> Foo : A link
note on link #red: note that is red

Dummy --> Foo2 : Another link
note right on link #green
	this is my note on right link
	and in green
end note
{% endplantuml %}

## 时序图
你可以用->来绘制参与者之间传递的消息， 而不必显式地声明参与者。
你也可以使用 --> 绘制一个虚线箭头。

另外，你还能用 <- 和 <--，这不影响绘图，但可以提高可读性。 注意：仅适用于时序图，对于其它示意图，规则是不同的。
{% plantuml %}
Alice -> Bob: Authentication Request
Bob --> Alice: Authentication Response

Alice -> Bob: Another authentication Request
Alice <-- Bob: another authentication Response
{% endplantuml %}

### 声明参与者
关键字 participant 用于改变参与者的先后顺序。

你也可以使用其它关键字来声明参与者：
* actor
* boundary
* control
* entity
* database

{% plantuml %}
actor Foo1
boundary Foo2
⌃ Foo3
entity Foo4
database Foo5
collections Foo6
Foo1 -> Foo2 : To boundary
Foo1 -> Foo3 : To ⌃
Foo1 -> Foo4 : To entity
Foo1 -> Foo5 : To database
Foo1 -> Foo6 : To collections
{% endplantuml %}

您可以使用关键字 order自定义顺序来打印参与者

{% plantuml %}
participant Last order 30
participant Middle order 20
participant First order 10
{% endplantuml %}

### 修改箭头样式
箭头样式有以下几种:
* 表示一条丢失的消息：末尾加 x
* 让箭头只有上半部分或者下半部分：将<和>替换成\或者 /
* 细箭头：将箭头标记写两次 (如 >> 或 //)
* 虚线箭头：用 -- 替代 -
* 箭头末尾加圈：->o
* 双向箭头：<->

{% plantuml %}
Bob ->x Alice
Bob -> Alice
Bob ->> Alice
Bob -\ Alice
Bob \\- Alice
Bob //-- Alice

Bob ->o Alice
Bob o\\-- Alice

Bob <-> Alice
Bob <->o Alice
{% endplantuml %}

你可以用以下记号修改箭头的颜色：

{% plantuml %}
Bob -[#red]> Alice : hello
Alice -[#0000FF]->Bob : ok
{% endplantuml %}

### 对消息序列编号
关键字 autonumber 用于自动对消息编号。
{% plantuml %}
autonumber
Bob -> Alice : Authentication Request
Bob <- Alice : Authentication Response
{% endplantuml %}

语句 autonumber start 用于指定编号的初始值，而 autonumber startincrement 可以同时指定编号的初始值和每次增加的值。
{% plantuml %}
autonumber
Bob -> Alice : Authentication Request
Bob <- Alice : Authentication Response

autonumber 15
Bob -> Alice : Another authentication Request
Bob <- Alice : Another authentication Response

autonumber 40 10
Bob -> Alice : Yet another authentication Request
Bob <- Alice : Yet another authentication Response
{% endplantuml %}

你还可以用语句 autonumber stop 和 autonumber resume incrementformat 来表示暂停或继续使用自动编号。
{% plantuml %}
autonumber 10 10 "<b>[000]"
Bob -> Alice : Authentication Request
Bob <- Alice : Authentication Response

autonumber stop
Bob -> Alice : dummy

autonumber resume "<font color=red><b>Message 0  "
Bob -> Alice : Yet another authentication Request
Bob <- Alice : Yet another authentication Response

autonumber stop
Bob -> Alice : dummy

autonumber resume 1 "<font color=blue><b>Message 0  "
Bob -> Alice : Yet another authentication Request
Bob <- Alice : Yet another authentication Response
{% endplantuml %}

### 组合消息
我们可以通过以下关键词将组合消息：
* alt/else
* opt
* loop
* par
* break
* critical
* group, 后面紧跟着消息内容

可以在标头(header)添加需要显示的文字(group除外)。

关键词 end 用来结束分组。

注意，分组可以嵌套使用。
{% plantuml %}
Alice -> Bob: Authentication Request

alt successful case

	Bob -> Alice: Authentication Accepted
	
else some kind of failure

	Bob -> Alice: Authentication Failure
	group My own label
		Alice -> Log : Log attack start
	    loop 1000 times
	        Alice -> Bob: DNS Attack
	    end
		Alice -> Log : Log attack end
	end
	
else Another type of failure

   Bob -> Alice: Please repeat
   
end
{% endplantuml %}

### 添加注释
我们可以通过在消息后面添加 note left 或者 note right 关键词来给消息添加注释。

你也可以通过使用 end note 来添加多行注释
{% plantuml %}
Alice->Bob : hello
note left: this is a first note

Bob->Alice : ok
note right: this is another note

Bob->Bob : I am thinking
note left
	a note
	can also be defined
	on several lines
end note
{% endplantuml %}

可以使用note left of，note right of或note over在节点(participant)的相对位置放置注释。

还可以通过修改背景色来高亮显示注释。

以及使用关键字end note来添加多行注释

{% plantuml %}
participant Alice
participant Bob
note left of Alice #aqua
	This is displayed 
	left of Alice. 
end note
 
note right of Alice: This is displayed right of Alice.

note over Alice: This is displayed over Alice.

note over Alice, Bob #FFAAAA: This is displayed\n over Bob and Alice.

note over Bob, Alice
	This is yet another
	example of
	a long note.
end note
{% endplantuml %}

### 分隔符
你可以通过使用 == 关键词来将你的图表分割多个步骤。
{% plantuml %}
== Initialization ==

Alice -> Bob: Authentication Request
Bob --> Alice: Authentication Response

== Repetition ==

Alice -> Bob: Another authentication Request
Alice <-- Bob: another authentication Response
{% endplantuml %}

### 延迟
你可以使用...来表示延迟，并且还可以给延迟添加注释。
{% plantuml %}
Alice -> Bob: Authentication Request
...
Bob --> Alice: Authentication Response
...5 minutes latter...
Bob --> Alice: Bye !
{% endplantuml %}

### 空间
你可以使用|||来增加空间。

还可以使用数字指定增加的像素的数量。
{% plantuml %}
Alice -> Bob: message 1
Bob --> Alice: ok
|||
Alice -> Bob: message 2
Bob --> Alice: ok
||45||
Alice -> Bob: message 3
Bob --> Alice: ok
{% endplantuml %}

### 生命线的激活与撤销
关键字activate和deactivate用来表示参与者的生命活动。
一旦参与者被激活，它的生命线就会显示出来。
activate和deactivate适用于以上情形。
destroy表示一个参与者的生命线的终结。
{% plantuml %}
participant User

[-> User: DoWork

User -> A: DoWork
activate A #FFBBBB

A -> A: Internal call
activate A #DarkSalmon

A -> B: << createRequest >>
activate B

B --> A: RequestCreated
deactivate B
deactivate A
A -> User: Done

[<- User: Done
deactivate A
{% endplantuml %}

###移除脚注
使用hide footbox关键字移除脚注。
{% plantuml %}
hide footbox
title Footer removed

Alice -> Bob: Authentication Request
Bob --> Alice: Authentication Response

{% endplantuml %}

### 包裹参与者
可以使用box和end box画一个盒子将参与者包裹起来。
还可以在box关键字之后添加标题或者背景颜色。
{% plantuml %}
box "Internal Service" #LightBlue
	participant Bob
	participant Alice
end box
participant Other

Bob -> Alice : hello
Alice -> Other : hello
{% endplantuml %}

##流程图
这里只介绍新语法

{% plantuml %}
start
:Hello world;
:This is on defined on
several **lines**;
end
{% endplantuml %}

###条件语句
在图示中可以使用关键字if，then和else设置分支测试。标注文字则放在括号中。
{% plantuml %}
start

if (condition A) then (yes)
  :gogogo;
elseif (condition B) then (yes)
  :Text 2;
  stop
elseif (condition C) then (yes)
  :Text 3;
else (no)
  :process only
  __sequence__ and __activity__ diagrams;
endif

stop
{% endplantuml %}

###重复
你可以使用关键字repeat和repeatwhile进行重复循环。
{% plantuml %}
start

repeat
  :read data;
  :generate diagrams;
repeat while (more data?)

stop
{% endplantuml %}

###while循环
可以使用关键字while和end while进行while循环。
{% plantuml %}
start

while (data available?)
  :read data;
  :generate diagrams;
endwhile

stop
{% endplantuml %}

###并行处理
你可以使用关键字fork，fork again和end fork表示并行处理。
{% plantuml %}
start

if (multiprocessor?) then (yes)
  fork
	:Treatment 1;
  fork again
	:Treatment 2;
  end fork
else (monoproc)
  :Treatment 1;
  :Treatment 2;
endif
{% endplantuml %}

###注释
{% plantuml %}
start
:foo1;
floating note left: This is a note
:foo2;
note right
  This note is on several
  //lines// and can
  contain <b>HTML</b>
  ====
  * Calling the method ""foo()"" is prohibited
end note
stop
{% endplantuml %}

###箭头
使用->标记，你可以给箭头添加文字或者修改箭头颜色。
同时，你也可以选择点状 (dotted)，条状(dashed)，加粗或者是隐式箭头

{% plantuml %}
:foo1;
-> You can put text on arrows;
if (test) then
  -[#blue]->
  :foo2;
  -[#green,dashed]-> The text can
  also be on several lines
  and **very** long...;
  :foo3;
else
  -[#black,dotted]->
  :foo4;
endif
-[#gray,bold]->
:foo5;
{% endplantuml %}

### 组合
通过定义分区(partition)，你可以把多个活动组合(group)在一起。
{% plantuml %}
start
partition Initialization {
	:read config file;
	:init internal variable;
}
partition Running {
	:wait for user interaction;
	:print information;
}

stop
{% endplantuml %}

###泳道
{% plantuml %}
|Swimlane1|
start
:foo1;
|#AntiqueWhite|Swimlane2|
:foo2;
:foo3;
|Swimlane1|
:foo4;
|Swimlane2|
:foo5;
stop
{% endplantuml %}

###分离
可以使用关键字detach移除箭头。
{% plantuml %}
:start;
 fork
   :foo1;
   :foo2;
 fork again
   :foo3;
   detach
 endfork
 if (foo4) then
   :foo5;
   detach
 endif
 :foo6;
 detach
 :foo7;
 stop
{% endplantuml %}


一个比较完整的例子
{% plantuml %}
start
:ClickServlet.handleRequest();
:new page;
if (Page.onSecurityCheck) then (true)
  :Page.onInit();
  if (isForward?) then (no)
	:Process controls;
	if (continue processing?) then (no)
	  stop
	endif
	
	if (isPost?) then (yes)
	  :Page.onPost();
	else (no)
	  :Page.onGet();
	endif
	:Page.onRender();
  endif
else (false)
endif

if (do redirect?) then (yes)
  :redirect process;
else
  if (do forward?) then (yes)
	:Forward request;
  else (no)
	:Render page template;
  endif
endif

stop
{% endplantuml %}

## 用例图
### 用例
用例用圆括号括起来, 也可以用关键字usecase来定义用例。 还可以用关键字as定义一个别名，这个别名可以在以后定义关系的时候使用。
```
{% plantuml %}
(First usecase)
(Another usecase) as (UC2)  
usecase UC3
usecase (Last\nusecase) as UC4
{% endplantuml %}
```

{% plantuml %}
(First usecase)
(Another usecase) as (UC2)  
usecase UC3
usecase (Last\nusecase) as UC4
{% endplantuml %}

### 角色
角色用两个冒号包裹起来, 也可以用actor关键字来定义角色。 还可以用关键字as来定义一个别名，这个别名可以在以后定义关系的时候使用。
```
{% plantuml %}
:First Actor:
:Another\nactor: as Men2  
actor Men3
actor :Last actor: as Men4
{% endplantuml %}
```

{% plantuml %}
:First Actor:
:Another\nactor: as Men2  
actor Men3
actor :Last actor: as Men4
{% endplantuml %}

### 用例描述
如果想定义跨越多行的用例描述，可以用双引号将其裹起来。还可以使用这些分隔符：--..==__。 并且还可以在分隔符中间放置标题。
```
{% plantuml %}
usecase UC1 as "You can use
several lines to define your usecase.
You can also use separators.
--
Several separators are possible.
==
And you can add titles:
..Conclusion..
This allows large description."
{% endplantuml %}
```

{% plantuml %}
usecase UC1 as "You can use
several lines to define your usecase.
You can also use separators.
--
Several separators are possible.
==
And you can add titles:
..Conclusion..
This allows large description."
{% endplantuml %}

### 基础用例
用箭头-->连接角色和用例。
横杠-越多，箭头越长。 通过在箭头定义的后面加一个冒号及文字的方式来添加标签。
在这个例子中，User并没有定义，而是直接拿来当做一个角色使用。
```
{% plantuml %}
User -> (Start)
User --> (Use the application) : A small label

:Main Admin: ---> (Use the application) : This is\nyet another\nlabel
{% endplantuml %}

如果一个角色或者用例继承于另一个，那么可以用<|--符号表示
{% plantuml %}
:Main Admin: as Admin
(Use the application) as (Use)

User <|-- Admin
(Start) <|-- (Use)
{% endplantuml %}
```

{% plantuml %}
User -> (Start)
User --> (Use the application) : A small label

:Main Admin: ---> (Use the application) : This is\nyet another\nlabel
{% endplantuml %}

如果一个角色或者用例继承于另一个，那么可以用<|--符号表示
{% plantuml %}
:Main Admin: as Admin
(Use the application) as (Use)

User <|-- Admin
(Start) <|-- (Use)
{% endplantuml %}

### 注释
可以用note left of , note right of , note top of , note bottom of等关键字给一个对象添加注释。
注释还可以通过note关键字来定义，然后用..连接其他对象。
```
{% plantuml %}
:Main Admin: as Admin
(Use the application) as (Use)

User -> (Start)
User --> (Use)

Admin ---> (Use)

note right of Admin : This is an example.

note right of (Use)
  A note can also
  be on several lines
end note

note "This note is connected\nto several objects." as N2
(Start) .. N2
N2 .. (Use)
{% endplantuml %}
```

{% plantuml %}
:Main Admin: as Admin
(Use the application) as (Use)

User -> (Start)
User --> (Use)

Admin ---> (Use)

note right of Admin : This is an example.

note right of (Use)
  A note can also
  be on several lines
end note

note "This note is connected\nto several objects." as N2
(Start) .. N2
N2 .. (Use)
{% endplantuml %}

### 改变箭头方向
默认连接是竖直方向的，用--表示，可以用一个横杠或点来表示水平连接。
```
{% plantuml %}
:user: --> (Use case 1)
:user: -> (Use case 2)
{% endplantuml %}
```

{% plantuml %}
:user: --> (Use case 1)
:user: -> (Use case 2)
{% endplantuml %}

也可以通过翻转箭头来改变方向。
```
{% plantuml %}
(Use case 1) <.. :user:
(Use case 2) <- :user:
{% endplantuml %}
```

{% plantuml %}
(Use case 1) <.. :user:
(Use case 2) <- :user:
{% endplantuml %}

还可以通过给箭头添加left, right, up或down等关键字来改变方向。
```
{% plantuml %}
:user: -left-> (dummyLeft) 
:user: -right-> (dummyRight) 
:user: -up-> (dummyUp)
:user: -down-> (dummyDown)
{% endplantuml %}
```

{% plantuml %}
:user: -left-> (dummyLeft) 
:user: -right-> (dummyRight) 
:user: -up-> (dummyUp)
:user: -down-> (dummyDown)
{% endplantuml %}

默认从上往下构建图示, 你可以用left to right direction命令改变图示方向。
```
{% plantuml %}
left to right direction
user1 --> (Usecase 1)
user2 --> (Usecase 2)
{% endplantuml %}
```

{% plantuml %}
left to right direction
user1 --> (Usecase 1)
user2 --> (Usecase 2)
{% endplantuml %}

### 改变颜色
用skinparam改变字体和颜色。
```
{% plantuml %}
skinparam handwritten true

skinparam usecase {
	BackgroundColor DarkSeaGreen
	BorderColor DarkSlateGray

	BackgroundColor<< Main >> YellowGreen
	BorderColor<< Main >> YellowGreen
	
	ArrowColor Olive
	ActorBorderColor black
	ActorFontName Courier

	ActorBackgroundColor<< Human >> Gold
}

User << Human >>
:Main Database: as MySql << Application >>
(Start) << One Shot >>
(Use the application) as (Use) << Main >>

User -> (Start)
User --> (Use)

MySql --> (Use)
{% endplantuml %}
```

{% plantuml %}
skinparam handwritten true

skinparam usecase {
	BackgroundColor DarkSeaGreen
	BorderColor DarkSlateGray

	BackgroundColor<< Main >> YellowGreen
	BorderColor<< Main >> YellowGreen
	
	ArrowColor Olive
	ActorBorderColor black
	ActorFontName Courier

	ActorBackgroundColor<< Human >> Gold
}

User << Human >>
:Main Database: as MySql << Application >>
(Start) << One Shot >>
(Use the application) as (Use) << Main >>

User -> (Start)
User --> (Use)

MySql --> (Use)
{% endplantuml %}

一个较完整的例子
```
{% plantuml %}
left to right direction
skinparam packageStyle rectangle
actor customer
actor clerk
rectangle checkout {
  customer -- (checkout)
  (checkout) .> (payment) : include
  (help) .> (checkout) : extends
  (checkout) -- clerk
}
{% endplantuml %}
```

{% plantuml %}
left to right direction
skinparam packageStyle rectangle
actor customer
actor clerk
rectangle checkout {
  customer -- (checkout)
  (checkout) .> (payment) : include
  (help) .> (checkout) : extends
  (checkout) -- clerk
}
{% endplantuml %}