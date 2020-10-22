---
title: Linux常用命令
tags:
  - linux
  - 命令
categories:
  - linux
  - command
date: 2018-03-08 15:09:41
---

## 文件/目录

### ls 文件列表
```bash
# -1 :每个文件展示一行  -l :展示全部信息(权限, 时间等, 首字母为文件类型: -:普通文件 d:文件夹 s:socket文件 l:链接文件)  
ls -t    # 按更新时间排序, 最新的在前
ls -tr    # 按更新时间排序, 最旧的在前, -r:倒序
ls -h    # 使用可读单位(K, M, G)显示文件大小
ls -d /dir    # 只显示目录信息, 不显示目录内文件信息
ls -a    # 显示所有文件, 包括 . 和 ..
ls -A    # 显示所有文件, 不包括 . 和 ..
ls -R    # 递归展示所有文件, 及子目录内所有文件
```
<!-- more -->
### mkdir 建目录
```bash
mkdir dir   # 创建目录
mkdir -p path/to/dir       # 递归创建多个目录
mkdir -m 777 dir    # 创建权限为777的目录
mkdir -p dir, dir2, path/to/dir3    # 创建多个目录
```

### rm  删除
```bash
rm file      # 删除前需要确认
rm -f file    # 强行删除, 系统不再提示
rm -i file*     # 删除多个文件, 逐个确认
rm -r dir      # 递归删除文件夹下所有文件, 并删除该文件夹
rm -rf dir     # 毁天灭地
```

### cp  复制
```bash
cp file1 path/          # 复制到目录, 保持文件名
cp file1 path/file2     # 复制并重命名
cp -r path/dir1 path/to/dir2    # 递归复制一个目录
cp -i file1 file2    # 重命名文件为file2, 如果file2已存在则提示是否覆盖
cp -a dir1 dir2     # 如果dir2已存在, 则将dir1复制到dir2目录里, 否则dir1并重命名为dir2
cp -b file1 file2    # file2已存在时, 会先备份, 再复制
```

### mv 移动
```bash
mv -i file1 file2   # 如果file2已存在则提示
mv -f file1 file2   # 如果file2已存在直接覆盖, 不提示
mv -b file1 file2   # 如果file2已存在, 则覆盖, 但会先备份
mv file1 file2 dir  # 移动多个文件到dir
```

### chmod  改权限
`ls -al`输出类似`-rw-r--r-- 1 root root 302108 11-13 06:03 xx.log`的内容. 分三部分: 文件所属人的权限`u`, 文件所属组的权限`g`, 其他用户的权限`o`.

权限范围:
- **u** : 目录或文件的当前用户
- **g** : 目录或文件的当前群组
- **o** : 除了目录或文件的当前用户和群组之外的用户或者群组
- **a** : 所有的用户及群组

权限代号:
- **r** : 读权限, 用数字4表示
- **w** : 写权限, 用数字2表示
- **x** : 执行权限, 用数字1表示
- **\-** : 删除权限, 用数字0表示
- **s** : 特殊权限

```bash
chmod a+x file      # 写法1, 所有人加执行权限
chmod ug+x,o-x file     # 所有人及所在组加执行权限, 其他去除执行权限
chmod o=r file          # 其他用户只有读取权限
chmod u=rwx,g=rx,o=x file   # 指定每个范围的权限
chmod 755 file      # 写法2
chmod -R u+x dir    # 递归修改
chmod =r file   # 为所有用户分配权限
```

### chown  改所属
```bash
chown root file     # 修改文件所属人
chown redis:redis file      # 修改文件所属人及组
chown root: file    # 修改文件所属人为root, 组改为root所在组
chown :mail file    # 仅修改组
chown -R root dir   # 递归修改所属人
```

### find  查找
```bash
find -name "Hello.java"   # 按名称在当前目录遍历查找
find / -name "Hello.java"    # 遍历查找文件
find / -iname "Hello.java"   # 不区分文件名大小写查找
find . -regex "\./\w+\.log$"   # 正则查找
find . -iregex "\./\w+\.log$"   # 不区分大小, 正则查找
find / -maxdepth 2 -name "Hello.java"  # 限制遍历深度, 2: 当前目录算1级 + 一级子目录
find / -mindepth 2 -maxdepth 4 -name "Hello.java"  # 限制遍历层级范围
find -iname "Hello.java" -exec md5sum {} \;  # 对查找到的文件执行命令, {}为文件名占位符
find -not -iname "Hello.java"  # 查找名称不是"Hello.java"的文件
find . -perm 755 -type f -exec ls -l {} \;   # 按文件权限查找 type: b:块设备文件, d:目录 c:字符设备文件 p:管道文件 l:符号链接文件 f:普通文件 s:socket文件
find . -empty   # 查找空文件, 即文件大小为0
find . -type f -exec ls -s {} \; | sort -n -r | head -5    # 查找最大的前5个文件
find . -not -empty -type f -exec ls -s {} \; | sort -n | head -5   # 查找前5个最小的非空文件
find -newer file1   # 查找比file1更新的文件
find . -size 100M   # 查找大小为100M的文件
find . -size +100k -size -1M    # 查找大小在100k 到 1M的文件

# atime:access time, 被读取或执行的时间  ctime:change time: 文件状态改变时间  mtime: modify time, 文件内容改变时间  通过stat命令查看这些时间
find . -mtime 2   # 往前3天内(2*24 - 3*24)前被修改的文件
find . -mtime -2  # 从2天前开始
find . -mtime +2  # 2天之前一直往前
find . -mmin 2   # 按分钟查找
# -mtime n : 计算方式:(current_time - file_time) / 86400
```

### locate
```bash
locate pwd      # 类似find -name, 但该命令较快, 因为它是从数据库(/var/lib/locatedb)里查询, 可以使用updatedb命令更新数据
locate ~/m      # 查找以m开头的文件
locate -i ~/m   # 忽略大小写
```

### which
```bash
which grep      # 在PATH变量指定的路径中, 查询某个命令的位置, 并且返回第一个搜索结果
```

### whereis
```bash
# 只能搜索程序名, 而且只搜索二进制(-b), man文件(-m), 和源码(-s)
whereis grep
```

### type
```bash
# 区分某个命令是否是shell自带的
type cd     # cd is a shell builtin
type grep   # grep is /bin/grep
type -p grep    # 加 -p 等同于which
```

### sort
```bash
# -f 忽略大小写, -C检查文件是否已排好序，如果乱序，不输出内容，仅返回1
sort file   # 文件中每行按首字母排序后打印
sort -r file  # 逆序
ls -l . | sort -nk2   # 按第2列排序, 默认ascii排序(会出现10 < 2), 基于数字值排序时必须加n
sort -nk2 -t : file   # 按数字排序第2列, 使用:作为分隔符
sort -u file   # 不输出重复行
sort file -o file   # 排序后写入原文件
sort -n -t : -k 3nr -k 2 file   # 以:分隔列, 先按第3列倒叙排, 相同时再按第2列排序
sort -n -k 2.2,3.1 file    # 以第2列的第3个字符到第3列的第1个字符排序
```

### realpath

### readlink



## 文件内容

### cat  读取内容
```bash
cat file1 file2    # 顺序打印多个文件内容
cat -n file        # 每行前加行号
cat -b file        # 空白行不加行号
cat file1 > file2  # 将file1的内容输入到file2, file2原有内容被清空
cat file2 >> file2  # file1文件追加到file2
```

### grep 过滤
```bash
# -i :不区分大小写;  -4 :前后4行;  -n :输出行号
grep "the" file
grep "the" file_*  # 查找所有file_开头的文件
grep -i "the" file  # 不区分大小写查找
grep "th.*" file  # 正则查找, 推荐用egrep 或 -e
egrep "th.{2}" file
grep -v -e "^th" -e "se$" -e "at"  # 除了th开头, se结尾, 包含at 的其他行
grep -w "is" file  # 完整匹配, 不会匹配上this/his, 只查找is

grep -A 3 -i "the" file  # 输出匹配到的行, 及之后的3行
grep -B 3 -i "the" file  # 输出匹配到的行, 及之前的3行
grep -3 -i "the" file  # 输出匹配到的行, 及前后的3行, 可以写 -3i

# 高亮匹配到的字符
export GREP_OPTIONS='--color=auto' GREP_COLOR='1;31'
grep -w "is" file

grep -r 'is' *   # 遍历查询当前目录及子目录中的所有文件
grep -v "is" file  # 查找不包含is的行
grep -c "is" file  # 统计匹配到的行数, 再加-v统计不包含的行数
grep -l "is" *.conf  # 打印内容匹配上的行所在的文件名
grep -o "is.*good"   # 只打印匹配上的字符串, 而不是打印整行内容
```

