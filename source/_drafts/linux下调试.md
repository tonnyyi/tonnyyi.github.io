# gbd 调试利器
GDB是一个由GNU开源组织发布的、UNIX/LINUX操作系统下的、基于命令行的、功能强大的程序调试工具。 对于一名Linux下工作的c++程序员，gdb是必不可少的工具

# ldd 查看程序依赖库

# lsof 一切皆文件

# ps 进程查看器

# pstack 跟踪进程栈

# strace 跟踪进程中的系统调用
```shell
$ strace -o output.txt -f -T -tt -e trace=all -p 11209
$ strace -f -t -T -e trace=all -p 11209 2<&1 | tee -a output.txt
```


# ipcs 查询进程间通信状态

# top linux下的任务管理器

# free 查询可用内存
```shell
[root@AI125 ~]# free -h
              total        used        free      shared  buff/cache   available
Mem:            23G        7.1G        9.2G        1.1G        7.1G         14G
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

# readelf

# objdump

# nm

# size

# wget

# scp

# crontab
