---
title: maven
tags:
  - maven
categories:
  - maven
  - tools
date: 2018-03-30 15:16:30
---

## 依赖示例
```xml
<dependency>
    <groupId>com.zaxxer</groupId>
    <artifactId>HikariCP</artifactId>
    <version>2.4.6</version>
    <type>jar</type>
    <scope>compile</scope>
    <optional>true</optional>
    <classifier>jdk15</classifier>
    <systemPath>${project.basedir}/lib/my-jar.jar</systemPath>
    <exclusions>
        <exclusion>
            <groupId>org.slf4j</groupId>
            <artifactId>slf4j-api</artifactId>
        </exclusion>
    </exclusions>
</dependency>
```
<!-- more -->
### 坐标
构件的唯一标识
- `groupId` : 定义项目所属组, **必选**
- `artifactId` : 定义项目在组内的唯一id, **必选**
- `1.0.0-SNAPSHOT` : 版本号, **必选**, Spring: M1, M2, RC1, RC2, RELEASE

**其他非必选属性**
- `type` : 打包方式, 默认: `jar`, 可选值: `war`, `pom`, `ejb-client`
- `scope`: 作用域, 默认: `compile`, 会影响依赖调解
- `optional`: 可选依赖, 默认:`false`, 比如: jdbc驱动实现mysql/Oracle, 依赖者根据需要手动添加
- `classifier` : 帮助输出辅助构件, jdk14/jdk15/javadoc/source, **无法直接定义自己项目的classifier**
- `systemPath`: 指定jar在本地文件系统的位置, **只能在`SYSTEM`作用域下使用**
- `exclusions`: 依赖排除, 只需要`groupId`+`artifactId `

### 版本号
> <主版本>.<次版本>.<增量版本>

**快照版** : xxx-api-0.0.1-20180329.080444-1029.jar
- `0.0.1`: `0.0.1-SNAPSHOT`
- `20180329.080444`: 2018年3月29号 8点4分44秒推送到了仓库
- `1029`: 当前版本的第1029次快照


## 依赖

### 依赖范围
依赖范围用于控制传传递性依赖和影响classpath. maven编译/测试/运行时使用不同classpath, 有三种: 编译/测试/运行时
scope可选值有: 
- `compile`: **默认值**, 编译 + 测试 + 运行时, 如: spring-core
- `provided`: 编译 + 测试, JDK或运行时容器已提供, 如: servlet-api
- `runtime`: 测试 + 运行时
- `test`: 测试, 用于测试代码编译与执行, 如: junit
- `system`: 同`provided`, 通过`systemPath`指定jar包的位置, 不通过Maven仓库解析, 往往与本机系统绑定. **不推荐**
- `import`: 2.0.9版本后可用, **对三种都不生效**, 在依赖类型为pom时使用, 将依赖的`dependencyManagement`部分合并到当前项目中

### 传递性依赖
A -> B -> C, 只需要显示的依赖A, B 和 C自动引入
A 对于 B 是第一直接依赖, B 对于 C 是第二直接依赖, A 对于 C 是传递性依赖
传递性依赖范围(scope)的确定:

|        | compile | test | provided | runtime |
|:------:|:-------:|:----:|:--------:|:-------:|
|compile | compile |   -  |     -    | runtime |
|  test  |   test  |   -  |     -    |  test   |
|provided|provided |   -  | provided | provided|
| runtime| runtime |   -  |     -    | runtime |

第一列代表第一直接依赖, 第一行代表第二直接依赖, 中间为传递性依赖范围

### 传递调解
**原则: **
> 1. 路径最短者优先
> 2. 第一声明者优先

关于1:
a -> c -> d -> M2.0 
a -> b -> M1.0
最终 a -> M1.0 

关于2:
a -> c -> M1.0
a -> b -> M2.0
最终 a -> M1.0

