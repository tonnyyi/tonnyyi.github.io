---
title: jar命令
tags:
  - java
  - jar
categories:
  - java
date: 2021-06-11 14:28:27
---

## jar命令

.jar（Java Archive）是Java中的一种归档文件格式，只要安装了 JDK ，都可以使用 jar 命令将多个文件打包成一个 .jar 格式的 jar 包或者 war 包，就像 Linux 里的 tar 命令一样，以下为 jar 命令的参数：

```shell
用法: jar {ctxui}[vfmn0PMe] [jar-file] [manifest-file] [entry-point] [-C dir] files ...
选项:
    -c  创建新档案
    -t  列出档案目录
    -x  从档案中提取指定的 (或所有) 文件
    -u  更新现有档案
    -v  在标准输出中生成详细输出
    -f  指定档案文件名
    -m  包含指定清单文件中的清单信息
    -n  创建新档案后执行 Pack200 规范化
    -e  为捆绑到可执行 jar 文件的独立应用程序
        指定应用程序入口点
    -0  仅存储; 不使用任何 ZIP 压缩
    -P  保留文件名中的前导 '/' (绝对路径) 和 ".." (父目录) 组件
    -M  不创建条目的清单文件
    -i  为指定的 jar 文件生成索引信息
    -C  更改为指定的目录并包含以下文件
如果任何文件为目录, 则对其进行递归处理。
清单文件名, 档案文件名和入口点名称的指定顺序
与 'm', 'f' 和 'e' 标记的指定顺序相同。

示例 1: 将两个类文件归档到一个名为 classes.jar 的档案中:
       jar cvf classes.jar Foo.class Bar.class
示例 2: 使用现有的清单文件 'mymanifest' 并
           将 foo/ 目录中的所有文件归档到 'classes.jar' 中:
       jar cvfm classes.jar mymanifest -C foo/ .
```

## 查看jar包内容

```shell
jar -tvf xx.jar

# 如果你想查看解压一个jar的详细过程，而这个jar包又很大，屏幕信息会一闪而过，这时你可以把列表输出到一个文件中，慢慢欣赏！
jar -tvf xx.jar>files.txt
```



## 解压jar/war文件

```shell
# 解压全部文件
jar -xvf xx.jar
jar -xvf xx.war

# 提取指定文件，提取后，文件会被放到与其package声明相同的目录结构下
java -xvf xx.jar com/test/Test.class
```



## 打包jar/war文件

```shell
# 把当前目录的所有文件都打进去
jar -cvf test.jar *

# 只打包某个文件或目录，后面接文件、目录即可
jar -cvf test.jar WEB-INF/ com/

# 打包时不压缩内容
jar -cvf0 test.jar *

# 先切换工作目录再打包  先切换到test目录再执行jar命令
jar -cvf test.jar * -C test/
```

### manifest文件

打包时会默认创建`mainifest.mf`文件，我们也可以选择不生成`manifest.mf`文件，或者使用已有的`manifest.mf`文件，特别是`springboot`打包出来的jar包中，`manifest.mf`包含很多扩展信息，我们必须要复用，否则会跑不起来。

```shell
# 不生成manifest.mf文件
jar -cvfM test.jar *

# 复用已有的manifest.mf文件
jar -cvfm test.jar mymanifest.mf *
```



#### `manifest.mf`文件规范

>- 不能有空行和空格的地方：第一行不可以是空行，行与行之间不能有空行，每行的行尾不可以有空格
>- 一定要有空行的地方，最后一行得是空行
>- 一定有空格的地方：key: value 在分号后面一定要写一个空格
>- 每行不能超过七十多的字符
>- Class-Path里边的内容用空格分隔而不是逗号或者分号



普通示例

```ini
Manifest-Version: 1.0
Created-By: JDJ example
Class-Path: mail.jar activation.jar
Main-Class: com.test.PackageClass
Name: com/example/myapp/
Specification-Title: MyApp
Specification-Version: 2.4
Specification-Vendor: example.com
Implementation-Title: com.example.myapp
Implementation-Version: 2002-03-05-A
Implementation-Vendor: example.com
```



springboot打包出来的manifest.mf文件示例

