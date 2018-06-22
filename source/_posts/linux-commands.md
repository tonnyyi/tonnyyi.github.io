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

### ls
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
### mkdir
```bash
mkdir dir   # 创建目录
mkdir -p path/to/dir       # 递归创建多个目录
mkdir -m 777 dir    # 创建权限为777的目录
mkdir -p dir, dir2, path/to/dir3    # 创建多个目录
```

### rm
```bash
rm file      # 删除前需要确认
rm -f file    # 强行删除, 系统不再提示
rm -i file*     # 删除多个文件, 逐个确认
rm -r dir      # 递归删除文件夹下所有文件, 并删除该文件夹
rm -rf dir     # 毁天灭地
```

### cp 
```bash
cp file1 path/          # 复制到目录, 保持文件名
cp file1 path/file2     # 复制并重命名
cp -r path/dir1 path/to/dir2    # 递归复制一个目录
cp -i file1 file2    # 重命名文件为file2, 如果file2已存在则提示是否覆盖
cp -a dir1 dir2     # 如果dir2已存在, 则将dir1复制到dir2目录里, 否则dir1并重命名为dir2
cp -b file1 file2    # file2已存在时, 会先备份, 再复制
```

### mv
```bash
mv -i file1 file2   # 如果file2已存在则提示
mv -f file1 file2   # 如果file2已存在直接覆盖, 不提示
mv -b file1 file2   # 如果file2已存在, 则覆盖, 但会先备份
mv file1 file2 dir  # 移动多个文件到dir
```

### chomd
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

### chown
```bash
chown root file     # 修改文件所属人
chown redis:redis file      # 修改文件所属人及组
chown root: file    # 修改文件所属人为root, 组改为root所在组
chown :mail file    # 仅修改组
chown -R root dir   # 递归修改所属人
```

### find
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

## 文件内容

### cat
```bash
cat file1 file2    # 顺序打印多个文件内容
cat -n file        # 每行前加行号
cat -b file        # 空白行不加行号
cat file1 > file2  # 将file1的内容输入到file2, file2原有内容被清空
cat file2 >> file2  # file1文件追加到file2
```

### grep
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

### tar
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

### tail/head
```bash
head file       # 默认展示前10行
head -5 file    # 前5行
head -n 5 file      # 前5行
head -c 5 file      # 前5个字符
head -n -5 file     # 除了最后5行的其他行
head -n 20 file | tail -n 10    # 查看前20行中的后10行
```

### less/more
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

### top
```
[root@host ~]# top
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

### ps
```bash
# linux进程状态: 运行(R), 可中断的睡眠状态(S), 不可中断的睡眠状态(D), 僵尸(Z), 停止(T), 退出,即将被销毁(X)
# 输出列含义: PID: 进程id  %CPU:使用CPU资源占比  %MEM:所占内存百分比  VSZ:用掉的虚拟内容(KB)  RSS:占用的固定的内存(KB)  TTY:在哪个终端上运作  STAT:当前状态 TIME:用掉的CPU时间  CMD:使用的命令
ps -aux     # 显示所有运行的进程
ps auxww    # 包含完整的命令路径
ps --user $(id -u) f       # 树状展示当前用户的进程
ps -o ppid= -p pid      # 展示摸个进程的父进程id
ps -ef
ps -u root  # 
```

### ifconfig
```bash
ifconfig etho0 192.168.1.110    # 配置IP地址
```

### netstat
```bash
netstat -nau | grep 3306    
lsof -i:8080    # mac上使用
```

### ping
```bash
ping www.baidu.com
ping -c 10 192.168.1.101    # 指定次数
ping -c 10 -i 0.5 192.168.1.101     # 指定次数和间隔(秒)
ping -i 3 -s 1024 -t 255 192.168.1.101  # -s 发送包大小为1024字节 -t TTL为255
```

### free
```bash
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

### df
```bash
df      
# Filesystem      Size  Used Avail Use% Mounted on
# /dev/sda2        11G  1.2G  8.8G  12% /
# 第一列代表文件系统对应的设备文件的路径名, 一般是分区, 第二列表示分区包含的数据块(1024字节)数目, 第3, 4列表示已用 和 可用的数据块数目. 第3, 4列加一起不等于第二列是因为每个分区保留了少量空间给管理员使用. 最后一列是挂载点.
df -t ext3      # 指定类型的磁盘
df -T       # 列出文件系统的类型
df -h   # 单位可读(M, G)
```