### 可选依赖
只对直接依赖生效, 间接依赖不传递.
A -> B, B -> X(optional), B -> Y(optional), A值对B有依赖, 对X, Y没有依赖, 不管它们之间的依赖范围是怎样. A可以根据实际使用功能引入依赖. mysql/orcale

### 排除依赖
`<exclusions>`用于排除传递依赖(不管层级), 不需要version. 层级无关: a -> b -> c -> d, 可在b依赖处exclusion掉d

### 归类依赖
使用`<properties>`控制多个依赖版本, 便于维护

### 依赖查看
```bash
mvn dependency:list     # 列表, 坐标+依赖范围
mvn dependency:tree     # 依赖树
mvn dependency:analyze  
```
结果有这几行: Used declared dependencies: 使用到但未显示声明; Unused declared dependencies: 未使用但显示声明的, 只分析编译主代码和测试代码用到的依赖, 运行时依赖发现不了. 需谨慎看待, 不要随便删.

## [生命周期](https://maven.apache.org/guides/introduction/introduction-to-the-lifecycle.html#Lifecycle_Reference)
> `mvn clean install` 到底干了什么, 怎么干的, 谁让他这么干的? 

maven之前项目构建的生命周期就已存在(清理/编译/测试/部署等), maven对其进行了抽象, 生命周期的实际行为全都有插件来完成.  每个生命周期包含一定的阶段(phase), 阶段间是有顺序的, 后面的阶段依赖于前面的阶段(调用一个阶段时会先执行其前面的阶段). 但生命周期之间是独立, 不互相依赖, 所以你执行clean时, 不会执行default生命周期. 定义: `${M2_HOME}/lib/maven-core-xx.jar/META-INF/plexus/components.xml`
### clean生命 
其目的是清理项目, 包含三个阶段: 
- `pre-clean`: 执行清理前的工作
- `clean`: 清理上一次构建生成的文件
- `post-clean`: 执行清理后的工作

### default生命周期
目的是构建项目, 包含的的阶段:
- `validate`
- `initialize`
- `generate-sources`
- `process-sources`: 对src/main/resources目录的内容进行变量替换等操作, 复制到项目输出的主classpath中
- `generate-resources`
- `process-resources`
- `compile`: 编译项目主源码, src/main/java
- `process-classes`
- `generate-test-sources`
- `process-test-sources`: 处理测试资源文件, src/test/resources
- `generate-test-resources`
- `process-test-resources`
- `test-compile`: 编译测试代码, src/test/java
- `process-test-classes`
- `test`: 单元测试, 测试代码不会打包或部署
- `prepare-package`
- `package`: 打包, jar/war
- `pre-integration-test`
- `integration-test`
- `post-integration-test`
- `verify`
- `install`: 安装到本地仓库
- `deploy`: 部署包到远程仓库

### site生命周期
目的为建立项目站点, 包含的阶段: 
- `pre-site`
- `site`: 生成项目站点文档
- `post-site`:
- `site-deploy`: 发布项目站点文档到服务器

### 命令行与生命周期关系
- `mvn clean`: 执行clean生命周期的`pre-clean`和`clean`阶段
- `mvn test`: 执行default生命周期从`validate`到`test`之间的所有阶段
- `mvn clean install`: 执行clean生命周期的`pre-clean`和`clean` + default生命周期的从`validate`到`isntall`的所有阶段.
- `mvn clean deploy site-deploy`: 执行clean生命周期的`pre-clean`和`clean` + default生命周期的的所有阶段 + site生命周期的所有阶段

## [插件](https://maven.apache.org/plugins/index.html)
### 插件目标(goal)
Maven核心只定义了抽象的生命周期, 具体的任务交由插件去完成. 
每个插件可以实现多个功能, 每个功能就是一插件目标. `dependency:list`, `dependency:tree`冒号前是插件前缀, 冒号后是该插件的目标.
生命周期的阶段与插件的目标相互绑定. 如: default生命周期的compile阶段与maven-compile-plugin插件的compile目标. 

