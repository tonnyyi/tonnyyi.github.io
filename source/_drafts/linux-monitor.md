---
title: linux下监控工具
tags:
  - null
categories:
  - null
date: 2019-01-10 08:36:03
---
# gbd 调试利器
GDB是一个由GNU开源组织发布的、UNIX/LINUX操作系统下的、基于命令行的、功能强大的程序调试工具。 对于一名Linux下工作的c++程序员，gdb是必不可少的工具
### 启动gdb
对C/C++程序的调试，需要在编译前就加上-g选项:
```bash
$ g++ -g hello.cpp -o hello
```

调试可执行文件:
```bash
$ gdb <program>
```
program也就是你的执行文件，一般在当前目录下。

调试core文件(core是程序非法执行后core dump后产生的文件):
```bash
$ gdb <program> <core dump file>
$ gdb program core.11127
```

调试服务程序:
```bash
$ gdb <program> <PID>
$ gdb hello 11127
```

如果你的程序是一个服务程序，那么你可以指定这个服务程序运行时的进程ID。gdb会自动attach上去，并调试他。program应该在PATH环境变量中搜索得到。

### gdb交互命令
启动gdb后，进入到交互模式，通过以下命令完成对程序的调试；注意高频使用的命令一般都会有缩写，熟练使用这些缩写命令能提高调试的效率；

#### 运行
* run：简记为 r ，其作用是运行程序，当遇到断点后，程序会在断点处停止运行，等待用户输入下一步的命令。
* continue （简写c ）：继续执行，到下一个断点处（或运行结束）
* next：（简写 n），单步跟踪程序，当遇到函数调用时，也不进入此函数体；此命令同 * * step 的主要区别是，step 遇到用户自定义的函数，将步进到函数中去运行，而 next 则直接调用函数，不会进入到函数体内。
* step （简写s）：单步调试如果有函数调用，则进入函数；与命令n不同，n是不进入调用的函数的
* until：当你厌倦了在一个循环体内单步跟踪时，这个命令可以运行程序直到退出循环体。
* until+行号： 运行至某行，不仅仅用来跳出循环
* finish： 运行程序，直到当前函数完成返回，并打印函数返回时的堆栈地址和返回值及参数值等信息。
* call 函数(参数)：调用程序中可见的函数，并传递“参数”，如：call gdb_test(55)
* quit：简记为 q ，退出gdb

#### 设置断点
* break n （简写b n）:在第n行处设置断点
（可以带上代码路径和代码名称： b OAGUPDATE.cpp:578）
* b fn1 if a＞b：条件断点设置
* break func（break缩写为b）：在函数func()的入口处设置断点，如：break cb_button
* delete 断点号n：删除第n个断点
* disable 断点号n：暂停第n个断点
* enable 断点号n：开启第n个断点
* clear 行号n：清除第n行的断点
* info b （info breakpoints） ：显示当前程序的断点设置情况
* delete breakpoints：清除所有断点：

#### 查看源代码
* list ：简记为 l ，其作用就是列出程序的源代码，默认每次显示10行。
* list 行号：将显示当前文件以“行号”为中心的前后10行代码，如：list 12
* list 函数名：将显示“函数名”所在函数的源代码，如：list main
* list ：不带参数，将接着上一次 list 命令的，输出下边的内容。
* 
#### 打印表达式
* print 表达式：简记为 p ，其中“表达式”可以是任何当前正在被测试程序的有效表达式，比如当前正在调试C语言的程序，那么“表达式”可以是任何C语言的有效表达式，包括数字，变量甚至是函数调用。
* print a：将显示整数 a 的值
* print ++a：将把 a 中的值加1,并显示出来
* print name：将显示字符串 name 的值
* print gdb_test(22)：将以整数22作为参数调用 gdb_test() 函数
* print gdb_test(a)：将以变量 a 作为参数调用 gdb_test() 函数
* display 表达式：在单步运行时将非常有用，使用display命令设置一个表达式后，它将在每次单步进行指令后，紧接着输出被设置的表达式及值。如： display a
* watch 表达式：设置一个监视点，一旦被监视的“表达式”的值改变，gdb将强行终止正在被调试的程序。如： watch a
* whatis ：查询变量或函数
* info function： 查询函数
* 扩展info locals： 显示当前堆栈页的所有变量

#### 查询运行信息
* where/bt ：当前运行的堆栈列表；
* bt backtrace 显示当前调用堆栈
* up/down 改变堆栈显示的深度
* set args 参数:指定运行时的参数
* show args：查看设置好的参数
* info program： 来查看程序的是否在运行，进程号，被暂停的原因。

#### 分割窗口
* layout：用于分割窗口，可以一边查看代码，一边测试：
* layout src：显示源代码窗口
* layout asm：显示反汇编窗口
* layout regs：显示源代码/反汇编和CPU寄存器窗口
* layout split：显示源代码和反汇编窗口
* Ctrl + L：刷新窗口

# ldd 查看程序依赖库
用来查看程式运行所需的共享库,常用来解决程式因缺少某个库文件而不能运行的一些问题。
查看test程序运行所依赖的库:
```bash
/opt/app/todeav1/test$ldd test
libstdc++.so.6 => /usr/lib64/libstdc++.so.6 (0x00000039a7e00000)
libm.so.6 => /lib64/libm.so.6 (0x0000003996400000)
libgcc_s.so.1 => /lib64/libgcc_s.so.1 (0x00000039a5600000)
libc.so.6 => /lib64/libc.so.6 (0x0000003995800000)
/lib64/ld-linux-x86-64.so.2 (0x0000003995400000)
```
* 第一列：程序需要依赖什么库
* 第二列: 系统提供的与程序需要的库所对应的库
* 第三列：库加载的开始地址

通过上面的信息，我们可以得到以下几个信息：
通过对比第一列和第二列，我们可以分析程序需要依赖的库和系统实际提供的，是否相匹配
通过观察第三列，我们可以知道在当前的库中的符号在对应的进程的地址空间中的开始位置
如果依赖的某个库找不到，通过这个命令可以迅速定位问题所在

# lsof 一切皆文件
lsof（list open files）是一个查看当前系统文件的工具。在linux环境下，任何事物都以文件的形式存在，通过文件不仅仅可以访问常规数据，还可以访问网络连接和硬件。如传输控制协议 (TCP) 和用户数据报协议 (UDP) 套接字等，系统在后台都为该应用程序分配了一个文件描述符，该文件描述符提供了大量关于这个应用程序本身的信息。

lsof打开的文件可以是：

* 普通文件
* 目录
* 网络文件系统的文件
* 字符或设备文件
* (函数)共享库
* 管道，命名管道
* 符号链接
* 网络文件（例如：NFS file、网络socket，unix域名socket）
* 还有其它类型的文件，等等

## 命令参数
* -a 列出打开文件存在的进程
* -c<进程名> 列出指定进程所打开的文件
* -g 列出GID号进程详情
* -d<文件号> 列出占用该文件号的进程
* +d<目录> 列出目录下被打开的文件
* +D<目录> 递归列出目录下被打开的文件
* -n<目录> 列出使用NFS的文件
* -i<条件> 列出符合条件的进程。（4、6、协议、:端口、 @ip ）
* -p<进程号> 列出指定进程号所打开的文件
* -u 列出UID号进程详情
* -h 显示帮助信息
* -v 显示版本信息

## 使用实例
### 无任何参数
```bash
$ lsof | more
COMMAND      PID    TID           USER   FD      TYPE             DEVICE  SIZE/OFF       NODE NAME
systemd        1                  root  cwd       DIR                8,3       268         64 /
systemd        1                  root  rtd       DIR                8,3       268         64 /
systemd        1                  root  txt       REG                8,3   1482128  134504040 /usr/lib/systemd/systemd
systemd        1                  root  mem       REG                8,3     20040   67110605 /usr/lib64/libuuid.so.1.3.0
systemd        1                  root  mem       REG                8,3    256960   67577906 /usr/lib64/libblkid.so.1.1.0
systemd        1                  root  mem       REG                8,3     90248   68646954 /usr/lib64/libz.so.1.2.7
systemd        1                  root  mem       REG                8,3    157424   67110594 /usr/lib64/liblzma.so.5.2.2
systemd        1                  root  mem       REG                8,3     23968   67111001 /usr/lib64/libcap-ng.so.0.0.0
systemd        1                  root  mem       REG                8,3     19888   67110989 /usr/lib64/libattr.so.1.1.0
systemd        1                  root  mem       REG                8,3     19776   67110432 /usr/lib64/libdl-2.17.so
systemd        1                  root  DEL       REG                8,3             67110572 /usr/lib64/libpcre.so.1.2.0;5c10c78e
systemd        1                  root  mem       REG                8,3   2173512   67110426 /usr/lib64/libc-2.17.so
...
```