### tar 压缩包
```bash
# -c :创建压缩文件;  -x :解压压缩文件; -t :查看压缩文件
# -v :verbose  -f :file

# 创建压缩文件
tar cvf xxx.tar dirname/
tar cvf xxx.tar file1 file2
tar cvzf xxx.tar.gz dirname/   # gzip压缩
tar cvjf xxx.tar.bz2 dirname/  # bzip2压缩
tar cavf xxx.tar.yy file  # 根据文件后缀yy自动选择压缩程序

# 解压
tar xvf xxx.tar
tar xvzf xxx.tar.gz
tar xvjf xxx.tar.bz2
tar xavf xxx.tar.yy   # 根据文件后缀yy自动选择压缩程序
tar xvf xxx.tar /path/to/file  #从压缩文件中解压出file文件
tar xvf xxx.tar /path/to/dir1/ /path/to/dir2/  #从压缩文件中解压出dir1, dir2两个目录
tar xvf xxx.tar --wildcards "*.txt"  # 只解压出压缩文件中的txt文件

# 往tar文件中添加文件, 不可以往gz/bz2文件中添加
tar rvf xxx.tar newfile

# 查看压缩文件内的文件
tar tvf xxx.tar
tar tvzf xxx.tar.gz
tar tvjf xxx.tar.bz2
```

### tail/head 头尾
```bash
head file       # 默认展示前10行
head -5 file    # 前5行
head -n 5 file      # 前5行
head -c 5 file      # 前5个字符
head -n -5 file     # 除了最后5行的其他行
head -n 20 file | tail -n 10    # 查看前20行中的后10行
```

### less/more 分段浏览
```bash
# more只能往后看, less可以看后翻页
# ctrl + F :前移1屏   ctrl + B :后移1屏   ctrl + D :前移半屏    ctrl + U :后移半屏
# j :前移1行  k :后移1行   G :移动到最后1行  g :移动到首行  q :退出
history | less
less -N file    # 加行号
```

### sed
sed是一种在线编辑器, 每次处理一行内容. 处理时, 把当前行存储在临时缓冲区中, 称为"模式空间"(pattern space), 接着sed命令处理缓冲区中的内容. 处理完成后, 将缓冲区内容输出到屏幕. 然后接着处理下一行, 直至文件结束.
```bash
# a:新增  c:替换  d:删除  i:插入  p:打印  s:取代
nl file | sed '2,5d'        # 删除2,5行
nl file | sed '2,$d'        # 从第2行删除到最后
nl file | sed '2a test'     # 在第2行后加上一行, 内容是test
nl file | sed '2i test'     # 在第2行前加一行, 内容是test
nl file | sed '2,4c Hello World'    # 2到4行替换为Hello World, 就一行
sed -n '2,5p' file      # 显示2 - 5行内容
nl file | sed '/root/d'     # 删除包含root的行
nl file | sed 's/^.*test/hello/g'   # sed 's/待替换字符/替换字符/g'
sed -i 's/\.$/\!/g' file    # 直接修改文件内容, 将行末.替换为!
```

### awk
```bash

```



## 系统信息

### uname
```bash
uname -a    # Linux host.localdomain 4.10.5-1.el6.elrepo.x86_64 #1 SMP Wed Mar 22 14:55:33 EDT 2017 x86_64 x86_64 x86_64 GNU/Linux
```

### passwd
存放用户信息:
> /etc/passwd
> /etc/shadow

存放组信息:
> /etc/group
> /etc/gshadow

用户文件信息组成:
> 例: jack:X:503:504:::/home/jack/:/bin/bash
> jack  //用户名  
> X     //口令,密码  
> 503   //用户id, 0代表root, 普通用户从500开始
> 504   //所在组
> :  //描述
> /home/jack/        //用户的家目录
> /bin/bash     // 用户默认shell

组文件信息组成:
> 例: jack:$!$:???:13801:0:99999:7:*:*:
> jack          // 组名
> $!$           // 被加密的口令
> 13801         // 创建日期与今天相隔的天数
> 0             // 口令的最短位数
> 99999      // 用户口令
> 7          // 到7天时提醒
> \*          // 禁用天数
> \*          // 过期天数

```bash
passwd  # 命令行修改密码, 先输入旧的, 再输入新的
passwd USERNAME    # root用户可以修改其他用户的密码
passwd -d USERNAME   # root用户可以删除别的用户的密码, 删除后该用户可以不输入密码进入系统
passwd -S USERNAME   # 查看用户密码信息, 看不到密码内容
```



### top 任务管理器

```
$ top
top - 16:41:15 up 58 days,  4:44,  1 user,  load average: 0.18, 0.23, 0.22
Tasks: 137 total,   1 running, 136 sleeping,   0 stopped,   0 zombie
Cpu(s):  1.3%us,  0.6%sy,  0.0%ni, 97.5%id,  0.4%wa,  0.0%hi,  0.0%si,  0.2%st
Mem:   3918972k total,  3714896k used,   204076k free,     9192k buffers
Swap:  2096440k total,    59020k used,  2037420k free,   194376k cached

  PID USER      PR  NI  VIRT  RES  SHR S %CPU %MEM    TIME+  COMMAND
 4667 sankai   20   0 2608m  30m 4608 S  4.0  0.8   1069:43 sg_agent
30349 nobody    20   0 7251m 3.0g  12m S  4.0 80.1 114:47.91 java
 4617 sankai   20   0  308m 6644 2712 S  2.0  0.2  81:07.54 cplugin
    1 root      20   0 51236  27m  536 S  0.0  0.7   0:03.28 init
```
前5行为信息统计区:
1. **16:41:15** - 系统当前时间;  **up 58 days,  4:44** - 运行时长, 期间没有重启过; **1user** - 当前有1个用户登录; **load average: 0.18, 0.23, 0.22** - 系统在过去*1*分钟, *5*分钟, *15*分钟的负载情况
2. 系统当前有137个进程, 1个在运行, 136个在休眠, 0个停止, 0个僵尸态
3. *1.3%us* - 用户空间占用CPU的百分比; *0.6%sy* - 内核空间占用CPU的百分比; *0.0%ni* - 改变过优先级的进程占用CPU的百分比; *97.5%id* - 空闲CPU百分比;  *0.4%wa* - IO等待占用CPU百分比; *0.0%hi* - 硬中断(Hardware Interrunpt)占用CPU百分比; *0.2%st* - 软中断占用CPU百分比
4. *3918972k total* - 物理内存总容量(4G); *3714896k used* - 使用中的内存容量; *204076k free* - 空闲的内存容量; *9192k buffers* - 缓存的内存容量
5. *2096440k total* - 交换区总容量(2G); *59020k used* - 使用的交换区容量; *2037420k free* - 空闲的交换区容量; *194376k cached* - 缓冲的交换区容量
6. 空行
7. 进程状态
    1. PID - 进程id
    2. USER - 进程所有者
    3. PR - 进程优先级
    4. NI - nice值, 负值表示高优先级, 正值表示低优先级
    5. VIRT - 进程使用的虚拟内存总量, 单位KB, VIRT=SWAP+RES
    6. RES - 进程使用的未被换出的物理内存大小, KB. RES=CODE+DATA
    7. SHR - 共享内存大小, KB.
    8. S - 进程状态
    9. %CPU - 上次更新到现在的CPU时间占用百分比
    10. %MEN - 进程使用的物理内存百分比
    11. TIME+ - 进程使用的CPU时间总计, 单位: 1/100秒
    12. COMMAND - 进程使用的命令

> load average : 处理进程占CPU能力的百分比, 每隔5秒钟检查一次. 如果CPU每分钟最多处理100个进程，那么系统负荷0.2，意味着CPU在这1分钟里只处理了20个进程. 超过1说明有进程在排队等待CPU处理. 单CPU时负载大于0.7就需要注意了. 达到5时系统基本死机状态.
> `cat /proc/cpuinfo` 命令可查看CPU信息, `grep -c 'model name' /proc/cpuinfo` 查看cpu核心数. **负载值不要超过核心数**
> 应该关注15分钟的负载值, 1分钟的说明只是暂时现象
> 第四行中使用中的内存总量（used）指的是现在系统内核控制的内存数，空闲内存总量（free）是内核还未纳入其管控范围的数量。纳入内核管理的内存不见得都在使用中，还包括过去使用过的现在可以被重复利用的内存，内核并不把这些可被重新使用的内存交还到free中去，因此在linux上free内存会越来越少，但不用为此担心。
>如果出于习惯去计算可用内存数，这里有个近似的计算公式：第四行的free + 第四行的buffers + 第五行的cached，按这个公式此台服务器的可用内存：18537836k +169884k +3612636k = 22GB左右。
>对于内存监控，在top里我们要时刻监控第五行swap交换分区的used，如果这个数值在不断的变化，说明内核在不断进行内存和swap的数据交换，这是真正的内存不够用了。