**如果生命周期的阶段没有与任何插件的目标绑定, 则该阶段啥也会干**

#### [内置绑定](https://maven.apache.org/guides/introduction/introduction-to-the-lifecycle.html#Built-in_Lifecycle_Bindings) 
绑定关系定义(版本3以后才有): `${M2_HOME}/lib/maven-core-xx.jar/META-INF/plexus/default-binding.xml`

**clean生命周期阶段与插件目标绑定关系**

|生命周期阶段|插件目标|
|:---:|:---:|
|pre-clean|-|
|clean|maven-clean-plugin:clean|
|post-clean|-|

**default生命周期阶段与插件目标绑定关系** 
> 1. 项目的打包类型(jar/war/pom/ear/maven-plugin)不同, 绑定关系有所区别
> 2. default生命周期还有许多其他阶段, 默认没有绑定任何插件, 因此也没有任何实际行为

|生命周期阶段|插件目标|执行任务|
|:---:|:---:|:---:|
|process-resources|maven-resource-plugin:resources|复制主资源文件到主输出目录|
|compile|maven-compiler-plugin:compile|编译主代码到主输出目录|
|process-test-resources|maven-resources-plugin:testResources|复制测试资源文件到测试输出目录|
|test-compile|maven-compile-plugin:testCompile|编译测试代码到测试数据目录|
|test|maven-surefire-plugin:test|执行测试用例|
|package|maven-jar-plugin:jar|创建项目jar等|
|install|maven-install-plugin:install|部署构建到本地仓库|
|deploy|maven-deploy-plugin:deploy|部署构件到远程仓库|

**site生命周期阶段与插件目标绑定关系**

|生命周期阶段|插件目标|
|:---:|:---:|
|pre-site|-|
|site|maven-site-plugin:site|
|post-site|-|
|site-deploy|maven-site-plugin:deploy|

#### 自定义绑定
首先查询插件详细信息, 关注目标默认绑定的生命周期阶段(Bound to phase)
```bash
mvn help:describe -Dplugin=org.apache.maven.plugins:maven-source-plugin:3.0.1 -Ddetail
mvn help:describe -Dplugin=source -Ddetail  # 简略写法
```
输出
```
...
source:jar
   Description: This plugin bundles all the sources into a jar archive.
   Implementation: org.apache.maven.plugins.source.SourceJarMojo
   Language: java
   Bound to phase: package
   Before this mojo executes, it will call:
    Phase: 'generate-sources'
...
```
示例: 生成源码jar包
```xml
<build>
    <plugins>
        <plugin>
            <groupId>org.apache.maven.plugins</groupId>
            <artifactId>maven-source-plugin</artifactId>
            <version>3.0.0</version>
            <executions>
                <execution>
                    <id>attach-sources</id>
                    <phase>verify</phase>
                    <goals>
                        <goal>jar</goal>
                    </goals>
                </execution>
            </executions>
            <configuration>
                <attach>true</attach>
            </configuration>
        </plugin>
    </plugins>
</build>
```

### 插件配置
#### 命令行中使用
命令行时使用`-D`参数(这是java自带的, 通过-D设置java系统属性): 
```bash
mvn install -DskipTests     # 编译不执行
mvn install -Dmaven.test.skip=true  # 不编译不执行
```

#### pom中全局配置
```xml
<plugin>
    <groupId>org.apache.maven.plugins</groupId>
    <artifactId>maven-compiler-plugin</artifactId>
    <version>3.1</version>
    <configuration>
        <source>${java.src.version}</source>
        <target>${java.target.version}</target>
        <encoding>${project.encoding}</encoding>
    </configuration>
</plugin>
```

#### pom中插件任务配置
注意configuration的位置
```xml
<plugin>
    <groupId>org.apache.maven.plugins</groupId>
    <artifactId>maven-antrun-plugin</artifactId>
    <executions>
        <execution>
            <id>ant-validate</id>
            <phase>validate</phase>
            <goals>
                <goal>run</goal>
            </goals>
            <configuration>
                <tasks>
                    <echo>Ant run bound to validate phase.</echo>
                </tasks>
            </configuration>
        </execution>
    </executions>
</plugin>
```