lsof输出各列信息的意义如下：
* COMMAND：进程的名称
* PID：进程标识符
* PPID：父进程标识符（需要指定-R参数）
* USER：进程所有者
* PGID：进程所属组
* FD：文件描述符，应用程序通过文件描述符识别该文件。如cwd、txt等:
    - cwd：表示current work dirctory，即：应用程序的当前工作目录，这是该应用程序启动的目录，除非它本身对这个目录进行更改
    - txt ：该类型的文件是程序代码，如应用程序二进制文件本身或共享库，如上列表中显示的 /sbin/init 程序
    - lnn：library references (AIX);
    - er：FD information error (see NAME column);
    - jld：jail directory (FreeBSD);
    - ltx：shared library text (code and data);
    - mxx ：hex memory-mapped type number xx.
    - m86：DOS Merge mapped file;
    - mem：memory-mapped file;
    - mmap：memory-mapped device;
    - pd：parent directory;
    - rtd：root directory;
    - tr：kernel trace file (OpenBSD);
    - v86  VP/ix mapped file;
    - 0：表示标准输入
    - 1：表示标准输出
    - 2：表示标准错误
* TYPE：文件类型，如DIR、REG等，常见的文件类型:
    - DIR：表示目录
    - CHR：表示字符类型
    - BLK：块设备类型
    - UNIX： UNIX 域套接字
    - FIFO：先进先出 (FIFO) 队列
    - IPv4：网际协议 (IP) 套接字
* DEVICE：指定磁盘的名称
* SIZE：文件的大小
* NODE：索引节点（文件在磁盘上的标识）
* NAME：打开文件的确切名称

### 查找某个文件相关的进程
```bash
$ lsof /bin/bash
```

### 列出某个用户打开的文件信息
```bash
$ lsof -u username
```

### 列出某个程序进程所打开的文件信息
```bash
$ lsof -c mysql
```

### 列出所有网络连接信息
```bash
$ lsof -i
# 只列出所有tcp连接
$ lsof -i tcp
```


### 列出谁在使用某个端口
```bash
$ lsof -i :3306
```

### 列出某个用户的所有活跃的网络端口
```bash
$ lsof -a -u test -i
```

### 列出被进程号为1234的进程所打开的所有IPV4 network files
```bash
$ lsof -i 4 -a -p 1234
```

### 列出目前连接主机nf5260i5-td上端口为：20，21，80相关的所有文件信息，且每隔3秒重复执行
```bash
$ lsof -i @nf5260i5-td:20,21,80 -r 3
```

# ps 进程查看器
Linux中的ps命令是Process Status的缩写。ps命令用来列出系统中当前运行的那些进程。ps命令列出的是当前那些进程的快照，就是执行ps命令的那个时刻的那些进程，如果想要动态的显示进程信息，就可以使用top命令。

linux上进程有5种状态:
* 运行(正在运行或在运行队列中等待)
* 中断(休眠中, 受阻, 在等待某个条件的形成或接受到信号)
* 不可中断(收到信号不唤醒和不可运行, 进程必须等待直到有中断发生)
* 僵死(进程已终止, 但进程描述符存在, 直到父进程调用wait4()系统调用后释放)
* 停止(进程收到SIGSTOP, SIGTSTP, SIGTTIN, SIGTTOU信号后停止运行运行)

ps工具标识进程的5种状态码:
* D 不可中断 uninterruptible sleep (usually IO)
* R 运行 runnable (on run queue)
* S 中断 sleeping
* T 停止 traced or stopped
* Z 僵死 a defunct (”zombie”) process

## 命令参数
* a 显示所有进程
* -a 显示同一终端下的所有程序
* -A 显示所有进程
* c 显示进程的真实名称
* -N 反向选择
* -e 等于“-A”
* e 显示环境变量
* f 显示程序间的关系
* -H 显示树状结构
* r 显示当前终端的进程
* T 显示当前终端的所有程序
* u 指定用户的所有进程
* -au 显示较详细的资讯
* -aux 显示所有包含其他使用者的行程
* -C<命令> 列出指定命令的状况
* –lines<行数> 每页显示的行数
* –width<字符数> 每页显示的字符数
* –help 显示帮助信息
* –version 显示版本显示

## 输出列的含义
* F 代表这个程序的旗标 (flag)， 4 代表使用者为 super user
* S 代表这个程序的状态 (STAT)，关于各 STAT 的意义将在内文介绍
* UID 程序被该 UID 所拥有
* PID 进程的ID
* PPID 则是其上级父程序的ID
* C CPU 使用的资源百分比
* PRI 这个是 Priority (优先执行序) 的缩写，详细后面介绍
* NI 这个是 Nice 值，在下一小节我们会持续介绍
* ADDR 这个是 kernel function，指出该程序在内存的那个部分。如果是个 running的程序，一般就是 “-“
* SZ 使用掉的内存大小
* WCHAN 目前这个程序是否正在运作当中，若为 - 表示正在运作
* TTY 登入者的终端机位置
* TIME 使用掉的 CPU 时间。
* CMD 所下达的指令为何

## 使用实例
### 显示所有进程信息，连同命令行
```bash
$ ps -ef
```

### 显示指定用户信息
```bash
$ ps -u root
```

### ps 与grep 组合使用，查找特定进程
```bash
$ ps -ef | grep mysql
```

### 列出目前所有的正在内存中的程序
```bash
$ ps aux
```

# pstack 跟踪进程栈
此命令可显示每个进程的栈跟踪。pstack 命令必须由相应进程的属主或 root 运行。可以使用 pstack 来确定进程挂起的位置。此命令允许使用的唯一选项是要检查的进程的 PID。
这个命令在排查进程问题时非常有用，比如我们发现一个服务一直处于work状态（如假死状态，好似死循环），使用这个命令就能轻松定位问题所在；可以在一段时间内，多执行几次pstack，若发现代码栈总是停在同一个位置，那个位置就需要重点关注，很可能就是出问题的地方.

查看bash程序进程栈:
```bash
[root@AI125 ~]# ps -fe| grep bash
root      77957  77952  0 15:42 pts/0    00:00:00 -bash
root      79729  79725  0 15:55 pts/1    00:00:00 -bash
root      80309  79729  0 15:59 pts/1    00:00:00 grep --color=auto bash
[root@AI125 ~]# pstack 77957
#0  0x00007fc91859cc70 in __read_nocancel () from /lib64/libc.so.6
#1  0x000000000049d959 in rl_getc ()
#2  0x000000000049e184 in rl_read_key ()
#3  0x000000000048a0e3 in readline_internal_char ()
#4  0x000000000048a755 in readline ()
#5  0x000000000041e66a in yy_readline_get ()
#6  0x00000000004205d2 in shell_getc ()
#7  0x0000000000422f9a in read_token.constprop.6 ()
#8  0x0000000000426369 in yyparse ()
#9  0x000000000041df4a in parse_command ()
#10 0x000000000041e00c in read_command ()
#11 0x000000000041e20c in reader_loop ()
#12 0x000000000041c8ee in main ()
```

# strace 跟踪进程中的系统调用
strace常用来跟踪进程执行时的系统调用和所接收的信号。 在Linux世界，进程不能直接访问硬件设备，当进程需要访问硬件设备(比如读取磁盘文件，接收网络数据等等)时，必须由用户态模式切换至内核态模式，通过系统调用访问硬件设备。strace可以跟踪到一个进程产生的系统调用,包括参数，返回值，执行消耗的时间。