```ini
Manifest-Version: 1.0
Implementation-Title: znyj-server
Implementation-Version: 0.0.1-FY1013
Built-By: tonnyyi
Implementation-Vendor-Id: com.iflytek
Spring-Boot-Version: 2.1.2.RELEASE
Main-Class: org.springframework.boot.loader.JarLauncher
Start-Class: com.iflytek.znyjserver.ZnyjServerApplication
Spring-Boot-Classes: BOOT-INF/classes/
Spring-Boot-Lib: BOOT-INF/lib/
Created-By: Apache Maven 3.6.3
Build-Jdk: 1.8.0_261
Implementation-URL: https://projects.spring.io/spring-boot/#/spring-bo
 ot-starter-parent/znyj-server
```

属性介绍

```
Manifest-Version:
用来定义manifest文件的版本，例如：Manifest-Version: 1.0

Created-By:
声明该文件的生成者，一般该属性是由jar命令行工具生成的，例如：Created-By: Apache Ant 1.5.1

Main-Class
定义jar文件的入口类，该类必须是一个可执行的类(包含main方法的类)，一旦定义了该属性即可通过 java -jar x.jar来运行该jar文件。
运行Jar: java -jar test.jar
当运行上述命令时JVM将在test.jar文件中的MANIFEST.MF文件中查找Main-Class属性的值，并尝试运行该类。如果在yveshe.jar文件中未包含Main-Class属性，则上述命令将生成错误。

Class-Path:
指定jar包的依赖关系，class loader会依据这个路径来搜索class.默认是相对路径,相对该jar所在的父文件夹.
可以在其manifest 文件中为JAR文件设置CLASSPATH。属性名称叫作类路径，必须在自定义清单文件中指定。 它是一个空格分隔的jar文件，zip文件和目录的列表。(不区分系统都是以空格来分隔多个jar文件)以下是一个属性配置例子:
Class-Path: reference1.jar file:/c:/book/ http://www.test.com/reference2.jar
这条命令配置了该test.jar依赖了三个jar:
一个JAR文件reference1.jar，一个使用文件协议文件的目录：c:/book/和另一个使用HTTP协议的JAR文件http://www.test.com/reference2.jar
注意: 当使用java命令使用-jar(比如java -jar test.jar)选项运行JAR文件时，将忽略Jar中 manifest文件之外的任何CLASSPATH设置。

Sealed:
密封 JAR 文件中的一个包意味着在这个包中定义的所有类都必须在同一个 JAR 文件中找到。这使包的作者可以增强打包类之间的版本一致性。密封还提供了防止代码篡改的手段。
要密封包，需要在 JAR 的 manifest 文件中为包添加一个 Name 头，然后加上值为“true”的 Sealed 头。与可执行的 JAR 一样，可以在创建 JAR 时，通过指定一个具有适当头元素的 manifest 文件密封一个 JAR，如下所示：
Name: com/yveshe/
Sealed: true
Name 头标识出包的相对路径名。它以一个“/”结束以与文件名区别。在 Name 头后面第一个空行之前的所有头都作用于在 Name 头中指定的文件或者包。在上述例子中，因为 Sealed 头出现在 Name 头后并且中间没有空行，所以 Sealed 头将被解释为只应用到包 com/yveshe上。如果试图从密封包所在的 JAR 文件以外的其他地方装载密封包中的一个类，那么 JVM 将抛出一个SecurityException 。文件头Name的值为该封装的相对路径名。注意，该路径名由‘/’结束以区别于文件名。
```



## 替换jar包内文件

要想替换文件，在linux下可以把jar包解压，替换后再打成jar，麻烦。其实可以使用jar命令直接替换。

```shell
# 一定先建好目录结构com/test，然后把test.class文件放入test目录，再打包
# mkdir -vp com/test
jar -uvf test.jar com/test/test.class
```

> **注意**：`test.class`文件一定要按其package声明，放到对应的目录层级下，然后再打包，否则打包后，其在jar包中的路径会与其package声明不一致，从而导致出错。



## 其他

```shell
# 编译时依赖其他jar包
javac -cp b.jar lib/a.jar XX.java
```

反编译

```shell
# 只看方法声明
javap org/springframework/boot/loader/ExecutableArchiveLauncher.class

# 输出方法体，以java字节码的形式
javap -c org/springframework/boot/loader/ExecutableArchiveLauncher.class
```