### 插件解析
通过命令行执行插件目标: 
```bash
mvn org.apache.maven.plugins:maven-dependency-plugin:2.10:tree
mvn dependency:tree     # 使用目标前缀执行插件目标
```
#### [目标前缀解析](https://maven.apache.org/guides/introduction/introduction-to-plugin-prefix-mapping.html)
执行`mvn dependency:tree`时, maven如如何选择对应的插件的? 
如果插件的artifactId符合`maven-${prefix}-plugin`(官方插件格式, 自己别用) 或者 `${prefix}-maven-plugin`格式, maven会自动映射prefix到这个插件上.

查找流程:

1. 下载远程插件仓库的`maven-metadata.xml`文件, 重命名为`maven-metadata-${repoId}.xml`, 放到${groupId}路径下
2. 将该路径下的`maven-metadata-local.xml`文件(如果有的话)和`maven-metadata-${repoId}.xml`文件合并
3. 从合并后的内容中尝试解析出前缀对应的插件, 如果没找到则回到第一步, 查找下一个插件组.
  `~/.m2/repository/org/apache/maven/plugins/maven-metadata-<仓库ID>.xml` 和 `/~/.m2/repository/org/codehaus/mojo/maven-metadata-<仓库ID>.xml`
  配合`setting.xml`中`pluginGroups`配置

> pluginGroups内的配置其用处就是在命令行使用某个插件但未指定groupId时搜索对应的插件

## 仓库
```
Maven仓库
    - 本地仓库(~/.m2/repository)
    - 远程仓库
        - 中央仓库
        - 其他公共库
        - 私服Nexus
```
中央仓库的定义在超级POM中(`${M2_HOME}/lib/maven-model-builder-xx.jar/org/apache/maven/model/pom-4.0.0.xml`), 所有项目都会继承它.

进入本地仓库方式:
1. 从远程仓库拉取
2. `mvn install:install-file -Dfile=./xxx -DgroupId=com.xxx -DartifactId=xxx -Dversion=xxx -Dpackaging=jar`

远程仓库:
```xml
<repositories>
    <repository>
        <id>xxx-xxxxx-releases</id>
        <name>Repository for releases artifacts</name>
        <url>http://pixel.xxxxx.com/repository/group-releases</url>
        <snapshots>
            <enabled>false</enabled>
        </snapshots>
        <releases>
            <enabled>true</enabled>
            <updatePolicy>interval:60</updatePolicy>
        </releases>
    </repository>
</repositories>    
```
几个子元素的说明:
- `enabled`: 是否允许发布/快照版本下载
- `updatePolicy`: 从远程仓库更新的频率, 默认: Daily
- `checksumPolicy`: 校验文件策略, 上传构件时会同时上传校验文件. 下载时maven集合校验文件来验证构件, 校验失败后依据该配置执行操作. 默认: warn, 还有fail/ignore

### 远程仓库认证
**`id`必须与`repository`中的id一致**
```xml
<servers>
    <server>
        <id>xxxxx.repo</id>
        <username>fw-deploy</username>
        <password>example</password>
    </server>
</servers>
```

### 从仓库解析依赖流程

1. 如果依赖范围是`system`, 直接从本地文件系统解析构件
2. 根据依赖坐标计算仓库路径, 尝试直接从本地仓库寻找构件
3. 当本地仓库不存在构件, 且依赖的是正式版本, 则从遍历所有远程仓库, 发现后下载
4. 如果依赖版本的是RELEASE或LATEST, 则基于更新策略读取所有远程仓库的元数据(groupId/artifactId/maven-metadata.xml), 并重命名为`maven-metadata-<RepositoryID>.xml`, 与本地仓库对应的元数据(如果有的话)合并后(maven-metadata-local.xml), 计算真实值
5. 如果依赖的是SNAPSHOT逻辑同上
6. 如果解析到的构件版本是时间戳格式, 则将其复制为非时间戳格式(如: SNAPSHOT), 并使用该非时间戳版本