## 输出参数含义
每一行都是一条系统调用，等号左边是系统调用的函数名及其参数，右边是该调用的返回值。 strace 显示这些调用的参数并返回符号形式的值。strace 从内核接收信息，而且不需要以任何特殊的方式来构建内核。
```bash
[root@AI125 ~]# strace cat /dev/null
execve("/usr/bin/cat", ["cat", "/dev/null"], [/* 29 vars */]) = 0
brk(0)                                  = 0xacc000
mmap(NULL, 4096, PROT_READ|PROT_WRITE, MAP_PRIVATE|MAP_ANONYMOUS, -1, 0) = 0x7f13bea47000
access("/etc/ld.so.preload", R_OK)      = -1 ENOENT (No such file or directory)
open("/etc/ld.so.cache", O_RDONLY|O_CLOEXEC) = 3
fstat(3, {st_mode=S_IFREG|0644, st_size=43692, ...}) = 0
mmap(NULL, 43692, PROT_READ, MAP_PRIVATE, 3, 0) = 0x7f13bea3c000
...
```

## 参数
* -c 统计每一系统调用的所执行的时间,次数和出错的次数等.
* -d 输出strace关于标准错误的调试信息.
* -f 跟踪由fork调用所产生的子进程.
* -ff 如果提供-o filename,则所有进程的跟踪结果输出到相应的filename.pid中,pid是各进程的进程号.
* -F 尝试跟踪vfork调用.在-f时,vfork不被跟踪.
* -h 输出简要的帮助信息.
* -i 输出系统调用的入口指针.
* -q 禁止输出关于脱离的消息.
* -r 打印出相对时间关于,,每一个系统调用.
* -t 在输出中的每一行前加上时间信息.
* -tt 在输出中的每一行前加上时间信息,微秒级.
* -ttt 微秒级输出,以秒了表示时间.
* -T 显示每一调用所耗的时间.
* -v 输出所有的系统调用.一些调用关于环境变量,状态,输入输出等调用由于使用频繁,默认不输出.
* -V 输出strace的版本信息.
* -x 以十六进制形式输出非标准字符串
* -xx 所有字符串以十六进制形式输出.
* -a column 设置返回值的输出位置.默认 为40.
* -e expr 指定一个表达式,用来控制如何跟踪.格式如下:
    [qualifier=][!]value1[,value2]...
    qualifier只能是 trace,abbrev,verbose,raw,signal,read,write其中之一.value是用来限定的符号或数字.默认的 qualifier是 trace.感叹号是否定符号.例如:
    * -eopen等价于 -e trace=open,表示只跟踪open调用.而-etrace!=open表示跟踪除了open以外的其他调用.有两个特殊的符号 all 和 none. 注意有些shell使用!来执行历史记录里的命令,所以要使用\\.
    * -e trace=set 只跟踪指定的系统 调用.例如:-e trace=open,close,rean,write表示只跟踪这四个系统调用.默认的为set=all.
    * -e trace=file 只跟踪有关文件操作的系统调用.
    * -e trace=process 只跟踪有关进程控制的系统调用.
    * -e trace=network 跟踪与网络有关的所有系统调用.
    * -e strace=signal 跟踪所有与系统信号有关的 系统调用
    * -e trace=ipc 跟踪所有与进程通讯有关的系统调用
    * -e abbrev=set 设定 strace输出的系统调用的结果集.-v 等与 abbrev=none.默认为abbrev=all.
    * -e raw=set 将指 定的系统调用的参数以十六进制显示.
    * -e signal=set 指定跟踪的系统信号.默认为all.如 signal=!SIGIO(或者signal=!io),表示不跟踪SIGIO信号.
    * -e read=set 输出从指定文件中读出 的数据.例如: -e read=3,5
    * -e write=set 输出写入到指定文件中的数据.
* -o filename 将strace的输出写入文件filename
* -p pid 跟踪指定的进程pid.
* -s strsize 指定输出的字符串的最大长度.默认为32.文件名一直全部输出.
* -u username 以username 的UID和GID执行被跟踪的命令

## 命令实例
### 跟踪可执行程序
```bash
$ strace -f -F -o ~/straceout.txt myserver
```
-f -F选项告诉strace同时跟踪fork和vfork出来的进程，-o选项把所有strace输出写到~/straceout.txt里 面，myserver是要启动和调试的程序。

### 跟踪可执行程序
```bash
$ strace -o output.txt -T -tt -e trace=all -p 28979
```
跟踪28979进程的所有系统调用（-e trace=all），并统计系统调用的花费时间，以及开始时间（并以可视化的时分秒格式显示），最后将记录结果存在output.txt文件里面。

# ipcs 查询进程间通信状态
ipcs是Linux下显示进程间通信设施状态的工具。可以显示消息队列、共享内存和信号量的信息。对于程序员非常有用，普通的系统管理员一般用不到此指令。

## 综合应用
### 查询user1用户环境上是否存在积Queue现象
```bash
# 查询队列Queue
$ ipcs -q

------ Message Queues --------
key        msqid      owner      perms      used-bytes   messages
0x49060005 58261504   user1    660        0            0
0x4f060005 58294273   user1    660        0            0
...

# 找出第6列大于0的服务
$ ipcs -q |grep user1 |awk '{if($5>0) print $0}'
0x00000000 1071579324 user1       644        1954530      4826
0x00000000 1071644862 user1       644        1961820      4844
0x00000000 1071677631 user1       644        1944810      4802
0x00000000 1071710400 user1       644        1961820      4844
```

# top linux下的任务管理器
top命令是Linux下常用的性能分析工具，能够实时显示系统中各个进程的资源占用状况，类似于Windows的任务管理器。top是一个动态显示过程,即可以通过用户按键来不断刷新当前状态.如果在前台执行该命令,它将独占前台,直到用户终止该程序为止.比较准确的说,top命令提供了实时的对系统处理器的状态监视.它将显示系统中CPU最“敏感”的任务列表.该命令可以按CPU使用.内存使用和执行时间对任务进行排序；而且该命令的很多特性都可以通过交互式命令或者在个人定制文件中进行设定。
```bash
[root@AI125 ~]# top
top - 09:14:56 up 264 days, 20:56,  1 user,  load average: 0.02, 0.04, 0.00
Tasks:  87 total,   1 running,  86 sleeping,   0 stopped,   0 zombie
Cpu(s):  0.0%us,  0.2%sy,  0.0%ni, 99.7%id,  0.0%wa,  0.0%hi,  0.0%si,  0.2%st
Mem:    377672k total,   322332k used,    55340k free,    32592k buffers
Swap:   397308k total,    67192k used,   330116k free,    71900k cached
PID USER      PR  NI  VIRT  RES  SHR S %CPU %MEM    TIME+  COMMAND
1 root      20   0  2856  656  388 S  0.0  0.2   0:49.40 init
2 root      20   0     0    0    0 S  0.0  0.0   0:00.00 kthreadd
3 root      20   0     0    0    0 S  0.0  0.0   7:15.20 ksoftirqd/0
4 root      RT   0     0    0    0 S  0.0  0.0   0:00.00 migration/0
...
```

* **第一行**
    * 09:14:56 ： 系统当前时间
    * 264 days, 20:56 ： 系统开机到现在经过了多少时间
    * 1 users ： 当前2用户在线
    * load average: 0.02, 0.04, 0.00： 系统1分钟、5分钟、15分钟的CPU负载信

* **第二行**
    * Tasks：任务;
    * 87 total：很好理解，就是当前有87个任务，也就是87个进程。
    * 1 running：1个进程正在运行
    * 86 sleeping：86个进程睡眠
    * 0 stopped：停止的进程数
    * 0 zombie：僵死的进程数
    
* **第三行**
    * Cpu(s)：表示这一行显示CPU总体信息
    * 0.0%us：用户态进程占用CPU时间百分比，不包含renice值为负的任务占用的CPU的时间。
    * 0.7%sy：内核占用CPU时间百分比
    * 0.0%ni：改变过优先级的进程占用CPU的百分比
    * 99.3%id：空闲CPU时间百分比
    * 0.0%wa：等待I/O的CPU时间百分比
    * 0.0%hi：CPU硬中断时间百分比
    * 0.0%si：CPU软中断时间百分比
    * 注：这里显示数据是所有cpu的平均值，如果想看每一个cpu的处理情况，按1即可；折叠，再次按1；