#### 查看线程
```bash
top -H -p
```

#### 交互操作
`q` 退出top命令
`<Space>` 立即刷新
`s` 设置刷新时间间隔
`c` 显示命令完全模式
`t` 显示或隐藏进程和CPU状态信息
`m` 显示或隐藏内存状态信息
`l` 显示或隐藏uptime信息
`f` 增加或减少进程显示标志
`S` 累计模式，会把已完成或退出的子进程占用的CPU时间累计到父进程的MITE+
`P` 按%CPU使用率排行
`T` 按MITE+排行
`M` 按%MEM排行
`u` 指定显示用户进程
`r` 修改进程renice值
`kkill` 进程
`i` 只显示正在运行的进程
`W` 保存对top的设置到文件^/.toprc，下次启动将自动调用toprc文件的设置
`1` 显示各逻辑CPU状况
`h` 帮助命令



### ps  查看进程

```bash
# linux进程状态: 运行(R), 可中断的睡眠状态(S), 不可中断的睡眠状态(D), 僵尸(Z), 停止(T), 退出,即将被销毁(X)
# 输出列含义: PID: 进程id  %CPU:使用CPU资源占比  %MEM:所占内存百分比  VSZ:用掉的虚拟内容(KB)  RSS:占用的固定的内存(KB)  TTY:在哪个终端上运作  STAT:当前状态 TIME:用掉的CPU时间  CMD:使用的命令
px -ef 	    # 显示进程信息, 包括命令行, 比axu多了父进程id, 且运行时间以时分秒格式显示
ps aux      # 显示进程信息, 包括命令行, CPU/内存使用率, 运行数据为分秒格式
ps auxww    # 包含完整的命令路径
ps --user $(id -u) f       # 树状展示当前用户的进程
ps -o ppid= -p pid      # 展示某个进程的父进程id
ps -u root  # 显示指定用户启动的进程
ps -efL  # 显示线程信息
ps axjf  # 显示线程信息
```



## 网络

### ifconfig  网络管理

```bash
# 显示网络设备信息(激活状态的)
ifconfig
# 全部网络接口
ifconfig -a

# 启动关闭指定网卡
ifconfig eth0 up
ifconfig eth0 down

# 配置IP地址
ifconfig eth0 192.168.1.110    
ifconfig eth0 192.168.1.100 netmask 255.255.255.0
ifconfig eth0 192.168.1.100 netmask 255.255.255.0 broadcast 192.168.1.255

# 修改MAC地址
ifconfig eth0 hw ether 00:AA:BB:CC:dd:EE

# 设置网卡最大传输单元MTU
ifconfig eth0 mtu 1500
```



### IP 网络配置

```bash
# 显示网络接口信息
ip link show
# 显示更加详细的设备信息 可以看到各接口数据统计
ip -s link list

# 开启/关闭网卡
ip link set eth0 up
ip link set eth0 down

# 设置网卡队列长度
ip link set eth0 txqueuelen 1200
# 设置网卡最大传输单元MTU
ip link set eth0 mtu 1500

# 显示网卡ip信息
ip addr show

# 设置/删除网卡IP地址
ip addr add 192.168.1.100/24 dev eth0
ip addr del 192.168.1.100/24 dev eth0

# 显示系统路由表条目
ip route show
ip route list
# 显示局域网内设备的MAC
ip neigh list
# 设置默认路由
ip route add default via 192.168.1.254
# 设置默认网关
ip route add default via 192.168.0.254 dev eth0
# 设置192.168.4.0网段的网关为192.168.0.254, 数据走eth0接口
ip route add 192.168.4.0/24 via 192.168.0.254 dev eth0
# 删除192.168.4.0网段的网关
ip route del 192.168.4.0/24
# 删除默认路由
ip route del default
# 删除路由
ip route delete 192.168.1.0/24 dev eth0

# 查看ip地址路由包从哪里来
ip route get 8.8.8.8
8.8.8.8 via 192.168.64.1 dev eth0  src 192.168.64.6
    cache
```



### netstat

```bash
netstat -nau | grep 3306    
lsof -i :8080
```

我们常说的丢包有三种：

- 一种是内核能够记录的，也就是协议栈上的丢包；
- 另一种是网卡能够记录的，就是网卡处理不过来丢包；
- 第三种就是传输过程中被丢弃的，也就是离开本机之后，未能到达目标主机的。

不管哪种丢包，根据TCP协议的重传规则，我们都可以通过重传来估算系统的丢包情况：

```bash
$ netstat -st | grep retrans
    2500090 segments retransmited
    725553 fast retransmits
    1146287 forward retransmits
    22308 retransmits in slow start
    6113 SACK retransmits failed

# 使用watch持续观察
$ watch netstat -st
```

查看全队列配置情况

```bash
$ netstat -st |grep SYN

$ netstat -st | grep overflowed
$ netstat -st | grep dropped
```

如果`xx times the listen queue of a socket overflowed` 和 `xx SYNs to LISTEN sockets dropped`出现持续的增长, 说明服务器的全队列过小, 全队列发生溢出, 后续的请求就会被丢弃, 服务端出现请求数量上不去的现象.

查看接收buffer情况

```bash
$ netstat -s
```
通常我们可以通过查找 netstat -s 中 overrun、 collapse、pruned 等事件来观察，如果相关的统计项一直都在增长，那么说明应用的接收缓冲区需要调整了



### ss - 网络

```bash
# 显示所有TCP连接
ss -nat

# socket摘要
ss -s

# 显示正在监听的TCP连接
ss -nlt
# Recv-Q: 当前全连接队列的大小, 即完成三次握手并等待进程调用accept()的TCP数量
# Send-Q: 当前TCP服务的最大全连接数

# 非监听状态的TCP连接
ss -nt
# Recv-Q: 已收到但未被应用进程读取的字节数
# Send-Q: 已发送但未收到确认的字节数

# 统计各状态的TCP连接
ss -nat | awk '{print $1}' | sort | uniq -c | sort -m

# 查看某个状态的连接 established/time-wait/fin-wait-1/fin-wait-2/syn-recv
# syn-sent/listening/last-ack/closing/close-wait/closed
# all:上面的全部 connected: 除了list和closed  synchronized: connected中除了syn-sent
# bucket: maintained as minisockets,如：time-wait和syn-recv big: 和bucket相反.
ss state time-wait
ss -a state SYN-RECV

# 查看连接到端口的socket
ss dst :3306

# 查看连接到目标ip(端口)的链接(tcp/udp)
ss dst 192.168.1.5
ss dst 192.168.1.5:3306
# 根据源ip查看连接
ss src 192.168.1.230

# 根据端口
ss dport = :3306   # 连接到3306端口的
ss sport = :http   # 从80端发起连接的
# <= or le    >= or ge    == or eq    != or ne    < or gt     > or lt
ss dport \> :1024
ss sport \> :1024
ss sport \< :32000
ss sport eq :22
ss dport != :22

# 列举出处于established状态的源端口为80或者443，目标网络为193.233.7/24所有tcp套接字
ss -o state established '( sport = :http or sport = :https )' dst 193.233.7/24

# 占用端口
ss -ntp | grep 6379
```

- `-a` 显示所有套接字    `-l` 显示监听状态的套接字   `-s` 显示套接字使用概况
- `-t` 仅显示tcp连接,   `-u` 仅显示udp连接
- `-4` 仅显示IPv4套接字    `-6` 仅显示IPv6套接字
- `-p` 显示使用该套接字的进程信息, 会显示进程命令/进程id/文件描述符
- `-m` 显示套接字的内存使用情况
- `-e` 显示详细的套接字信息   `-i` 显示TCP内部信息  
- `-r` 解析主机名   `-n`不解析服务名称(不加的话不显示端口, 而是显示服务名)