### install
1. install时1.0.0-SNAPSHOT/下生成maven-metadata-local.xml
2. 从远程仓库拉取maven-metadata.xml, 命名为"maven-metadata-\<RepositoryID>.xml"，并保存到本地仓库相应目录
3. 比较lastUpdated字段, 如果本地文件中的值较大, 则使用本地仓库中的jar, 否则下载远程仓库的`xxx-api-0.0.1-20180329.080444-1029.jar`到本地, 然后复制并重命名为`xxx-api-0.0.1-SNAPSHOT.jar`, 把原来的jar覆盖掉

## [超级pom](http://maven.apache.org/ref/3.0.4/maven-model-builder/super-pom.html)
文件`${M2_HOME}/lib/maven-model-builder-xx.jar/org/apache/maven/model/pom-4.0.0.xml`是所有maven项目都会默认继承它, 类似Java里的Object类.

### 标准目录结构
```
src
    main
    java
    resources
    webapp
        WEB-INF
        web.xml
    test
        java
        resources 
target
    classes
    generated-sources
    generated-test-sources
    {build-filename}
    test-classes
pom.xml
```

### archetype生成项目骨架
```bash
mvn archetype:generate
mvn groupId:artifactId:version:goal
```

maven3默认使用jdk1.5编译

```xml
<properties>
   <maven.compiler.source>1.8</maven.compiler.source>
   <maven.compiler.target>1.8</maven.compiler.target>
</properties>

<plugin>
    <groupId>org.apache.maven.plugins</groupId>
    <artifactId>maven-compiler-plugin</artifactId>
    <version>(whatever version is current)</version>
    <configuration>
        <!-- or whatever version you use -->
        <source>1.7</source>
        <target>1.7</target>
    </configuration>
</plugin>
```

### 聚合与继承
通常一起使用
> **聚合是为了快速方便的构建项目, 继承是为了消除重复配置.**
> 对于聚合模块来说, 它知道有哪些被聚合的模块, 但那些被聚合的模块不知道该模块的存在.
> 对于继承关系的父POM来说, 它不知道有哪些模块继承于它, 但那些子模块都必须知道自己的父POM是什么.

聚合模块
- packaging: pom
- 一般位于顶层

模块
- 一般作为子目录
- parent内的groupId/artifactId/version是必须的, relativePath默认值为../pom.xml

#### 可继承的pom元素
- **groupId** : 组织id
- **version** : 版本号
- **description** : 项目描述
- **organization** : 项目所属组织信息
- **inceptionYear** : 项目创始年份
- **url** : 项目URL
- **developers** : 开发者列表 
- **contributors** : 贡献者列表
- **distributionManagement** : 部署配置 
- **issueManagement** : 缺陷跟踪系统信息
- **ciManagerment** : 持续集成配置
- **scm** : 版本控制信息
- **mailingLists** : 邮件列表
- **propertes** : 属性列表
- **dependencies** : 依赖列表
- **dependencyManagement** : 依赖管理配置 
- **repositories** : 仓库配置
- **build** : 源码目录配置, 输出目录配置, 插件配置, 插件管理配置等
- **reporting**: 项目报告输出目录, 报告插件配置等

#### dependencyManagement
不会引入实际的依赖, 但会约束dependencies下的依赖使用. 可以省去version/scope配置. 省去的配置不多, 但推荐用, 而且要放到父POM中.
**import**依赖范围的作用: 将目标POM中的dependencyManagement配置导入并合并到当前POM的dependencyManagement元素中. 一般都指向的打包类型为POM的模块

### Maven属性
1. 内置属性
- `${basedir}` : 项目根目录, 即包含pom.xml的目录
- `${version}` : 项目版本