* **第四行**
    * Mem：内存的意思
    * 8175320kk total：物理内存总量
    * 8058868k used：使用的物理内存量
    * 116452k free：空闲的物理内存量
    * 283084k buffers：用作内核缓存的物理内存量

* **第五行**
    * Swap：交换空间
    * 6881272k total：交换区总量
    * 4010444k used：使用的交换区量
    * 2870828k free：空闲的交换区量
    * 4336992k cached：缓冲交换区总量
    
* **进程信息** 
    * PID：进程的ID
    * USER：进程所有者
    * PR：进程的优先级别，越小越优先被执行
    * NInice：值
    * VIRT：进程占用的虚拟内存
    * RES：进程占用的物理内存
    * SHR：进程使用的共享内存
    * S：进程的状态。S表示休眠，R表示正在运行，Z表示僵死状态，N表示该进程优先值为负数
    * %CPU：进程占用CPU的使用率
    * %MEM：进程使用的物理内存和总内存的百分比
    * TIME+：该进程启动后占用的总的CPU时间，即占用CPU使用时间的累加值。
    * COMMAND：进程启动命令名称

## top命令交互操作指令
* q：退出top命令
* <Space>：立即刷新
* s：设置刷新时间间隔
* c：显示命令完全模式
* t:：显示或隐藏进程和CPU状态信息
* m：显示或隐藏内存状态信息
* l：显示或隐藏uptime信息
* f：增加或减少进程显示标志
* S：累计模式，会把已完成或退出的子进程占用的CPU时间累计到父进程的MITE+
* P：按%CPU使用率排行
* T：按MITE+排行
* M：按%MEM排行
* u：指定显示用户进程
* r：修改进程renice值
* kkill：进程
* i：只显示正在运行的进程
* W：保存对top的设置到文件^/.toprc，下次启动将自动调用toprc文件的设置。
* h：帮助命令。
* q：退出

使用频率最高的是P、T、M，因为通常使用top，我们就想看看是哪些进程最耗cpu资源、占用的内存最多； 注：通过”shift + >”或”shift + <”可以向右或左改变排序列

## 使用实例
### 多核CPU监控
在top基本视图中，按键盘数字“1”，可监控每个逻辑CPU的状况；
```bash
[root@AI125 ~]# top
top - 16:24:16 up 73 days,  2:35,  2 users,  load average: 0.02, 0.09, 0.13
Tasks: 196 total,   1 running, 195 sleeping,   0 stopped,   0 zombie
%Cpu0  :  0.7 us,  0.0 sy,  0.0 ni, 99.3 id,  0.0 wa,  0.0 hi,  0.0 si,  0.0 st
%Cpu1  :  1.0 us,  1.0 sy,  0.0 ni, 98.0 id,  0.0 wa,  0.0 hi,  0.0 si,  0.0 st
%Cpu2  :  0.7 us,  0.0 sy,  0.0 ni, 99.3 id,  0.0 wa,  0.0 hi,  0.0 si,  0.0 st
%Cpu3  :  0.7 us,  0.0 sy,  0.0 ni, 99.3 id,  0.0 wa,  0.0 hi,  0.0 si,  0.0 st
%Cpu4  :  0.7 us,  0.7 sy,  0.0 ni, 98.7 id,  0.0 wa,  0.0 hi,  0.0 si,  0.0 st
%Cpu5  :  1.0 us,  0.3 sy,  0.0 ni, 98.7 id,  0.0 wa,  0.0 hi,  0.0 si,  0.0 st
%Cpu6  :  1.0 us,  0.3 sy,  0.0 ni, 98.7 id,  0.0 wa,  0.0 hi,  0.0 si,  0.0 st
%Cpu7  :  1.7 us,  0.7 sy,  0.0 ni, 97.7 id,  0.0 wa,  0.0 hi,  0.0 si,  0.0 st
KiB Mem : 24508288 total,  8633108 free,  9300772 used,  6574408 buff/cache
KiB Swap:        0 total,        0 free,        0 used. 13057912 avail Mem
```

### 高亮显示当前运行进程
在top基本视图中,按键盘“b”（打开/关闭加亮效果）；

### 显示完整的程序命令
```bash
[root@AI125 ~]# top -c
Tasks: 196 total,   1 running, 195 sleeping,   0 stopped,   0 zombie
%Cpu(s):  1.1 us,  0.7 sy,  0.0 ni, 98.1 id,  0.0 wa,  0.0 hi,  0.0 si,  0.0 st
KiB Mem : 24508288 total,  8632296 free,  9301368 used,  6574624 buff/cache
KiB Swap:        0 total,        0 free,        0 used. 13057120 avail Mem

   PID USER      PR  NI    VIRT    RES    SHR S  %CPU %MEM     TIME+ COMMAND
 99731 root      20   0 11.864g 1.800g  19512 S   4.0  7.7 438:50.22 java -jar ailegal-case.jar
   928 root      20   0 1074876  96480  34928 S   2.6  0.4   1269:07 /usr/bin/dockerd
 99453 root      20   0 11.601g 1.568g  14336 S   1.3  6.7 167:22.31 java -jar ailegal-center.jar
  2893 root      20   0   45240  21944  12528 S   0.7  0.1 439:25.27 /usr/local/bin/kube-proxy --config=/var/lib/kube-proxy/config.conf
  ...
```

### 显示指定的进程信息
```bash
[root@AI125 ~]# top -p 99502
top - 16:27:22 up 73 days,  2:38,  2 users,  load average: 0.08, 0.10, 0.13
Tasks:   1 total,   0 running,   1 sleeping,   0 stopped,   0 zombie
%Cpu(s):  0.4 us,  0.3 sy,  0.0 ni, 99.3 id,  0.0 wa,  0.0 hi,  0.0 si,  0.0 st
KiB Mem : 24508288 total,  8631532 free,  9302276 used,  6574480 buff/cache
KiB Swap:        0 total,        0 free,        0 used. 13056384 avail Mem

   PID USER      PR  NI    VIRT    RES    SHR S  %CPU %MEM     TIME+ COMMAND
 99502 root      20   0 11.762g 2.325g  14076 S   0.3  9.9  63:01.91 java
```

# free 查询可用内存
```bash
[root@AI125 ~]# free
              total        used        free      shared  buff/cache   available
Mem:       24508288     9301828     8631916     1156080     6574544    13056784
Swap:             0           0           0
```
free的输出一共三行，第三行为交换区的信息，分别是交换的总量（total），使用量（used）和有多少空闲的交换区（free），这个比较清楚，不说太多。

## 使用实例
### 让输出结果可读
```bash
[root@ailegal-ecs-no ~]# free -h
              total        used        free      shared  buff/cache   available
Mem:            31G        4.4G         25G         80M        1.6G         26G
Swap:            0B          0B          0B
```

# vmstat 监视内存使用情况
```shell
[root@AI125 ~]# vmstat 5 5
procs -----------memory---------- ---swap-- -----io---- -system-- ------cpu-----
 r  b   swpd   free   buff  cache   si   so    bi    bo   in   cs us sy id wa st
 1  0      0 10668404    884 6477940    0    0     0     5    0    1  1  0 99  0  0
 0  0      0 10668212    884 6477900    0    0     0    12 2770 4923  1  0 99  0  0
 0  0      0 10668304    884 6477900    0    0     0    46 2732 4886  0  0 99  0  0
 0  0      0 10668336    884 6477908    0    0     0    45 2614 4708  0  0 99  0  0
 0  0      0 10668368    884 6477920    0    0     0    19 2768 4816  1  0 99  0  0
```
## 字段说明
* Procs（进程）:
    - r: 运行队列中进程数量
    - b: 等待IO的进程数量
* Memory（内存）:
    - swpd: 使用虚拟内存大小
    - free: 可用内存大小
    - buff: 用作缓冲的内存大小
    - cache: 用作缓存的内存大小
* Swap:
    - si: 每秒从交换区写到内存的大小
    - so: 每秒写入交换区的内存大小
    - IO：（现在的Linux版本块的大小为1024bytes）
    - bi: 每秒读取的块数
    - bo: 每秒写入的块数
* system：
    - in: 每秒中断数，包括时钟中断
    - cs: 每秒上下文切换数
