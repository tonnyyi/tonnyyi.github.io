---
title: MacOS 下编译 OpenJDK8
tags:
  - jdk
  - macos
categories:
  - java
date: 2019-11-11 14:15:18
---

## 环境信息
macOS Catalina 10.15

## 环境准备
- [Xcode 11](https://itunes.apple.com/us/app/xcode/id497799835?mt=12)
- xcode-select

    ```bash
    $ xcode-select -install
    ```
- [homebrew](https://brew.sh/)
- [XQuartz](https://www.xquartz.org/)
- freetype

    ```bash
    $ brew install freetype
    ```
    
- JDK 8

    ```bash
    $ java -version
    java version "1.8.0_191"
    Java(TM) SE Runtime Environment (build 1.8.0_191-b12)
    Java HotSpot(TM) 64-Bit Server VM (build 25.191-b12, mixed mode)
    ```

## 下载源码
下载源码有两种方式:

1. 从 openjdk 的 mercurial 仓库下载, 本地安装 mercurial 后执行如下命令:
    ```bash
    $ hg clone http://hg.openjdk.java.net/jdk8/jdk8 JDK8
    $ cd JDK8
    $ bash ./get_source.sh
    ```

1. 打开http://jdk.java.net/, 选择版本, 在新页面的**RI Source Code**区域, 点击**zip file**连接, 下载源码压缩包, 这种方式下载下来的源码版本是固定的. 

这里采用第二种方式, 并将下载下来的 zip 包解压到 `openjdk8` 目录

# 系统准备
- 先创建一个 `envsetup.sh`, 写入以下内容

    ```shell
    # 设定语言选项，必须设置
    export LANG=C
    # Mac平台，C编译器不再是GCC，是clang
    export CC=gcc
    # 跳过clang的一些严格的语法检查，不然会将N多的警告作为Error
    export COMPILER_WARNINGS_FATAL=false
    # 链接时使用的参数
    export LFLAGS='-Xlinker -lstdc++'
    # 是否使用clang
    export USE_CLANG=true
    # 使用64位数据模型
    export LP64=1
    # 告诉编译平台是64位，不然会按32位来编译
    export ARCH_DATA_MODEL=64
    # 允许自动下载依赖
    export ALLOW_DOWNLOADS=true
    # 并行编译的线程数，编译时间长，为了不影响其他工作，我选择为2
    export HOTSPOT_BUILD_JOBS=2
    # 是否跳过与先前版本的比较
    export SKIP_COMPARE_IMAGES=true
    # 是否使用预编译头文件，加快编译速度
    export USE_PRECOMPILED_HEADER=true
    # 是否使用增量编译
    export INCREMENTAL_BUILD=true
    # 编译内容
    export BUILD_LANGTOOLS=true
    export BUILD_JAXP=false
    export BUILD_JAXWS=false
    export BUILD_CORBA=false
    export BUILD_HOTSPOT=true
    export BUILD_JDK=true
    # 编译版本
    export SKIP_DEBUG_BUILD=true
    export SKIP_FASTDEBUG_BUILD=false
    export DEBUG_NAME=debug
    # 避开javaws和浏览器Java插件之类的部分的build
    export BUILD_DEPLOY=false
    export BUILD_INSTALL=false
    # 加上产生调试信息时需要的 objcopy
    export OBJCOPY=gobjcopy
    ```
    然后执行命令: `source envsetup.sh`

- Xcode 10 以后移除了 libstdc++ 的支持, 但 hotspot 编译时还需要依赖这个过期 5 年的库, 因此需要从 Xcode9 中拷贝出libstdc++和 c++包, 复制到响应位置. 也可以打开 [这个链接](https://github.com/tonnyyi/xcode-missing-libstdc-), clone 到本地，参考 `install.sh` 将文件链接或者复制到对应位置(**慎重直接执行**, 请一定事先核对路径是否正确!)

## configure
在 openjdk8 目录下执行以下命令

```bash
$ bash ./configure \
    --with-target-bits=64 \
    --with-debug-level=slowdebug \
    --enable-debug-symbols \
    ZIP_DEBUGINFO_FILES=0
```
### 错误1
```bash
configure: error: GCC compiler is required. Try setting --with-tools-dir.
```
编辑`common/autoconf/generated-configure.sh`, 搜索 `GCC compiler is required`, 在搜索结果的每一行加上`#`, 注释掉, 例如:

```bash
# as_fn_error $? "GCC compiler is required. Try setting --with-tools-dir." "$LINENO" 5
```

### 错误 2
```bash
checking if we should generate debug symbols... configure: error: Unable to find objcopy, cannot enable debug-symbols
```
因为没有正确执行上面的`envsetup.sh`文件, 解决办法:
```bash
$ source envsetup.sh
```

再次执行 `configure` 命令后, 输出如下成功信息
```bash
====================================================
A new configuration has been successfully created in
/Users/tonnyyi/workspace/sourceCode/openjdk8/build/macosx-x86_64-normal-server-slowdebug
using configure arguments '--with-target-bits=64 --with-debug-level=slowdebug --enable-debug-symbols ZIP_DEBUGINFO_FILES=0'.

Configuration summary:
* Debug level:    slowdebug
* JDK variant:    normal
* JVM variants:   server
* OpenJDK target: OS: macosx, CPU architecture: x86, address length: 64

Tools summary:
* Boot JDK:       java version "1.8.0_191" Java(TM) SE Runtime Environment (build 1.8.0_191-b12) Java HotSpot(TM) 64-Bit Server VM (build 25.191-b12, mixed mode)  (at /Library/Java/JavaVirtualMachines/jdk1.8.0_191.jdk/Contents/Home)
* C Compiler:      version  (at /usr/bin/gcc)
* C++ Compiler:    version  (at /usr/bin/g++)

Build performance summary:
* Cores to use:   4
* Memory limit:   16384 MB
* ccache status:  installed, but disabled (version older than 3.1.4)

Build performance tip: ccache gives a tremendous speedup for C++ recompilations.
You have ccache installed, but it is a version prior to 3.1.4. Try upgrading.
```

## make
执行如下命令:
```bash
$ make all LOG=debug  2>&1 | tee make_mac_x64.log
```

### 错误 1
```bash
if (Universe::narrow_oop_base() > 0) { // Implies UseCompressedOops.
```
编辑`hotspot/src/share/vm/opto/lcm.cpp`文件, 作如下调整
```c++
// if (Universe::narrow_oop_base() > 0) { // Implies UseCompressedOops.
if (Universe::narrow_oop_base() != NULL) { // Implies UseCompressedOops.
```

### 错误 2
```bash
assert(rng->Opcode() == Op_LoadRange || _igvn.type(rng)->is_int() >= 0, "must be");
```
编辑`hotspot/src/share/vm/opto/loopPredicate.cpp`文件, 做如下调整
```c++
//assert(rng->Opcode() == Op_LoadRange || _igvn.type(rng)->is_int() >= 0, "must be");
assert(rng->Opcode() == Op_LoadRange || _igvn.type(rng)->is_int()->_lo >= 0, "must be");
```

### 错误 3
```bash
if (base() > 0) {
```
编辑`hotspot/src/share/vm/runtime/virtualspace.cpp`, 做如下调整:
```c++
// if (base() > 0) {
if (base() != 0) {
```

### 错误 4
```bash
#import <JavaNativeFoundation/JavaNativeFoundation.h>
```
因为 Xcode 之前会安装类似 xxx-for-java-command-lines-tools 的框架包到 /System/Library/Frameworks, 而自从 macOS 10.14 开始, 这些框架包全部都被安装到了 /Library/Developer/CommandLineTools/SDKs/MacOSX10.1x.sdk, 首先执行如下命令行
```bash
$ find / -name "*JavaNativeFoundation.h*"
...
/Library/Developer/CommandLineTools/SDKs/MacOSX10.14.sdk/System/Library/Frameworks/JavaVM.framework/Versions/A/Frameworks/JavaNativeFoundation.framework/Versions/A/Headers/JavaNativeFoundation.h
/Library/Developer/CommandLineTools/SDKs/MacOSX10.15.sdk/System/Library/Frameworks/JavaVM.framework/Versions/A/Frameworks/JavaNativeFoundation.framework/Versions/A/Headers/JavaNativeFoundation.h
...
```
得到了真正的`JavaVM.framework`是被安装到了`/Library/Developer/CommandLineTools/SDKs/MacOSX10.14.sdk`目录下
编辑`hotspot/make/bsd/makefiles/saproc.make`文件, 作如下调整, 加上路径前缀`/Library/Developer/CommandLineTools/SDKs/MacOSX10.14.sdk`
```
# SALIBS = -g -framework Foundation -F/System/Library/Frameworks/JavaVM.framework/Frameworks -framework JavaNativeFoundation -framework Security -framework CoreFoundation
SALIBS = -g -framework Foundation -F/Library/Developer/CommandLineTools/SDKs/MacOSX10.14.sdk/System/Library/Frameworks/JavaVM.framework/Frameworks -framework JavaNativeFoundation -framework Security -framework CoreFoundation
```

```
# -I/System/Library/Frameworks/JavaVM.framework/Headers
-I/Library/Developer/CommandLineTools/SDKs/MacOSX10.14.sdk/System/Library/Frameworks/JavaVM.framework/Headers
```

### 错误 5
```
#import <CoreGraphics/CGBase.h>
```
这个问题和上面一样, 需要编辑两个文件
- `jdk/make/lib/PlatformLibraries.gmk`
- `jdk/make/lib/Awt2dLibraries.gmk`

将包含 `ApplicationServices.framework` 的路径前替换成
`/Library/Developer/CommandLineTools/SDKs/MacOSX10.14.sdk/System/Library/Frameworks/CoreGraphics.framework`

将包含 `JavaVM.framwork` 的路径替换成
`/Library/Developer/CommandLineTools/SDKs/MacOSX10.14.sdk/System/Library/Frameworks/JavaVM.framework/Frameworks`

### 错误 6
```
Undefined symbols for architecture x86_64:
  "_attachCurrentThread", referenced from:
      +[ThreadUtilities getJNIEnv] in ThreadUtilities.o
      +[ThreadUtilities getJNIEnvUncached] in ThreadUtilities.o
ld: symbol(s) not found for architecture x86_64
```
编辑`jdk/src/macosx/native/sun/osxapp/ThreadUtilities.m`, 作如下调整
```c++
// inline void attachCurrentThread(void** env) {
static inline void attachCurrentThread(void** env) {
```

### 编译完成
```bash
## Finished docs (build time 00:01:39)

----- Build times -------
Start 2019-11-11 15:47:26
End   2019-11-11 15:51:36
00:00:01 corba
00:00:40 demos
00:01:39 docs
00:00:00 hotspot
00:01:28 images
00:00:00 jaxp
00:00:00 jaxws
00:00:14 jdk
00:00:00 langtools
00:00:08 nashorn
00:04:10 TOTAL
-------------------------
Finished building OpenJDK for target 'all'
```

## 测试验证
执行如下命令
```bash
$ build/macosx-x86_64-normal-server-slowdebug/jdk/bin/java -version
```

### 错误 1
```bash
#
# A fatal error has been detected by the Java Runtime Environment:
#
#  SIGILL (0x4) at pc=0x000000010840f0e8, pid=36538, tid=9987
#
# JRE version: OpenJDK Runtime Environment (8.0) (build 1.8.0-internal-debug-tonnyyi_2019_11_11_14_52-b00)
# Java VM: OpenJDK 64-Bit Server VM (25.40-b25-debug mixed mode bsd-amd64 compressed oops)
# Problematic frame:
# V  [libjvm.dylib+0xa0f0e8]  PerfData::~PerfData()+0x8
#
# Failed to write core dump. Core dumps have been disabled. To enable core dumping, try "ulimit -c unlimited" before starting Java again
#
# An error report file with more information is saved as:
# /Users/tonnyyi/workspace/sourceCode/openjdk8/hs_err_pid36538.log
#
# If you would like to submit a bug report, please visit:
#   http://bugreport.java.com/bugreport/crash.jsp
#

[error occurred during error reporting , id 0x4]
```
编辑`hotspot/src/share/vm/runtime/perfMemory.cpp`的`perfMemory_exit()`函数, 作如下调整:
```c++
  if (!StatSampler::is_active())
    PerfDataManager::destroy();
    
  if (!StatSampler::is_active())
    // PerfDataManager::destroy();
```

然后再次重新编译
```bash
$ make all LOG=debug  2>&1 | tee make_mac_x64.log
```

但是这样处理后，使用jstat监控JVM时可能会导致 **内存泄露**

再次验证
```bash
$ build/macosx-x86_64-normal-server-slowdebug/jdk/bin/java -version
openjdk version "1.8.0-internal-debug"
OpenJDK Runtime Environment (build 1.8.0-internal-debug-tonnyyi_2019_11_11_14_52-b00)
OpenJDK 64-Bit Server VM (build 25.40-b25-debug, mixed mode)
```

### 参考资料
https://www.jianshu.com/p/ee7e9176632c
https://imkiva.com/2018/02/24/building-openjdk8-on-macos/
https://juejin.im/post/5a6d7d106fb9a01ca47abd8b
http://menzhongxin.com/2017/04/27/MAC%E4%B8%8B%E7%BC%96%E8%AF%91openJDK/
https://iyichen.xyz/2019/10/mac-compile-openjdk/