2. POM属性
用户可以使用该类型的属性引用POM文件对应元素的值. 如`${project.artifactId}`就对应了`<project>`节点下`<artifactId>`节点的值. 常用POM属性包括: 
- `${project.build.sourceDirectory}` : 项目主源码目录, 默认: `src/main/java`
- `${project.build.testSourceDirectory}` : 项目测试代码目录, 默认: `src/test/java`
- `${project.build.directory}` : 项目构建输出目录, 默认: `target/`
- `${project.outputDirectory}` : 项目主代码编译输出目录, 默认: `target/classes/`
- `${project.testOutputDirectory}` : 项目测试代码输出目录, 默认: `target/test-classes`
- `${project.groupId}` : 项目的groupId
- `${project.artifactId}` : 项目的artifactId
- `${project.version}` : 项目的版本, 等同于 `${version}`
- `${project.build.finalName}` : 项目打包输出文件名称, 默认: `${project.artifact}-${project.version}`

3. 自定义属性
`<properties>`节点下的定义的属性

4. settings属性
与POM属性类似, 不过引用的Settings.xml中的元素值. 以`settings`开头, 如: `${settings.localRepository}`

5. Java系统属性
引用Java系统属性, 比如`${user.home}`指向了用户目录. 可以使用`mvn help:system`

6. 环境变量属性
以`env.`开头, 如: `${env.JAVA_HOME}`, 可以使用`mvn help:system`

### [setting.xml](https://maven.apache.org/settings.html)
```xml
<settings xmlns="http://maven.apache.org/SETTINGS/1.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:schemaLocation="http://maven.apache.org/SETTINGS/1.0.0 http://maven.apache.org/xsd/settings-1.0.0.xsd">
    <localRepository>/Users/tonnyyi/workspace/MavenRepository</localRepository>
    
    <mirrors>
        <mirror>
            <id>nexus-aliyun</id>
            <mirrorOf>central</mirrorOf>
            <name>Nexus aliyun</name>
            <url>http://maven.aliyun.com/nexus/content/groups/public</url>
        </mirror>
    </mirrors>
    <profiles>
        <!-- 项目默认的编译级别为1.8 -->
        <profile>
            <id>jdk-1.8</id>
            <activation>
                <jdk>1.8</jdk>
            </activation>
            <properties>
                <maven.compiler.source>1.8</maven.compiler.source>
                <maven.compiler.target>1.8</maven.compiler.target>
                <maven.compiler.compilerVersion>1.8</maven.compiler.compilerVersion>
            </properties>
        </profile>

        <profile>
            <id>aliyunRep</id>
            <activation>
                <jdk>1.8</jdk>
            </activation>
            <repositories>
                <repository>
                    <id>aliyunRep</id>
                    <name>Aliyun Maven Respository</name>
                    <url>http://maven.aliyun.com/nexus/content/groups/public/</url>
                    <releases>
                        <enabled>true</enabled>
                    </releases>
                    <snapshots>
                        <enabled>false</enabled>
                    </snapshots>
                </repository>
            </repositories>
            <pluginRepositories>
                <pluginRepository>
                    <id>aliyunRep</id>
                    <name>Aliyun Maven Respository</name>
                    <url>http://maven.aliyun.com/nexus/content/groups/public/</url>
                    <releases>
                        <enabled>true</enabled>
                    </releases>
                    <snapshots>
                        <enabled>false</enabled>
                    </snapshots>
                </pluginRepository>
            </pluginRepositories>
        </profile>
    </profiles>

    <pluginGroups>
        <pluginGroup>org.apache.maven.plugins</pluginGroup>
        <pluginGroup>org.unidal.maven.plugins</pluginGroup>
        <pluginGroup>org.jvnet.hudson.tools</pluginGroup>
    </pluginGroups>

    <servers>
        <server>
            <configuration>
                <httpConfiguration>
                  <all>
                    <connectionTimeout>5000</connectionTimeout>
                    <readTimeout>10000</readTimeout>
                  </all>
                </httpConfiguration>
            </configuration>
        </server>
    </servers>
    <activeProfiles>
        <activeProfile>jdk-1.8</activeProfile>
        <activeProfile>aliyunRep</activeProfile>
    </activeProfiles>
</settings>
```