* CPU（以百分比表示）
    - us: 用户进程执行时间(user time)
    - sy: 系统进程执行时间(system time)
    - id: 空闲时间(包括IO等待时间)
    - wa: 等待IO时间

# iostat 监视I/O子系统
```shell
[root@AI125 ~]# iostat
Linux 3.10.0-514.el7.x86_64 (AI125) 	01/29/2019 	_x86_64_	(8 CPU)

avg-cpu:  %user   %nice %system %iowait  %steal   %idle
           0.85    0.00    0.36    0.02    0.00   98.77

Device:            tps    kB_read/s    kB_wrtn/s    kB_read    kB_wrtn
sda               1.38         0.46        41.82    2327063  213314535
```

**cpu属性值说明**
* %user：CPU处在用户模式下的时间百分比。
* %nice：CPU处在带NICE值的用户模式下的时间百分比。
* %system：CPU处在系统模式下的时间百分比。
* %iowait：CPU等待输入输出完成时间的百分比。
* %steal：管理程序维护另一个虚拟处理器时，虚拟CPU的无意识等待时间百分比。
* %idle：CPU空闲时间百分比。

> 注：如果%iowait的值过高，表示硬盘存在I/O瓶颈，%idle值高，表示CPU较空闲，如果%idle值高但系统响应慢时，有可能是CPU等待分配内存，此时应加大内存容量。%idle值如果持续低于10，那么系统的CPU处理能力相对较低，表明系统中最需要解决的资源是CPU。

**disk属性值说明：**
* tps：该设备每秒的传输次数（Indicate the number of transfers per second that were issued to the device.）。“一次传输”意思是“一次I/O请求”。多个逻辑请求可能会被合并为“一次I/O请求”。“一次传输”请求的大小是未知的。
* kB_read/s：每秒从设备（drive expressed）读取的数据量；
* kB_wrtn/s：每秒向设备（drive expressed）写入的数据量；
* kB_read：读取的总数据量；
* kB_wrtn：写入的总数量数据量；

```shell
# 每隔2秒刷新显示，且显示3次
$ iostat 2 3

# 查看TPS和吞吐量
$ iostat -d -k 1 1
[root@ailegal-ecs-no ~]# iostat -d -k 1 1
Linux 3.10.0-693.el7.x86_64 (ailegal-ecs-no.04.novalocal) 	01/29/2019 	_x86_64_	(8 CPU)

Device:            tps    kB_read/s    kB_wrtn/s    kB_read    kB_wrtn
vda               0.94         0.05        10.97     201566   40724977
scd0              0.00         0.00         0.00        174          0
vdb               0.00         0.00         0.03      13516     105310
```

## 查看设备使用率(%util)和响应时间(await)
```shell
[root@ailegal-ecs-no ~]# iostat -d -x -k 1 1
Linux 3.10.0-693.el7.x86_64 (ailegal-ecs-no.04.novalocal) 	01/29/2019 	_x86_64_	(8 CPU)

Device:         rrqm/s   wrqm/s     r/s     w/s    rkB/s    wkB/s avgrq-sz avgqu-sz   await r_await w_await  svctm  %util
vda               0.00     0.05    0.00    0.94     0.05    10.97    23.51     0.01    7.01   19.55    6.99   3.20   0.30
scd0              0.00     0.00    0.00    0.00     0.00     0.00     6.82     0.00    1.18    1.18    0.00   1.18   0.00
vdb               0.00     0.00    0.00    0.00     0.00     0.03   258.60     0.00    5.85    2.06    9.85   5.64   0.00
```

* rrqm/s: 每秒进行 merge 的读操作数目。即 rmerge/s
* wrqm/s: 每秒进行 merge 的写操作数目。即 wmerge/s
* r/s: 每秒完成的读 I/O 设备次数。即 rio/s
* w/s: 每秒完成的写 I/O 设备次数。即 wio/s
* rsec/s: 每秒读扇区数。即 rsect/s
* wsec/s: 每秒写扇区数。即 wsect/s
* rkB/s: 每秒读K字节数。是 rsect/s 的一半，因为每扇区大小为512字节。
* wkB/s: 每秒写K字节数。是 wsect/s 的一半。
* avgrq-sz: 平均每次设备I/O操作的数据大小 (扇区)。
* avgqu-sz: 平均I/O队列长度。
* await: 平均每次设备I/O操作的等待时间 (毫秒)。
* svctm: 平均每次设备I/O操作的服务时间 (毫秒)。
* %util: 一秒中有百分之多少的时间用于 I/O 操作，即被io消耗的cpu百分比

> 备注：如果 %util 接近 100%，说明产生的I/O请求太多，I/O系统已经满负荷，该磁盘可能存在瓶颈。如果 svctm 比较接近 await，说明 I/O 几乎没有等待时间；如果 await 远大于 svctm，说明I/O 队列太长，io响应太慢，则需要进行必要优化。如果avgqu-sz比较大，也表示有当量io在等待。 idle小于70% IO压力就较大了，一般读取速度有较多的wait。 同时可以结合vmstat 查看查看b参数(等待资源的进程数)和wa参数(IO等待所占用的CPU时间的百分比，高过30%时IO压力高)。

# sar 找出系统瓶颈的利器
sar是System Activity Reporter（系统活动情况报告）的缩写。sar工具将对系统当前的状态进行取样，然后通过计算数据和比例来表达系统的当前运行状态。它的特点是可以连续对系统取样，获得大量的取样数据；取样数据和分析的结果都可以存入文件，所需的负载很小。sar是目前Linux上最为全面的系统性能分析工具之一，可以从14个大方面对系统的活动进行报告，包括文件的读写情况、系统调用的使用情况、串口、CPU效率、内存使用状况、进程活动及IPC有关的活动等，使用也是较为复杂。

sar是查看操作系统报告指标的各种工具中，最为普遍和方便的；它有两种用法:
1. 追溯过去的统计数据（默认）
2. 周期性的查看当前数据

## 追溯过去的统计数据

## 参数说明
-A 汇总所有的报告
-a 报告文件读写使用情况
-B 报告附加的缓存的使用情况
-b 报告缓存的使用情况
-c 报告系统调用的使用情况
-d 报告磁盘的使用情况
-g 报告串口的使用情况
-h 报告关于buffer使用的统计数据
-m 报告IPC消息队列和信号量的使用情况
-n 报告命名cache的使用情况
-p 报告调页活动的使用情况
-q 报告运行队列和交换队列的平均长度
-R 报告进程的活动情况
-r 报告没有使用的内存页面和硬盘块
-u 报告CPU的利用率
-v 报告进程、i节点、文件和锁表状态
-w 报告系统交换活动状况
-y 报告TTY设备活动状况

## CPU资源监控

```shell
# 每隔1秒刷新 输出3次
[root@ailegal-ecs-no ~]# sar 1 3
Linux 3.10.0-693.el7.x86_64 (ailegal-ecs-no.04.novalocal) 	01/29/2019 	_x86_64_	(8 CPU)

03:00:20 PM     CPU     %user     %nice   %system   %iowait    %steal     %idle
03:00:21 PM     all      0.13      0.00      0.00      0.00      0.00     99.87
03:00:22 PM     all      0.25      0.00      0.00      0.00      0.00     99.75
03:00:23 PM     all      0.13      0.00      0.00      0.13      0.00     99.75
Average:        all      0.17      0.00      0.00      0.04      0.00     99.79
```

* %user 用户模式下消耗的CPU时间的比例；
* %nice 通过nice改变了进程调度优先级的进程，在用户模式下消耗的CPU时间的比例
* %system 系统模式下消耗的CPU时间的比例；
* %iowait CPU等待磁盘I/O导致空闲状态消耗的时间比例；
* %steal 利用Xen等操作系统虚拟化技术，等待其它虚拟CPU计算占用的时间比例；
* %idle CPU空闲时间比例；