[TCP 半连接队列和全连接队列满了会发生什么？又该如何应对？](https://www.cnblogs.com/xiaolincoding/p/12995358.html)



### ethtool 网卡信息查看

```bash
[root@jzcpx-no ~]# ethtool enp95s0f0
Settings for enp95s0f0:
	Supported ports: [ FIBRE ]
	Supported link modes:   10000baseT/Full
	Supported pause frame use: Symmetric
	Supports auto-negotiation: No
	Advertised link modes:  Not reported
	Advertised pause frame use: No
	Advertised auto-negotiation: No
	Speed: 10000Mb/s
	Duplex: Full
	Port: Direct Attach Copper
	PHYAD: 0
	Transceiver: external
	Auto-negotiation: off
	Supports Wake-on: g
	Wake-on: g
	Current message level: 0x0000000f (15)
			       drv probe link timer
	Link detected: yes
```

`Speed: 10000Mb/s`  万兆网卡,  最大1.22GB/s



### nslookup 查询DNS记录

```bash
$ nslookup nslookup www.baidu.com
Server:		100.100.2.136
Address:	100.100.2.136#53

Non-authoritative answer:
www.baidu.com	canonical name = www.a.shifen.com.
Name:	www.a.shifen.com
Address: 180.101.49.11
Name:	www.a.shifen.com
Address: 180.101.49.12
```



### dig 查询DNS解析过程

```bash
$ dig baidu.com

; <<>> DiG 9.11.4-P2-RedHat-9.11.4-16.P2.el7_8.6 <<>> baidu.com
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 43573
;; flags: qr rd ra; QUERY: 1, ANSWER: 2, AUTHORITY: 0, ADDITIONAL: 0

;; QUESTION SECTION:
;baidu.com.			IN	A

;; ANSWER SECTION:
baidu.com.		549	IN	A	39.156.69.79
baidu.com.		549	IN	A	220.181.38.148

;; Query time: 0 msec
;; SERVER: 100.100.2.136#53(100.100.2.136)
;; WHEN: 二 9月 29 18:37:00 CST 2020
;; MSG SIZE  rcvd: 59
```



### mount 磁盘挂载

```bash

```



### time 统计命令执行时间

```bash
$ time ps axu
USER       PID %CPU %MEM    VSZ   RSS TTY      STAT START   TIME COMMAND
...
real	0m0.025s
user	0m0.006s
sys	0m0.019s
```

- `real`  挂钟时间, 即命令从开始到结束的时间, 包括其他进程所占用的时间片和进程被阻塞所花费的时间
- `user`  花费在用户模式中的CPU时间, 这是唯一真正用于执行进程所花费的时间, 其他进程和阻塞时间不算在内
- `sys`  花费在内核模式中的CPU时间, 代表在内核中执行系统调用所花费的时间, 这也是真正由进程使用的CPU时间


### date 系统时间

```bash
date
date -s 20180308        # 设置系统日期, 时间部分全为0, 只有root有权限
date -s 01:23:30        # 设置时间, 日期部分不变
date -s "20180308 01:23:30"     # 设置系统时间
date -s "01:23:30 2018-03-08"   # 也可以这样设置
date +%Y%m%d    # 显示当前年月日
date +%Y%m%d --date='+1 day'   # 显示昨天的年月日
date +%Y%m%d --date="+1 month"  # 显示下一月的日期
date '+%d'      # 2018-03-10
date '+%T'      # 03:30:21
date -d '2 weeks'   # 2周后
date -d '-100 days'     # 100天后
date -d 'next monday'   # 下个周一
date -d last-month +%Y%m    # 上个月
```



## 应用

### yum

```bash
yum check-update   # 列出所有可更新的软件清单
yum update     # 更新所有软件
yum update xxx   # 更新指定软件 
yum install -y xxx    # 安装
yum list    # 列出可安装的软件列表
yum remove -y  xxx    # 卸载
yum search xxx   # 查找
yum info xxx   #显示指定的rpm软件包的描述信息和概要信息
yum resovledep xx #显示rpm软件包的依赖关系
yum localinstall xx   #安装本地的rpm软件包
yum deplist xxx  #显示rpm软件包的所有依赖关系

yum search pam*   # 通配符查找
# 检查 MySQL 是否已安装
yum list installed | grep mysql
yum list installed mysql*

# 系统初始化安装
yum install gcc gcc-c++ cmake pcre pcre-devel zlib zlib-devel openssl openssl-devel vim wget telnet setuptool lrzsz dos2unix net-tools bind-utils tree screen iftop ntpdate tree lsof iftop iotop -y
yum groupinstall "Development tools" -y
```

使用国内yum源

```bash
# 1. 备份原始源信息
mv /etc/yum.repos.d/CentOS-Base.repo /etc/yum.repos.d/CentOS-Base.repo.backup

# 2. 替换国内yum源
wget -o /etc/yum.repos.d/CentOS-Base.repo http://mirrors.163.com/.help/CentOS6-Base-163.repo

# 3. 生成缓存
yum clean all
yum makecache

# 安装epel源
mv /etc/yum.repos.d/epel.repo /etc/yum.repos.d/epel.repo.backup  
mv /etc/yum.repos.d/epel-testing.repo /etc/yum.repos.d/epel-testing.repo.backup  
curl -o /etc/yum.repos.d/epel.repo http://mirrors.aliyun.com/repo/epel-7.repo
cat /etc/yum.repos.d/epel.repo
```

> **阿里云与163源**
>
> **CentOS7**  
>
> http://mirrors.aliyun.com/repo/Centos-7.repo
>
> http://mirrors.163.com/.help/CentOS7-Base-163.repo
>
> **CentOS6**  
>
> http://mirrors.aliyun.com/repo/Centos-6.repo
>
> http://mirrors.163.com/.help/CentOS6-Base-163.repo
>
> **CentOS5** 
>
> http://mirrors.aliyun.com/repo/Centos-5.repo
>
> http://mirrors.163.com/.help/CentOS5-Base-163.repo



### rpm  软件包管理

对于已安装软件

```bash
rpm -qa | grep mysql # 查询
rpm -qf /usr/lib/libanl.so.1   # 查询安装文件属于哪个软件包
rpm -qi xx   # 查询一个已安装软件包的信息
rpm -ql unzip-6.0-16.el7.x86_64   # 查询已安装软件包都安装到了哪
rpm -qc xx   # 查询已安装软件的配置文件
rpm -qd xx   # 查询已安装软件的文档位置
rpm -qR xx   # 查询已安装软件所依赖的软件包与文件

# 查看软件包的安装位置
updatedb
locate mysql-community-server
```

对于未按照软件包

```bash
rpm -qip mysql-community-server-5.7.16-1.el7.x86_64.rpm  # 查询软件包信息
rpm -qpl xx.rpm  # 查询软件包所包含的文件
rpm -qpc xx.rpm  # 查询软件包所包含的文件
rpm -qpd xx.rpm  # 查询软件包的文档位置
rpm -qpR xx.rpm  # 查询软件包的依赖关系
```

安装升级删除

```bash
rpm -ivh xx.rpm --test  # 检查依赖关系, 不会真的安装
rpm -ivh xx.rpm   # 安装新的软件包
rpm -ivh xx.rpm --nodeps --force   # 强制安装, 忽略依赖缺失问题
rpm -ivh http://xxx/aaa.rpm    # 从网络安装

rpm -Uvh xx.rpm   # 升级一个软件包
rpm -Uvh --oldpackages xx.rpm   # 版本降级

rpm -e unzip-6.0-16.el7.x86_64
```





service

systemctl

init.d

journalctl



### export

```bash
export -p   # 列出当前的环境变量值
```



## 其他

### xargs
```bash
cat file.txt | xargs    # 所有内容单行输出
cat file.txt | xargs -n3    # 每3列一行输出
cat file.txt | xargs -n3 -d:    # 指定列分隔符为:
find . -name '.class' | xargs rm -v   # 删除所有claas文件
find . -name '.class' | xargs -p -n 1 rm -v   # 删除所有claas文件, 每次删除需要确认
find . -name '.class' | xargs -i rm -v {}   # -i 需要使用文件名替换{}, 使用小i默认占位符为{}
# 因为有些命令的变量在中间, 比如mv file /dir, 其中xx.tar变量, 而xargs默认是把变量放最后. 因此需要写成 ls *.tar | xargs -i mv {} /dir
find . -name '*.class' | xargs -0 -I [] rm -f []   # 自定义占位符为[], 使用文件名替换[], rm多次, 直接rm多个文件时, 可能会有错误. -0指使用\0作为占位符.
find . -type f "*.java" -print0 | xargs -0 wc -l   # 统计文件行数
# 如果文件名包含空格, xargs默认按空格分隔(file 1.log -> file 和 1.log), 此时后续命令会出错. 所以 print0会输出\0, 而在配合 -0 以\0为分隔符, 操作文件就没问题了
find . -maxdepth 1 ! -name "." -print0 | xargs -0 du -b | sort -nr | head -10 | nl   # 找到文件大小前10的, 文件名不为'.'的文件, nl可以为输出列加上编号
# find后执行xargs提示xargs: argument line too long解决方法
find . -type f -atime +0 -print0 | xargs -0 -l1 -t rm -f    # -1l是一次处理一个, -t是处理前打印命令
```



### wget 文件下载

```bash
wget www.xxx.com/test.html   # 获取网页内容并保存到test.html文件
wget -O file www.xx.com     # 保存到指定文件
wget -a file www.yy.com     # 追加到文件
wget -o log www.xx.com      # 下载记录保存到log文件
wget -i urls.txt -P path/dic   # 从文件读取链接, 下载到指定目录
wget -t 40 www.xx.com       # 最多重试40次(默认20)
wget -S www.xx.com/sdf      # 测试链接是否正确
wget --limit-rate=300k www.xx.com   # 限速下载
wget -b www.xx.com       # 后台下载
wget -c www.xx.com     # 断点续传
wget --limit-rate=300K      # 限速下载
wget --reject=gif www.xx.com    # 不下载gif文件
wget -r -A pdf www.xx.com   # 只下载pdf文件
wget --user=username --password=password https://example.com   # auth验证
```

### curl
```bash
# GET www.xx.com/search?a=aa&b=bb
curl -G -d 'a=aa' -d 'b=bb' www.xx.com/search
# URL 编码
curl -G --data-urlencode 'a=a bb' www.xx.com

# POST 
# 加 -d 后会自动加上请求头 Content-Type : application/x-www-form-urlencoded, 且请求转为 POST
curl -d '@data.txt' www.xx.com      # 读取文件内容作为请求体发送 
curl -d 'name=bob' www.xx.com/form  # post提交表单
curl -X POST --data "key=value" www.xx.com  # post发送参数
curl -X POST --date-urlencode "a=aa" www.xx.com        # 自动将发送的数据进行URL 编码
curl -d '{"a":"aa"}' -H 'Content-Type: application/json' www.xx.com   # 发送json请求

#上传
# -F 用来上传二进制文件, 会自动加上请求头Content-Type: multipart/form-data
curl -F 'file=@photo.png' www.xx.com
# 指定MIME类型, 默认是application/octet-stream
curl -F 'file=@photo.png;type=image/png' www.xx.com
curl -F 'file=@photo.png;filename=test.png' www.xx.com
# 下载
curl -O www.xx.com            # 以服务器上的名称保存
curl -o file www.xx.com       # 下载链接内容到指定文件
curl -O www.xx.com/pic[1-5].jpg     # 循环下载(pic1.jpg...pic5.jpg)
curl -o #1_#2.jpg ww.xxcom/{user,goods}/pic[1-5].jpg    # 循环下载并重命名
curl -# -O www.xx.com/pic.jpg       # 显示下载进度条
curl -C -O www.xx.com/pic.jpg       # 断点续传
# 分块下载
curl -r 0-100 -o pic_part1.jpg www.xx.com/pic.jpg
curl -r 100-200 -o pic_part2.jpg www.xx.com/pic.jpg
curl -r 200- -o pic_part3.jpg www.xx.com/pic.jpg
cat pic_part* > pic.jpg

# 发送HEAD请求
curl -I www.xx.com
curl --head www.xx.com

# 设置请求头
curl -H 'My-Header: 123' -X PUT www.xx.com      # 增加请求头, 指定方法, 等于--header

# 跳过SSL检测
curl -k https://www.xx.com

# cookie
curl --cookie "name=xxx" www.xx.com     # 设置cookie
curl -b cookie.txt www.xx.com       # 带上cookie访问
curl -b 'a=aa,b=bb' www.xx.com   
curl -c cookie.txt www.xx.com       # 将服务器设置的cookie信息保存到文件
curl -D header.txt www.xx.com       # 保存响应头到文件
curl -A "Mozilla/4.0 (compatible; MSIE 8.0; Windows NT 5.0)" http://www.linux.com

curl www.xx.com         # 打印html内容
curl -i www.xx.com      # 打印内容和响应头
curl -I www.xx.com      # 只显示响应头信息
curl --refere www.yy.com www.xx.com     # 设置来源
curl -L www.xx.com      # 自动跳转, 遇到301响应时
curl -s www.xx.com      # 不输出错误和进度信息 但会显示正常结果
curl -s -o /dev/null www.xx.com   # 不显示任何信息
curl -u 'admin:12345' www.xx.com  # 认证
curl -o /dev/null -s -w %{http_code} www.linux.com  # 测试网页响应code
curl -v www.xx.com      # 显示http通信完整过程, 包括端口和头信息, 用于调试
curl --trace output.txt www.xxx.com     # 更详细的信息
curl -x admin:12345@localhost:7890 www.xx.com  # 走代理
```



### scp 远程拷贝

```bash
# 从远程拷贝回本地
$ scp root@10.6.159.147:/opt/soft/demo.tar /opt/soft/

# 递归复制整个目录
$ scp -r root@10.6.159.147:/opt/soft/test /opt/soft/

# 上传到远程
$ scp /opt/soft/demo.tar root@10.6.159.147:/opt/soft/scptest
```



### crontab 定时任务

`-u user` :  对指定用户的crontab服务进行处理

```bash
*   *   *   *   *
分  时   天  月  周(1-6 0)

# 列出crontab文件
crontab -l
# 编辑
crontab -e
# 删除crontab文件
crontab -r

# 每分钟执行一次
* * * * * myCommand

# 每隔两天在上午8点到11的第3和第15分钟执行
3,15 8-11 */2  *  * myCommand
# 每周1上午8点到11的第3和第15分钟执行
3,15 8-11 * * 1 myCommand
# 每周六,周日的1:10重启smb
10 1 * * 6,0 /etc/init.d/smb restart
# 每天早上6点到12点, 每隔3个小时的0分钟执行
0 6-12/3 * 12 * myCommand
```

**注意**

- 脚本涉及到的文件路径要写全局路径

- 脚本用到java或其他环境变量时, 先通过`source`命令引入

  ```bash
  !/bin/sh
  source /etc/profile
  export RUN_CONF=/home/test/cbp.jboss.conf
  /usr/local/jboss-4.0.5/bin/run.sh -c mev &
  ```

- 当手动执行脚本OK, 但crontab不执行时, 很可能是环境变量的问题, 可尝试直接在crontab中引入环境变量

  ```bash
  0 * * * * . /etc/profile;/bin/sh /home/test/restart.sh
  ```

每次任务执行完, 系统会将输出信息通过邮件发给当前用户, 可以通过忽略日志输出避免

```bash
0 * * * * /xx.sh > /dev/null 2>&1
```

新创建的定时任务不会马上执行, 至少要过2分钟, 但重启cron则马上执行. 当crontab失效时可以尝试`/etc/init.d/crond restart`解决问题. 可以查看日志`/var/log/cron`定位问题.



### nohup

nohup 是 no hang up 的缩写，就是不挂断的意思。该命令可以在你退出帐户/关闭终端之后继续运行相应的进程。在缺省情况下该作业的所有输出都被重定向到当前目录的nohup.out的文件中, 如果当前目录的 nohup.out 文件不可写，输出重定向到 $HOME/nohup.out 文件中。

```bash
nohup java -jar xxx.jar > /dev/null 2>&1 &
```

`&` 在后台运行,  `2>&1` 是将标准出错重定向到标准输, `/dev/null 2>&1` 将标准输出和错误输出全部重定向到/dev/null中,也就是将产生的所有信息丢弃



## 系统监控

下面大部分命令属于`sysstat`软件包的一部分, 执行`yum install -y sysstat`

### mpstat  - cpu监控

每隔1s打印所有cpu使用情况, 打印10次

```bash
[root@demo ~]# mpstat -P ALL 1 10
Linux 3.10.0-862.14.4.el7.x86_64 (demo) 	2020年09月01日 	_x86_64_	(1 CPU)

10时07分12秒  CPU    %usr   %nice    %sys %iowait    %irq   %soft  %steal  %guest  %gnice   %idle
10时07分13秒  all    0.00    0.00    0.00    0.00    0.00    0.00    0.00    0.00    0.00  100.00
10时07分13秒    0    0.00    0.00    0.00    0.00    0.00    0.00    0.00    0.00    0.00  100.00
```

参数说明:

​	`-P`: `ALL` 或 `0 - CPU个数-1` 监控哪个cpu

结果说明:

```
%user      在间隔的时间段里，用户态的CPU时间(%)，不包含nice值为负进程  (usr/total)*100
%nice      在间隔的时间段里，nice值为负进程的CPU时间(%)   (nice/total)*100
%sys       在间隔的时间段里，内核时间(%)       (system/total)*100
%iowait    在间隔的时间段里，硬盘IO等待时间(%) (iowait/total)*100
%irq       在间隔的时间段里，硬中断时间(%)     (irq/total)*100
%soft      在间隔的时间段里，软中断时间(%)     (softirq/total)*100
%idle      在间隔的时间段里，CPU除去等待磁盘IO操作外的因为任何原因而空闲的时间闲置时间(%) (idle/total)*100
```


### vmstat 内存监控

```bash
# 每秒1次, 输出3次
$ vmstat 1 3
procs -----------memory---------- ---swap-- -----io---- -system-- ------cpu-----
 r  b   swpd   free   buff  cache   si   so    bi    bo   in   cs us sy id wa st
 1  0      0 8501324     40 11794708    0    0     0     5    0    0  1  0 98  0  0
```

- **procs** 进程
  - **r** 运行队列中进程数量
  - **b** 等待IO的进程数量
- **memory** 内存
  - **swpd** 使用虚拟内存大小
  - **free** 可用内存大小
  - **buff** 用作缓冲的大小
  - **cache** 用作缓存的大小
- **swap** 交换区
  - **si** 每秒从交换区写到内存的大小
  - **so** 每秒写入到交换区的大小
- **IO**
  - **bi** 每秒读取的块数   **这里的块不同于文件系统的块, 是操作系统内部用于缓存和缓冲操作的, 当前大小为1024Bytes**
  - **bo** 每秒写入的块数
- **system**  
  - **in**  每秒中断数 包括时钟中断
  - **cs** 每秒上下文切换数
- **CPU**
  - **us** 用户进程执行时间
  - **sy** 系统进程执行时间
  - **id** 空闲时间, 包括IO等待时间
  - **wa** 等待IO时间


### iostat IO监控

```bash
$ iostat 1 3
Linux 3.10.0-514.el7.x86_64 (dev) 	2020年10月09日 	_x86_64_	(16 CPU)

avg-cpu:  %user   %nice %system %iowait  %steal   %idle
           1.38    0.00    0.41    0.04    0.00   98.18

Device:            tps    kB_read/s    kB_wrtn/s    kB_read    kB_wrtn
vda               4.76         0.84        53.30   19037086 1211485860
```

CPU属性

- **%user** CPU在用户模式下的时间百分比
- **%nice** CPU在带NICE值的用户模式下的时间百分比
- **%system** CPU在系统模式下的时间百分比
- **%iowait** CPU等待输入输出完成的时间百分比
- **%steal** 管理程序维护另一个虚拟处理器时, 虚拟CPU的无意识等待时间百分比
- **%idle** CPU空闲时间百分比

> %iowait值过高表示磁盘存在IO瓶颈;  %idle值高表示CPU比较空闲;  如果%idle值高但系统响应慢, 可能是CPU等待内存分配; %idle持续低于10说明系统CPU处理能力不足

disk属性

- **tps** 设备每秒是IO请求数, 多个逻辑请求可能会被合并为一次IO请求, 一次IO请求的大小是未知的
- **kB_read/s** 每秒从设备读取的数据量
- **kB_wrtn/s** 每秒向设备写入的数据量
- **kB_read** 读取的数据总量
- **kB_wrtn** 写入的数据总量

```bash
# 显示更详细的io信息
$ iostat -x
...
Device:         rrqm/s   wrqm/s     r/s     w/s    rkB/s    wkB/s avgrq-sz avgqu-sz   await r_await w_await  svctm  %util
vda               0.00     0.12    0.02    4.74     0.84    53.32    22.76     0.04    7.76   14.59    7.73   1.36   0.65
```

`rrqm/s` 每秒进行 merge 的读操作数目。即 rmerge/s
`wrqm/s` 每秒进行 merge 的写操作数目。即 wmerge/s
`r/s` 每秒完成的读 I/O 设备次数。即 rio/s
`w/s` 每秒完成的写 I/O 设备次数。即 wio/s
`rsec/s` 每秒读扇区数。即 rsect/s
`wsec/s` 每秒写扇区数。即 wsect/s
`rkB/s` 每秒读K字节数。是 rsect/s 的一半，因为每扇区大小为512字节。
`wkB/s` 每秒写K字节数。是 wsect/s 的一半。
`avgrq-sz` 平均每次设备I/O操作的数据大小 (扇区)。
`avgqu-sz` 平均I/O队列长度。
`await` 平均每次设备I/O操作的等待时间 (毫秒)。
`svctm` 平均每次设备I/O操作的服务时间 (毫秒)。
`%util` 一秒中有百分之多少的时间用于 I/O 操作，即被io消耗的cpu百分比

> 如果 %util 接近 100%，说明产生的I/O请求太多，I/O系统已经满负荷，该磁盘可能存在瓶颈。如果 svctm 比较接近 await，说明 I/O 几乎没有等待时间；如果 await 远大于 svctm，说明I/O 队列太长，io响应太慢，则需要进行必要优化。如果avgqu-sz比较大，也表示有大量io在等待。



### pidstat - 进程资源占用监控

pidstat 是sysstat软件套件的一部分, CentOS使用该命令安装: `yum install sysstat`

**常用参数**

- `-u` 默认参数，显示各个进程的 CPU 统计信息
- `-r` 显示各个进程的内存使用情况
- `-d` 显示各个进程的 IO 使用
- `-w` 显示各个进程的上下文切换

显示所有进程的CPU使用信息, `pidstat` 或 `pidstat -u -p ALL`

```bash
$ pidstat
Linux 3.10.0-862.14.4.el7.x86_64 (demo) 	2020年09月01日 	_x86_64_	(1 CPU)

10时14分30秒   UID       PID    %usr %system  %guest    %CPU   CPU  Command
10时14分30秒     0         1    0.01    0.01    0.00    0.01     0  systemd
10时14分30秒     0         2    0.00    0.00    0.00    0.00     0  kthreadd
...

# 指定进程
pidstat -p 123

# 间隔1s, 显示3次
pidstat 1 3

```

`PID` 进程ID
`%usr` 进程在用户空间占用cpu的百分比  `%system` 进程在内核空间占用cpu的百分比
`%guest` 进程在虚拟机占用cpu的百分比
`%CPU` 进程占用cpu的百分比
`CPU` 处理进程的cpu编号
`Command` 当前进程对应的命令

#### 查看内存

```bash
$ pidstat -r -p 1205
10时22分46秒   UID       PID  minflt/s  majflt/s     VSZ    RSS   %MEM  Command
10时22分47秒     0      1205      0.00      0.00   47448   2672   0.14  nginx
```

`PID` 进程标识符
`Minflt/s` 任务每秒发生的次要错误，不需要从磁盘中加载页
`Majflt/s` 任务每秒发生的主要错误，需要从磁盘中加载页
`VSZ` 虚拟地址大小，虚拟内存的使用KB
`RSS` 常驻集合大小，非交换区五里内存使用KB
`Command` 当前进程对应的命令

#### 查看io

```bash
$ pidstat -d -p 1205
10时23分11秒   UID       PID   kB_rd/s   kB_wr/s kB_ccwr/s  Command
10时23分12秒     0      1205      0.00      0.00      0.00  nginx
```

`PID` 进程id
`kB_rd/s` 每秒从磁盘读取的KB
`kB_wr/s` 每秒写入磁盘KB
`kB_ccwr/s` 任务取消的写入磁盘的KB。当任务截断脏的pagecache的时候会发生
`Command` 当前进程对应的命令

#### 查看上下文切换

```bash
$ pidstat -w -p 4846
18时29分18秒   UID       PID   cswch/s nvcswch/s  Command
18时29分18秒     0      4846      0.00      0.00  java
```

`PID` 进程id
`Cswch/s` 每秒主动任务上下文切换数量
`Nvcswch/s` 每秒被动任务上下文切换数量
`Command` 当前进程对应的命令


#### 查看线程信息

```bash
$ pidstat -t -p 4846
18时33分00秒   UID      TGID       TID    %usr %system  %guest    %CPU   CPU  Command
18时33分00秒     0      4846         -    0.00    0.00    0.00    0.00    13  java
18时33分00秒     0         -      4846    0.00    0.00    0.00    0.00    13  |__java
18时33分00秒     0         -      4849    0.00    0.00    0.00    0.00    15  |__java
...
```

`PID` 进程ID
`%usr` 进程在用户空间占用cpu的百分比  `%system` 进程在内核空间占用cpu的百分比
`%guest` 进程在虚拟机占用cpu的百分比
`%CPU` 进程占用cpu的百分比
`CPU` 处理进程的cpu编号
`Command` 当前进程对应的命令



### sar - 系统活动报告

统计结果可以输出到文件

```bash
sar -o sarfile.log -u 1 3
```

#### CPU使用率

```bash
$ sar -u 1 3
15时31分47秒     CPU     %user     %nice   %system   %iowait    %steal     %idle
15时31分48秒     all      1.00      0.00      0.06      0.00      0.00     98.93
```

`CPU`: all表示统计的所有CPU
`%user`: 用户级运行cpu占比   `%nice`:用户级别用于nice操作CPU占比 `%system`: 核心级运行CPU占比
`%iowait`: 用于等待IO操作占用CPU时间的百分比
`%steal`: 管理程序(hypervisor 虚拟化)为另一个虚拟进程提供服务而等待虚拟 CPU 的百分比
`%idle`: 显示CPU空闲占比, 若%iowait过高则表示磁盘存在IO瓶颈; 若%idel高但系统响应慢则可能是内存不足, cpu在等待分配内存; 若%idle持续<1则系统cpu处理能力不足

#### 内存使用率

```bash
$ sar -r 1 3
15时30分02秒 kbmemfree kbmemused  %memused kbbuffers  kbcached  kbcommit   %commit  kbactive   kbinact   kbdirty
15时30分03秒  30212716  35595764     54.09        40   4453844  40602136     61.70  30280872   3281308       240
```

`kbmemfree`: 可用内存, 不含buffer/cache   `kbmemused`: 已用内存, 包含buffer/cache
`%memused`: kbmemused占总内存百分比
`kbbuffers`: buffer空间大小  `kbcached`:cache空间大小
`kbcommit`: 为了确保不溢出而需要的内存(RAM + swap)  `%commit`: kbcommit占比
`kbactive`: 最近使用的不被回收的   `kbinact`: 不经常使用的容易被回收  `kbdirty`: 赃页,等待写入磁盘

#### CPU负载

```bash
$ sar -q 1 3
15时49分19秒   runq-sz  plist-sz   ldavg-1   ldavg-5  ldavg-15   blocked
15时49分20秒         0       832      0.00      0.01      0.05         0
```

`runq-sz`:  运行队列长度, 即等待运行的进程数
`plist-sz`:  进程列表中进程和线程的数量
`ldavg-1` `ldavg-5` `ldavg-15` :  过去1/5/15分钟系统的平均负载
`blocked`: 正在等待io的任务数量

#### 内存页状态

```bash
$ sar -B 1 3
15时33分35秒  pgpgin/s pgpgout/s   fault/s  majflt/s  pgfree/s pgscank/s pgscand/s pgsteal/s    %vmeff
15时33分36秒      0.00     18.00     31.00      0.00     82.00      0.00      0.00      0.00      0.00
```

`pgpgin/s` `pgpgout/s` : 每秒从磁盘换入/出到内存的字节数(KB)
`fault/s`: 每秒系统产生的缺页数, 即major+minor;  `majflt/s`: 每秒产生的主缺页数
`pgfree/s`: 每秒被放入空闲队列的页个数  `pgscank/s`: 每秒被kswapd扫描的页数
`pgscand/s`: 每秒直接被扫描的   `pgsteal/s`: 每秒从cache中被清除用来满足内存需要的页数
`%vmeff`: 每秒清除的页(pgsteal)占总扫描页(pgscank+pgscand)

#### swap信息

```bash
$ sar -W 1 3
15时53分10秒  pswpin/s pswpout/s
15时53分11秒      0.00      0.00
```

`pswpin/s` `pswpout/s` : 每秒换入/出的交换页面(swap page)数量

#### IO和传输速率

```bash
$ sar -b 1 3
15时37分39秒       tps      rtps      wtps   bread/s   bwrtn/s
15时37分40秒      2.00      0.00      2.00      0.00     16.00
```

`tps`: 每秒物理设备的IO请求总数
`rtps` `wtps`: 每秒读/写物理设备的io请求数
`bread/s` `bwrtn/s`: 每秒读写物理设备的数据量, 单位: 块/s, 这里的块等于扇区, 所有大小为512Bytes

文件系统的块大小可以通过`blockdev --getbsz /dev/vda `或者 `stat -f /dev/vda` 查看(块是文件系统里的概念, 扇区(`fdisk- l`)是磁盘里的概念, 一般块是扇区的2^n倍)

#### 块设备信息

```bash
$ sar -d 1 3
平均时间:       DEV       tps  rd_sec/s  wr_sec/s  avgrq-sz  avgqu-sz     await     svctm     %util
平均时间:  dev253-0      0.67      0.00     13.33     20.00      0.00      0.50      0.50      0.03
```

`tps`: 每秒请求物理磁盘的次数, 多个逻辑请求会被合并成1个io请求, 一次传输的大小是不确定的
`rd_sec/s` `wr_sec/s` : 每秒读写的扇区数量, **扇区大小512Bytes**
`avgrq-sz` : 平均每次io请求的扇区大小
`avgqu-sz` : 磁盘请求队列的平均长度
`await`: 从请求磁盘操作到系统完成处理, 每次请求的平均消耗时间(毫秒), 包括请求队列等待时间
`svctm` : 系统处理每次请求的平均时间(毫秒), 不包括在请求队列中消耗的时间
`%util`:  I/O请求占CPU的百分比, 比率越大, 说明设备带宽使用率越高

#### 网卡流量

`-n`参数有6个开关:

- DEV显示网络接口信息。
- EDEV显示关于网络错误的统计数据。
- NFS统计活动的NFS客户端的信息。
- NFSD统计NFS服务器的信息
- SOCK显示套接字信息
- ALL显示所有5个开关

```bash
$ sar -n DEV 1 2
16时08分55秒     IFACE   rxpck/s   txpck/s    rxkB/s    txkB/s   rxcmp/s   txcmp/s  rxmcst/s
16时08分56秒      eth0     88.00    683.00      5.67   1008.74      0.00      0.00      0.00

# 查看xx日的网卡流量历史
$ sar -n DEV -f /var/log/sa/saxx
```

`IFACE` ：LAN接口
`rxpck/s`  `txpck/s` ：每秒钟接收/发送的数据包
`rxkB/s` `txkB/s`：每秒钟接收/发送的字节数 千字节/s
`rxcmp/s` `txcmp/s` ：每秒钟接收/发送的压缩数据包
`rxmcst/s` ：每秒钟接收的多播数据包

#### 统计网络设备通信失败信息

```bash
$ sar -n EDEV  1 3
17时04分54秒     IFACE   rxerr/s   txerr/s    coll/s  rxdrop/s  txdrop/s  txcarr/s  rxfram/s  rxfifo/s  txfifo/s
17时04分55秒 enp95s0f0      0.00      0.00      0.00      0.00      0.00      0.00      0.00      0.00      0.00
17时04分55秒      eno1      0.00      0.00      0.00      0.00      0.00      0.00      0.00      0.00      0.00
```

结果

`rxerr/s`   `txerr/s`  ：每秒钟接收/发送的坏数据包：每秒钟的坏数据包
`coll/s` ：每秒冲突数
`rxdrop/s`   `txdrop/s`  ：因为缓冲充满，每秒钟丢弃的已接收/发送数据包数
`txcarr/s` ：发送数据包时，每秒载波错误数\
`rxfram/s` ：每秒接收数据包的帧对齐错误数
`rxfifo/s`   `txfifo/s` ：接收/发送的数据包每秒FIFO过速的错误数

#### IP层统计

```bash
$ sar -n IP 1 3
17时20分06秒    irec/s  fwddgm/s    idel/s     orq/s   asmrq/s   asmok/s  fragok/s fragcrt/s
17时20分07秒     25.00      0.00     25.00     24.00      0.00      0.00      0.00      0.00
```

#### IP层错误统计

```bash
$ sar -n EIP 1 3
17时21分01秒 ihdrerr/s iadrerr/s iukwnpr/s   idisc/s   odisc/s   onort/s    asmf/s   fragf/s
17时21分02秒      0.00      0.00      0.00      0.00      0.00      0.00      0.00      0.00
```

#### TCP连接信息

```bash
$ sar -n TCP 1 3
17时16分00秒  active/s passive/s    iseg/s    oseg/s
17时16分01秒      3.00      1.00     37.00     45.00
```

`active/s` : 新的主动连接   `passive/s` : 新的被动连接
`iseg/s` : 接受的段   `oseg/s` : 输出的段

#### TCP层错误统计

```bash
$ sar -n ETCP 1 3
16时17分04秒  atmptf/s  estres/s retrans/s isegerr/s   orsts/s
16时17分05秒      0.00      0.00      0.00      0.00      1.00

# 可以结合使用
$ sar -n DEV -n ETCP
```

`atmptf/s` : 每秒重试失败数      `estres/s` : 每秒断开连接数 

`retrans/s` : 每秒重传数         `isegerr/s` :  每秒错误数       `orsts/s` : 每秒RST数

#### socket连接信息

```bash
$ sar -n SOCK 1 3
17时11分06秒    totsck    tcpsck    udpsck    rawsck   ip-frag    tcp-tw
17时11分07秒      2312       701         3         0         0        38
```

`totsck` : 当前被使用的socket总数
`tcpsck` : 当前正在被使用的TCP的socket总数
`udpsck` :  当前正在被使用的UDP的socket总数
`rawsck` : 当前正在被使用于RAW的skcket总数
`if-frag`  : 当前的IP分片的数目
`tcp-tw` : TCP套接字中处于TIME-WAIT状态的连接数量



### pstack 跟踪进程堆栈

pstack 命令必须由相应进程的属主或 root 运行, 可以使用 pstack 来确定进程挂起的位置, 可以在一段时间内，多执行几次pstack，若发现代码栈总是停在同一个位置，那个位置就需要重点关注，很可能就是出问题的地方

```bash
$ pstack 4846
Thread 72 (Thread 0x2b3304708700 (LWP 4849)):
#0  0x00002b330487b965 in pthread_cond_wait@@GLIBC_2.3.2 () from /lib64/libpthread.so.0
#1  0x00002b3305b89ec3 in os::PlatformEvent::park() () from /iflytek/server/jdk1.8.0_71/jre/lib/amd64/server/libjvm.so
#2  0x00002b3305b4abb7 in Monitor::IWait(Thread*, long) () from /iflytek/server/jdk1.8.0_71/jre/lib/amd64/server/libjvm.so
#3  0x00002b3305b4b380 in Monitor::wait(bool, long, bool) () from /iflytek/server/jdk1.8.0_71/jre/lib/amd64/server/libjvm.so
#4  0x00002b3305cdcb11 in Threads::destroy_vm() () from /iflytek/server/jdk1.8.0_71/jre/lib/amd64/server/libjvm.so
#5  0x00002b330593d421 in jni_DestroyJavaVM () from /iflytek/server/jdk1.8.0_71/jre/lib/amd64/server/libjvm.so
#6  0x00002b3304a9363d in JavaMain () from /iflytek/server/jdk1.8.0_71/bin/../lib/amd64/jli/libjli.so
#7  0x00002b3304877dd5 in start_thread () from /lib64/libpthread.so.0
#8  0x00002b3304fa402d in clone () from /lib64/libc.so.6
Thread 71 (Thread 0x2b3304809700 (LWP 4850)):
...
```



### strace 跟踪进程的系统调用

```bash
$ strace cat /dev/null
execve("/bin/cat", ["cat", "/dev/null"], [/* 32 vars */]) = 0
brk(0)                                  = 0x1d92000
mmap(NULL, 4096, PROT_READ|PROT_WRITE, MAP_PRIVATE|MAP_ANONYMOUS, -1, 0) = 0x2b0893333000
access("/etc/ld.so.preload", R_OK)      = -1 ENOENT (No such file or directory)
...

# 跟踪可执行程序 -f -F: 同时跟踪fork和vfork出来的进程, -o: 把输出写入文件
strace -f -F -o straceout.txt mysql

# 跟踪指定进程的所有系统调用 并统计耗时 以及开始时间
strace -o straceout.txt -T -tt -e trace=all -p 1234
```





### lsof 一切皆文件

用于查看打开文件的进程, 进程打开了哪些文件, 端口等. 由于lsof命令需要访问核心内存和各种文件, 所以需要root用户执行.

```bash
$ lsof /iflytek/server/jdk1.8.0_71/bin/java
COMMAND   PID USER  FD   TYPE DEVICE SIZE/OFF      NODE NAME
java     5401 root txt    REG 253,17     7734 805307256 /iflytek/server/jdk1.8.0_71/bin/java
```

`COMMAND`  : 进程名称   `PID` : 进程id   `USER` :  进程所有者   `FD` : 文件描述符  `TYPE` : 文件类型  `DEVICE`:  磁盘名称  `SIZE` : 文件大小  `NODE` :  索引节点(文件在磁盘上的标识)  `NAME` : 被打开文件的确切名称

```bash
# 查询某个用户打开的文件
lsof -u username

# 列出指定进程打开的文件
lsof -p PID

# 列出打开文件的进程
lsof FILENAME

# 显示打开80端口的进程
lsof -i :80
# 所有TCP连接
lsof -i tcp
# 列出所有IPv4连接
lsof -i 4
# 连接到指定主机的连接
lsof -i @172.31.47.13
# 连接到指定主机端口的连接
lsof -i @172.31.47.13:8085

# 列出正在等待连接的端口
lsof -i -sTCP:LISTEN
lsof -i | grep LISTEN

# 杀死用户运行的所有进程
kill -9 `lsof -t -u daniel`

# 根据文件描述列出对应的文件信息
lsof -d 5 | grep nginx
COMMAND     PID           USER   FD      TYPE             DEVICE  SIZE/OFF      NODE NAME
nginx      4072           root    5w      REG             253,17 141778115 808976418 /iflytek/server/nginx-1.10.2/logs/access.log

# 列出被指定进程号打开的所有IPv4网络文件
lsof -i 4 -nap 20531
java    20531 root   40u  IPv4 627722714      0t0  TCP 172.31.46.2:40646->172.31.46.2:mysql (ESTABLISHED)
java    20531 root   41u  IPv4 627761916      0t0  TCP 172.31.46.2:9890->172.31.46.111:34202 (ESTABLISHED)

# 列出指定主机上的指定端口相关的所有文件信息, 3秒刷新一次
lsof -i@172.31.46.2:6379,3306 -r 3
```


### ping

```bash
ping www.baidu.com
ping -c 10 192.168.1.101    # 指定次数
ping -c 10 -i 0.5 192.168.1.101     # 指定次数和间隔(秒)
ping -i 3 -s 1024 -t 255 192.168.1.101  # -s 发送包大小为1024字节 -t TTL为255
```


### free 查询可用内存

```bash
free -h
# -b -k -m -g : 以byte, KB, MB, GB为单位
free    
#              total       used       free     shared    buffers      cached
# Mem:        516372     293056     223316        152      75256     160428
# -/+ buffers/cache:      57372     459000
# Swap:       135164          0     135164
# total :总计物理内存,  used :已使用  free:可使用  shared :多个进程共享的  buffers/cached :磁盘缓存的大小
# swap :交换分区, 即虚拟缓存
free -s 10  # 每10秒刷新一次
```


### df  查询磁盘可用空间

```bash
$ df -h
Filesystem      Size  Used Avail Use% Mounted on
/dev/vda1       237G  7.9G  219G    4% /
devtmpfs         48G     0   48G    0% /dev
tmpfs            48G     0   48G    0% /dev/shm
tmpfs            48G  788K   48G    1% /run
tmpfs            48G     0   48G    0% /sys/fs/cgroup
tmpfs           9.5G     0  9.5G    0% /run/user/0
/dev/vdb1       2.9T  4.0G  2.8T    1% /iflytek

# 第一列代表文件系统对应的设备文件的路径名, 一般是分区, 第二列表示分区包含的数据块(1024字节)数目, 第3, 4列表示已用 和 可用空间. 第3, 4列加一起不等于第二列是因为每个分区保留了少量空间给管理员使用. 最后一列是挂载点.

df -t ext3      # 指定类型的磁盘
df -T       # 列出文件系统的类型
```

### du 磁盘使用统计

```bash
# 统计当前目录下各文件/目录占用磁盘大小
$ du -sh *
11M	fastdfs-5.08
146M	ffmpeg-4.1.4-amd64-static
339M	jdk1.8.0_71
```

### 



[Linux工具快速教程](https://linuxtools-rst.readthedocs.io/zh_CN/latest/index.html)

[CentOS7.5 系统最小化安装与初始化配置](https://www.cnblogs.com/tssc/p/11041464.html)