#### 超级pom
```xml
<project>
  <modelVersion>4.0.0</modelVersion>

  <repositories>
    <repository>
      <id>central</id>
      <name>Central Repository</name>
      <url>https://repo.maven.apache.org/maven2</url>
      <layout>default</layout>
      <snapshots>
        <enabled>false</enabled>
      </snapshots>
    </repository>
  </repositories>

  <pluginRepositories>
    <pluginRepository>
      <id>central</id>
      <name>Central Repository</name>
      <url>https://repo.maven.apache.org/maven2</url>
      <layout>default</layout>
      <snapshots>
        <enabled>false</enabled>
      </snapshots>
      <releases>
        <updatePolicy>never</updatePolicy>
      </releases>
    </pluginRepository>
  </pluginRepositories>

  <build>
    <directory>${project.basedir}/target</directory>
    <outputDirectory>${project.build.directory}/classes</outputDirectory>
    <finalName>${project.artifactId}-${project.version}</finalName>
    <testOutputDirectory>${project.build.directory}/test-classes</testOutputDirectory>
    <sourceDirectory>${project.basedir}/src/main/java</sourceDirectory>
    <scriptSourceDirectory>${project.basedir}/src/main/scripts</scriptSourceDirectory>
    <testSourceDirectory>${project.basedir}/src/test/java</testSourceDirectory>
    <resources>
      <resource>
        <directory>${project.basedir}/src/main/resources</directory>
      </resource>
    </resources>
    <testResources>
      <testResource>
        <directory>${project.basedir}/src/test/resources</directory>
      </testResource>
    </testResources>
    <pluginManagement>
      <!-- NOTE: These plugins will be removed from future versions of the super POM -->
      <!-- They are kept for the moment as they are very unlikely to conflict with lifecycle mappings (MNG-4453) -->
      <plugins>
        <plugin>
          <artifactId>maven-antrun-plugin</artifactId>
          <version>1.3</version>
        </plugin>
        <plugin>
          <artifactId>maven-assembly-plugin</artifactId>
          <version>2.2-beta-5</version>
        </plugin>
        <plugin>
          <artifactId>maven-dependency-plugin</artifactId>
          <version>2.8</version>
        </plugin>
        <plugin>
          <artifactId>maven-release-plugin</artifactId>
          <version>2.3.2</version>
        </plugin>
      </plugins>
    </pluginManagement>
  </build>

  <reporting>
    <outputDirectory>${project.build.directory}/site</outputDirectory>
  </reporting>

  <profiles>
    <!-- NOTE: The release profile will be removed from future versions of the super POM -->
    <profile>
      <id>release-profile</id>

      <activation>
        <property>
          <name>performRelease</name>
          <value>true</value>
        </property>
      </activation>

      <build>
        <plugins>
          <plugin>
            <inherited>true</inherited>
            <artifactId>maven-source-plugin</artifactId>
            <executions>
              <execution>
                <id>attach-sources</id>
                <goals>
                  <goal>jar</goal>
                </goals>
              </execution>
            </executions>
          </plugin>
          <plugin>
            <inherited>true</inherited>
            <artifactId>maven-javadoc-plugin</artifactId>
            <executions>
              <execution>
                <id>attach-javadocs</id>
                <goals>
                  <goal>jar</goal>
                </goals>
              </execution>
            </executions>
          </plugin>
          <plugin>
            <inherited>true</inherited>
            <artifactId>maven-deploy-plugin</artifactId>
            <configuration>
              <updateReleaseInfo>true</updateReleaseInfo>
            </configuration>
          </plugin>
        </plugins>
      </build>
    </profile>
  </profiles>

</project>
```