## 进程队列长度和平均负载状态监控
```shell
[root@AI125 ~]# sar -q 1 60
Linux 3.10.0-514.el7.x86_64 (AI125) 	01/29/2019 	_x86_64_	(8 CPU)

03:06:40 PM   runq-sz  plist-sz   ldavg-1   ldavg-5  ldavg-15   blocked
03:06:41 PM         1       881      1.70      0.86      0.42         0
03:06:42 PM         2       881      1.72      0.88      0.43         0
03:06:43 PM         1       881      1.72      0.88      0.43         0
03:06:44 PM         1       881      1.72      0.88      0.43         0
03:06:45 PM         1       881      1.72      0.88      0.43         0
```
输出项说明：
* runq-sz：运行队列的长度（等待运行的进程数）
* plist-sz：进程列表中进程（processes）和线程（threads）的数量
* ldavg-1：最后1分钟的系统平均负载 
* ldavg-5：过去5分钟的系统平均负载
* ldavg-15：过去15分钟的系统平均负载

## 内存和交换空间监控

```shell
[root@AI125 ~]# sar -r 1 3
Linux 3.10.0-514.el7.x86_64 (AI125) 	01/29/2019 	_x86_64_	(8 CPU)

03:14:04 PM kbmemfree kbmemused  %memused kbbuffers  kbcached  kbcommit   %commit  kbactive   kbinact   kbdirty
03:14:05 PM  10608724  13899564     56.71       884   4987532  13142180     53.62   9820600   2253864        60
03:14:06 PM  10608436  13899852     56.71       884   4987536  13142180     53.62   9820196   2253864        64
03:14:07 PM  10608436  13899852     56.71       884   4987536  13142180     53.62   9820196   2253864        64
Average:     10608532  13899756     56.71       884   4987535  13142180     53.62   9820331   2253864        63
```

* kbmemfree：这个值和free命令中的free值基本一致,所以它不包括buffer和cache的空间.
* kbmemused：这个值和free命令中的used值基本一致,所以它包括buffer和cache的空间.
* %memused：物理内存使用率，这个值是kbmemused和内存总量(不包括swap)的一个百分比.
* kbbuffers和kbcached：这两个值就是free命令中的buffer和cache.
* kbcommit：保证当前系统所需要的内存,即为了确保不溢出而需要的内存(RAM+swap).
* %commit：这个值是kbcommit与内存总量(包括swap)的一个百分比.

## 内存分页监控
```shell
[root@AI125 ~]# sar -B 1 3
Linux 3.10.0-514.el7.x86_64 (AI125) 	01/29/2019 	_x86_64_	(8 CPU)

03:27:23 PM  pgpgin/s pgpgout/s   fault/s  majflt/s  pgfree/s pgscank/s pgscand/s pgsteal/s    %vmeff
03:27:24 PM      0.00     35.00  28356.00      0.00  18765.00      0.00      0.00      0.00      0.00
03:27:25 PM      0.00      0.00  76894.06      0.00  22344.55      0.00      0.00      0.00      0.00
03:27:26 PM      0.00    353.00  96876.00      0.00  83092.00      0.00      0.00      0.00      0.00
Average:         0.00    128.90  67406.98      0.00  41337.21      0.00      0.00      0.00      0.00
```
输出项说明：
* pgpgin/s：表示每秒从磁盘或SWAP置换到内存的字节数(KB)
* pgpgout/s：表示每秒从内存置换到磁盘或SWAP的字节数(KB)
* fault/s：每秒钟系统产生的缺页数,即主缺页与次缺页之和(major + minor)
* majflt/s：每秒钟产生的主缺页数.
* pgfree/s：每秒被放入空闲队列中的页个数
* pgscank/s：每秒被kswapd扫描的页个数
* pgscand/s：每秒直接被扫描的页个数
* pgsteal/s：每秒钟从cache中被清除来满足内存需要的页个数
* %vmeff：每秒清除的页(pgsteal)占总扫描页(pgscank+pgscand)的百分比

## 系统交换活动信息监控
页面发生交换时，服务器的吞吐量会大幅下降；服务器状况不良时，如果怀疑因为内存不足而导致了页面交换的发生，可以使用这个命令来确认是否发生了大量的交换
```shell
[root@AI125 ~]# sar -W 1 3
Linux 3.10.0-514.el7.x86_64 (AI125) 	01/29/2019 	_x86_64_	(8 CPU)

03:15:45 PM  pswpin/s pswpout/s
03:15:46 PM      0.00      0.00
03:15:47 PM      0.00      0.00
03:15:48 PM      0.00      0.00
Average:         0.00      0.00
```
输出项说明：
* pswpin/s：每秒系统换入的交换页面（swap page）数量
* pswpout/s：每秒系统换出的交换页面（swap page）数量

##  I/O和传送速率监控
```shell
[root@AI125 ~]# sar -b 1 3
Linux 3.10.0-514.el7.x86_64 (AI125) 	01/29/2019 	_x86_64_	(8 CPU)

03:29:21 PM       tps      rtps      wtps   bread/s   bwrtn/s
03:29:22 PM    252.00      0.00    252.00      0.00  18143.00
03:29:23 PM     42.00      0.00     42.00      0.00   2632.00
03:29:24 PM      4.00      0.00      4.00      0.00     37.00
Average:        99.33      0.00     99.33      0.00   6937.33
```
输出项说明：
* tps：每秒钟物理设备的 I/O 传输总量
* rtps：每秒钟从物理设备读入的数据总量
* wtps：每秒钟向物理设备写入的数据总量
* bread/s：每秒钟从物理设备读入的数据量，单位为 块/s
* bwrtn/s：每秒钟向物理设备写入的数据量，单位为 块/s

## inode、文件和其他内核表监控
```shell
[root@AI125 ~]# sar -v 1 3
Linux 3.10.0-514.el7.x86_64 (AI125) 	01/29/2019 	_x86_64_	(8 CPU)

03:25:36 PM dentunusd   file-nr  inode-nr    pty-nr
03:25:37 PM    433158      6848    324556         3
03:25:38 PM    433158      6848    324556         3
03:25:39 PM    433158      6816    324556         3
Average:       433158      6837    324556         3
```
输出项说明：
* dentunusd：目录高速缓存中未被使用的条目数量
* file-nr：文件句柄（file handle）的使用数量
* inode-nr：索引节点句柄（inode handle）的使用数量
* pty-nr：使用的pty数量

## 设备使用情况监控
```shell
[root@AI125 ~]# sar -d 1 3 -p
Linux 3.10.0-514.el7.x86_64 (AI125) 	01/29/2019 	_x86_64_	(8 CPU)

03:33:14 PM       DEV       tps  rd_sec/s  wr_sec/s  avgrq-sz  avgqu-sz     await     svctm     %util
03:33:15 PM       sda      4.00      0.00     36.00      9.00      0.00      0.25      0.25      0.10

03:33:15 PM       DEV       tps  rd_sec/s  wr_sec/s  avgrq-sz  avgqu-sz     await     svctm     %util
03:33:16 PM       sda      0.00      0.00      0.00      0.00      0.00      0.00      0.00      0.00

03:33:16 PM       DEV       tps  rd_sec/s  wr_sec/s  avgrq-sz  avgqu-sz     await     svctm     %util
03:33:17 PM       sda      3.00      8.00     19.00      9.00      0.01      5.00      5.00      1.50

Average:          DEV       tps  rd_sec/s  wr_sec/s  avgrq-sz  avgqu-sz     await     svctm     %util
Average:          sda      2.33      2.67     18.33      9.00      0.01      2.29      2.29      0.53
```

其中：
参数-p可以打印出sda,hdc等磁盘设备名称,如果不用参数-p,设备节点则有可能是dev8-0,dev22-0

* tps:每秒从物理磁盘I/O的次数.多个逻辑请求会被合并为一个I/O磁盘请求,一次传输的大小是不确定的.
* rd_sec/s:每秒读扇区的次数.
* wr_sec/s:每秒写扇区的次数.
* avgrq-sz:平均每次设备I/O操作的数据大小(扇区).
* avgqu-sz:磁盘请求队列的平均长度.
* await:从请求磁盘操作到系统完成处理,每次请求的平均消耗时间,包括请求队列等待时间,单位是毫秒(1秒=1000毫秒).
* svctm:系统处理每次请求的平均时间,不包括在请求队列中消耗的时间.
* %util:I/O请求占CPU的百分比,比率越大,说明越饱和.

1. avgqu-sz 的值较低时，设备的利用率较高。
2. 当%util的值接近 1% 时，表示设备带宽已经占满。