### mount
```bash

```

### yum
```bash

```

### rpm
```bash

```


### date
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

### wget
```bash
wget www.xxx.com/test.html   # 获取网页内容并保存到test.html文件
wget -O file www.xx.com     # 保存到指定文件
wget -a file www.yy.com     # 追加到文件
wget -o log www.xx.com      # 下载记录保存到log文件
wget -i urls.txt    # 从文件读取链接, 下载
wget -t 40 www.xx.com       # 最多重试40次(默认20)
wget -S www.xx.com/sdf      # 测试链接是否正确
wget -b www.xx.com       # 后台下载
wget - c www.xx.com     # 断点续传
wget --limit-rate=300K      # 限速下载
wget --reject=gif www.xx.com    # 不下载gif文件
wget -r -A pdf www.xx.com   # 只下载pdf文件
```

### curl
```bash
curl www.xx.com     # 打印html内容
curl -i www.xx.com  # 打印内容和响应头
curl -I www.xx.com      # 只显示响应头信息
curl -v www.xx.com      # 显示http通信完整过程, 包括端口和头信息
curl --trace output.txt www.xxx.com     # 更详细的信息
curl --refere www.yy.com www.xx.com     # 设置来源
curl -L www.xx.com  # 自动跳转, 遇到301响应时

curl -O www.xx.com      # 以服务器上的名称保存
curl www.xx.com -o file     # 下载链接内容到指定文件
curl -O www.xx.com/pic[1-5].jpg     # 循环下载(pic1.jpg...pic5.jpg)
curl -o #1_#2.jpg ww.xxcom/{user,goods}/pic[1-5].jpg    # 循环下载并重命名
curl -# -O www.xx.com/pic.jpg       # 显示下载进度条
curl -C -O www.xx.com/pic.jpg       # 断点续传
# 分块下载
curl -r 0-100 -o pic_part1.jpg www.xx.com/pic.jpg
curl -r 100-200 -o pic_part2.jpg www.xx.com/pic.jpg
curl -r 200- -o pic_part3.jpg www.xx.com/pic.jpg
cat pic_part* > pic.jpg

curl -d 'name=bob' www.xx.com/form  # post提交表单
curl -X POST --data "key=value" www.xx.com  # post发送参数
curl -X POST --date-urlencode "key=value" www.xx.com        # curl帮忙转码
curl -H 'My-Header: 123' -X PUT www.xx.com      # 增加请求头, 指定方法, 等于--header
curl -d '{"name":"bob"}' -H 'Content-Type: application/json' wwx.xx.com     # 发送json请求
curl --form upload=@localFilename --form press=OK www.xx.com    # 文件上传
curl -o /dev/null -s -w %{http_code} www.linux.com  # 测试网页响应code

curl --cookie "name=xxx" www.xx.com     # 设置cookie
curl -c cookie.txt www.xx.com      # 保存cookie信息到文件
curl -b cookie.txt www.xx.com       # 带上cookie访问
curl -D header.txt www.xx.com       # 保存响应头到文件
curl -A "Mozilla/4.0 (compatible; MSIE 8.0; Windows NT 5.0)" http://www.linux.com       # 指定UA
# mac-chrome : Mozilla/5.0 (Macintosh; Intel Mac OS X 10_13_2) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/64.0.3282.186 Safari/537.36
```

### a.sh > xx.log 2>&1
```bash
echo log > /dev/null 2>&1
```
\> :代表重定向到哪里，例如：echo "123" > /home/123.txt
1 :表示stdout标准输出，系统默认值是1，所以">/dev/null"等同于"1>/dev/null"
2 :表示stderr标准错误
& :表示等同于的意思，2>&1，表示2的输出重定向等同于1

1 > /dev/null 2>&1 语句含义：

1 > /dev/null ： 首先表示标准输出重定向到空设备文件，也就是不输出任何信息到终端，说白了就是不显示任何信息。
2>&1 ：接着，标准错误输出重定向（等同于）标准输出，因为之前标准输出已经重定向到了空设备文件，所以标准错误输出也重定向到空设备文件。

在shell中, 每个进程都和三个系统文件相关联: 标准输入stdin, 标准输出stdout 和 标准错误stderr, 三个系统文件的文件描述符分别为0, 1, 2. 所以这里 2>&1 的意思就是将标准错误也输出到标准输出当中