## 总结
要判断系统瓶颈问题，有时需几个 sar 命令选项结合起来；
* 怀疑CPU存在瓶颈，可用 sar -u 和 sar -q 等来查看
* 怀疑内存存在瓶颈，可用sar -B、sar -r 和 sar -W 等来查看
* 怀疑I/O存在瓶颈，可用 sar -b、sar -u 和 sar -d 等来查看

# wget 文件下载
Linux系统中的wget是一个下载文件的工具，它用在命令行下。对于Linux用户是必不可少的工具，我们经常要下载一些软件或从远程服务器恢复备份到本地服务器。wget支持HTTP，HTTPS和FTP协议，可以使用HTTP代理。
wget 非常稳定，它在带宽很窄的情况下和不稳定网络中有很强的适应性.如果是由于网络的原因下载失败，wget会不断的尝试，直到整个文件下载完毕。如果是服务器打断下载过程，它会再次联到服务器上从停止的地方继续下载。这对从那些限定了链接时间的服务器上下载大文件非常有用。

## 命令格式
wget [参数] [URL地址]

### 命令参数
#### 启动参数
* -V, –version 显示wget的版本后退出
* -h, –help 打印语法帮助
* -b, –background 启动后转入后台执行
* -e, –execute=COMMAND 执行’.wgetrc’格式的命令，wgetrc格式参见/etc/wgetrc或~/.wgetrc

#### 记录和输入文件参数
* -o, –output-file=FILE 把记录写到FILE文件中
* -a, –append-output=FILE 把记录追加到FILE文件中
* -d, –debug 打印调试输出
* -q, –quiet 安静模式(没有输出)
* -v, –verbose 冗长模式(这是缺省设置)
* -nv, –non-verbose 关掉冗长模式，但不是安静模式
* -i, –input-file=FILE 下载在FILE文件中出现的URLs
* -F, –force-html 把输入文件当作HTML格式文件对待
* -B, –base=URL 将URL作为在-F -i参数指定的文件中出现的相对链接的前缀

–sslcertfile=FILE 可选客户端证书 –sslcertkey=KEYFILE 可选客户端证书的KEYFILE –egd-file=FILE 指定EGD socket的文件名

#### 下载参数
* -bind-address=ADDRESS 指定本地使用地址(主机名或IP，当本地有多个IP或名字时使用)
* -t, –tries=NUMBER 设定最大尝试链接次数(0 表示无限制).
* -O –output-document=FILE 把文档写到FILE文件中
* -nc, –no-clobber 不要覆盖存在的文件或使用.#前缀
* -c, –continue 接着下载没下载完的文件
* -progress=TYPE 设定进程条标记
* -N, –timestamping 不要重新下载文件除非比本地文件新
* -S, –server-response 打印服务器的回应
* -T, –timeout=SECONDS 设定响应超时的秒数
* -w, –wait=SECONDS 两次尝试之间间隔SECONDS秒
* -waitretry=SECONDS 在重新链接之间等待1…SECONDS秒
* -random-wait 在下载之间等待0…2*WAIT秒
* -Y, -proxy=on/off 打开或关闭代理
* -Q, -quota=NUMBER 设置下载的容量限制
* -limit-rate=RATE 限定下载输率

#### 目录参数
* -nd –no-directories 不创建目录
* -x, –force-directories 强制创建目录
* -nH, –no-host-directories 不创建主机目录
* -P, –directory-prefix=PREFIX 将文件保存到目录 PREFIX/…
* -cut-dirs=NUMBER 忽略 NUMBER层远程目录

#### HTTP选项
* -http-user=USER 设定HTTP用户名为 USER.
* -http-passwd=PASS 设定http密码为 PASS
* -C, –cache=on/off 允许/不允许服务器端的数据缓存 (一般情况下允许)
* -E, –html-extension 将所有text/html文档以.html扩展名保存
* -ignore-length 忽略 ‘Content-Length’头域
* -header=STRING 在headers中插入字符串 STRING
* -proxy-user=USER 设定代理的用户名为 USER
* proxy-passwd=PASS 设定代理的密码为 PASS
* referer=URL 在HTTP请求中包含 ‘Referer: URL’头
* -s, –save-headers 保存HTTP头到文件
* -U, –user-agent=AGENT 设定代理的名称为 AGENT而不是 Wget/VERSION
* no-http-keep-alive 关闭 HTTP活动链接 (永远链接)
* cookies=off 不使用 cookies
* load-cookies=FILE 在开始会话前从文件 FILE中加载cookie
* save-cookies=FILE 在会话结束后将 cookies保存到 FILE文件中

#### FTP 选项参数
* -nr, –dont-remove-listing 不移走 ‘.listing’文件
* -g, –glob=on/off 打开或关闭文件名的 globbing机制
* passive-ftp 使用被动传输模式 (缺省值).
* active-ftp 使用主动传输模式
* retr-symlinks 在递归的时候，将链接指向文件(而不是目录)

#### 递归下载参数
* -r, –recursive 递归下载－－慎用!
* -l, –level=NUMBER 最大递归深度 (inf 或 0 代表无穷)
* -delete-after 在现在完毕后局部删除文件
* -k, –convert-links 转换非相对链接为相对链接
* -K, –backup-converted 在转换文件X之前，将之备份为 X.orig
* -m, –mirror 等价于 -r -N -l inf -nr
* -p, –page-requisites 下载显示HTML文件的所有图片
* -A, –accept=LIST 分号分隔的被接受扩展名的列表
* -R, –reject=LIST 分号分隔的不被接受的扩展名的列表
* -D, –domains=LIST 分号分隔的被接受域的列表
* -exclude-domains=LIST 分号分隔的不被接受的域的列表
* -follow-ftp 跟踪HTML文档中的FTP链接
* -follow-tags=LIST 分号分隔的被跟踪的HTML标签的列表
* -G, –ignore-tags=LIST 分号分隔的被忽略的HTML标签的列表
* -H, –span-hosts 当递归时转到外部主机
* -L, –relative 仅仅跟踪相对链接
* -I, –include-directories=LIST 允许目录的列表
* -X, –exclude-directories=LIST 不被包含目录的列表
* -np, –no-parent 不要追溯到父目录

## 使用实例
### 使用wget下载单个文件
```bash
$ wget http://www.minjieren.com/wordpress-3.1-zh_CN.zip
```

### 使用wget -O下载并以不同的文件名保存
```bash
$ wget -O wordpress.zip http://www.minjieren.com/download.aspx?id=1080
```

### 使用wget –limit -rate限速下载
```bash
$ wget --limit-rate=300k http://www.minjieren.com/wordpress-3.1-zh_CN.zip
```

### 使用wget -c断点续传
```bash
$ wget -c http://www.minjieren.com/wordpress-3.1-zh_CN.zip
```

### 使用wget -b后台下载
```bash
$ wget -b http://www.minjieren.com/wordpress-3.1-zh_CN.zip
```
你可以使用以下命令来察看下载进度:
```bash
$ tail -f wget-log
```

### 使用wget -i下载多个文件
首先，保存一份下载链接文件,接着使用这个文件和参数-i下载:
```bash
$ cat > filelist.txt
url1
url2
url3
url4

$ wget -i filelist.txt
```

### 使用wget -r -A下载指定格式文件
```bash
$ wget -r -A.pdf url
```
可以在以下情况使用该功能
* 下载一个网站的所有图片
* 下载一个网站的所有视频
* 下载一个网站的所有PDF文件

### 使用wget FTP下载
```bash
# 匿名ftp下载
$ wget ftp-url

# 用户名和密码认证的ftp下载
$ wget --ftp-user=USERNAME --ftp-password=PASSWORD url
```

# scp 跨机远程拷贝
scp是secure copy的简写，用于在Linux下进行远程拷贝文件的命令，和它类似的命令有cp，不过cp只是在本机进行拷贝不能跨服务器，而且scp传输是加密的。当你服务器硬盘变为只读 read only system时，用scp可以帮你把文件移出来。

## 命令格式
scp [参数] [原路径] [目标路径]

### 命令参数
* -1 强制scp命令使用协议ssh1
* -2 强制scp命令使用协议ssh2
* -4 强制scp命令只使用IPv4寻址
* -6 强制scp命令只使用IPv6寻址
* -B 使用批处理模式（传输过程中不询问传输口令或短语）
* -C 允许压缩。（将-C标志传递给ssh，从而打开压缩功能）
* -p 留原文件的修改时间，访问时间和访问权限。
* -q 不显示传输进度条。
* -r 递归复制整个目录。
* -v 详细方式显示输出。scp和ssh(1)会显示出整个过程的调试信息。这些信息用于调试连接，验证和配置问题。
* -c cipher 以cipher将数据传输进行加密，这个选项将直接传递给ssh。
* -F ssh_config 指定一个替代的ssh配置文件，此参数直接传递给ssh。
* -i identity_file 从指定文件中读取传输时使用的密钥文件，此参数直接传递给ssh。
* -l limit 限定用户所能使用的带宽，以Kbit/s为单位。
* -o ssh_option 如果习惯于使用ssh_config(5)中的参数传递方式，
* -P port 注意是大写的P, port是指定数据传输用到的端口号
* -S program 指定加密传输时所使用的程序。此程序必须能够理解ssh(1)的选项。

### 使用实例
#### 从远处复制文件到本地目录
```bash
$ scp root@10.100.25.13:/opt/soft/demo.tar /opt/soft/
```
从10.100.25.13机器上的/opt/soft/的目录中下载demo.tar 文件到本地/opt/soft/目录中

#### 从远处复制目录到本地
```bash
$ scp -r root@10.100.25.13:/opt/soft/test /opt/soft/
```
从10.100.25.13机器上的/opt/soft/中下载test目录到本地的/opt/soft/目录来。

#### 上传本地文件到远程机器指定目录
```bash
$ scp /opt/soft/demo.tar root@10.100.25.13:/opt/soft/scptest
```
复制本地opt/soft/目录下的文件demo.tar 到远程机器10.100.25.13的opt/soft/scptest目录

#### 上传本地目录到远程机器指定目录
```bash
$ scp -r /opt/soft/test root@10.100.25.13:/opt/soft/scptest
```
上传本地目录 /opt/soft/test到远程机器10.100.25.13上/opt/soft/scptest的目录中

# crontab
通过crontab 命令，我们可以在固定的间隔时间执行指定的系统指令或 shell script脚本。时间间隔的单位可以是分钟、小时、日、月、周及以上的任意组合。这个命令非常适合周期性的日志分析或数据备份等工作。

## 命令格式
crontab [-u user] file crontab [-u user] [ -e | -l | -r ]

## 命令参数
* -u user：用来设定某个用户的crontab服务；
* file：file是命令文件的名字,表示将file做为crontab的任务列表文件并载入crontab。如果在命令行中没有指定这个文件，crontab命令将接受标准输入（键盘）上键入的命令，并将它们载入crontab。
* -e：编辑某个用户的crontab文件内容。如果不指定用户，则表示编辑当前用户的crontab文件。
* -l：显示某个用户的crontab文件内容，如果不指定用户，则表示显示当前用户的crontab文件内容。
* -r：从/var/spool/cron目录中删除某个用户的crontab文件，如果不指定用户，则默认删除当前用户的crontab文件。
* -i：在删除用户的crontab文件时给确认提示。

## crontab的文件格式
分 时 日 月 星期 要运行的命令
* 第1列分钟0～59
* 第2列小时0～23（0表示子夜）
* 第3列日1～31
* 第4列月1～12
* 第5列星期0～7（0和7表示星期天）
* 第6列要运行的命令

## 常用方法
### 创建一个新的crontab文件

## 使用实例
### 每1分钟执行一次myCommand
```bash
$ * * * * * myCommand
```

### 每小时的第3和第15分钟执行
```bash
$ 3,15 * * * * myCommand
```

### 在上午8点到11点的第3和第15分钟执行
```bash
$ 3,15 8-11 * * * myCommand
```

### 每隔两天的上午8点到11点的第3和第15分钟执行
```bash
$ 3,15 8-11 */2  *  * myCommand
```

### 每周一上午8点到11点的第3和第15分钟执行
```bash
$ 3,15 8-11 * * 1 myCommand
```

### 每晚的21:30重启smb
```bash
$ 30 21 * * * /etc/init.d/smb restart
```

### 每月1、10、22日的4 : 45重启smb
```bash
$ 45 4 1,10,22 * * /etc/init.d/smb restart
```

### 每周六、周日的1 : 10重启smb
```bash
$ 10 1 * * 6,0 /etc/init.d/smb restart
```

### 每天18 : 00至23 : 00之间每隔30分钟重启smb
```bash
$ ,30 18-23 * * * /etc/init.d/smb restart
```

### 每一小时重启smb
```bash
$ * */1 * * * /etc/init.d/smb restart
```

### 晚上11点到早上7点之间，每隔一小时重启smb
```bash
$ 0 23-7 * * * /etc/init.d/smb restart
```

## 使用注意事项
### 环境变量问题
有时我们创建了一个crontab，但是这个任务却无法自动执行，而手动执行这个任务却没有问题，这种情况一般是由于在crontab文件中没有配置环境变量引起的。
在crontab文件中定义多个调度任务时，需要特别注环境变量的设置，因为我们手动执行某个任务时，是在当前shell环境下进行的，程序当然能找到环境变量，而系统自动执行任务调度时，是不会加载任何环境变量的，因此，就需要在crontab文件中指定任务运行所需的所有环境变量，这样，系统执行任务调度时就没有问题了。

不要假定cron知道所需要的特殊环境，它其实并不知道。所以你要保证在shelll脚本中提供所有必要的路径和环境变量，除了一些自动设置的全局变量。所以注意如下3点：

1. 脚本中涉及文件路径时写全局路径；
2. 脚本执行要用到java或其他环境变量时，通过source命令引入环境变量，如:

    ```bash
    cat start_cbp.sh
    !/bin/sh
    source /etc/profile
    export RUN_CONF=/home/d139/conf/platform/cbp/cbp_jboss.conf
    /usr/local/jboss-4.0.5/bin/run.sh -c mev &
    ```
3. 当手动执行脚本OK，但是crontab死活不执行时,很可能是环境变量惹的祸，可尝试在crontab中直接引入环境变量解决问题。如:
    ```bash
    0 * * * * . /etc/profile;/bin/sh /var/www/java/audit_no_count/bin/restart_audit.sh
    ```
    
### 注意清理系统用户的邮件日志
每条任务调度执行完毕，系统都会将任务输出信息通过电子邮件的形式发送给当前系统用户，这样日积月累，日志信息会非常大，可能会影响系统的正常运行，因此，将每条任务进行重定向处理非常重要。 例如，可以在crontab文件中设置如下形式，忽略日志输出:
```bash
0 */3 * * * /usr/local/apache2/apachectl restart >/dev/null 2>&1
```

### 系统级任务调度与用户级任务调度
系统级任务调度主要完成系统的一些维护操作，用户级任务调度主要完成用户自定义的一些任务，可以将用户级任务调度放到系统级任务调度来完成（不建议这么做），但是反过来却不行，root用户的任务调度操作可以通过”crontab –uroot –e”来设置，也可以将调度任务直接写入/etc/crontab文件，需要注意的是，如果要定义一个定时重启系统的任务，就必须将任务放到/etc/crontab文件，即使在root用户下创建一个定时重启系统的任务也是无效的。

### 其他注意事项
新创建的cron job，不会马上执行，至少要过2分钟才执行。如果重启cron则马上执行。

当crontab失效时，可以尝试/etc/init.d/crond restart解决问题。或者查看日志看某个job有没有执行/报错tail -f /var/log/cron。

千万别乱运行crontab -r。它从Crontab目录（/var/spool/cron）中删除用户的Crontab文件。删除了该用户的所有crontab都没了。

在crontab中%是有特殊含义的，表示换行的意思。如果要用的话必须进行转义%，如经常用的date ‘+%Y%m%d’在crontab里是不会执行的，应该换成date ‘+%Y%m%d’。

更新系统时间时区后需要重启cron,在ubuntu中服务名为cron:
```bash
$ service cron restart
```
ubuntu下启动、停止与重启cron:
```bash
$ sudo /etc/init.d/cron start
$ sudo /etc/init.d/cron stop
$ sudo /etc/init.d/cron restart
```