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

### uniq

```bash
uniq -c     # 显示重复次数
uniq -U     # 仅显示出现一次的列
uniq -d 	# 仅显示重复的列
```

## wc

```bash
wc -l	# 统计行数
wc -c	# 统计字节数
wc -m	# 统计字符数， 不能与 -c 一起用
wc -w	# 统计字数，由空白、空格或换行符分隔
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
tar cvf xxx.tar dirname/	   # 压缩文件夹
tar cvf xxx.tar file1 file2	   # 压缩多个文件
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
tar xvf xxx.tar -C path/dir		# 解压到指定文件夹

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
# /xxx :向前查找， 按n/N查找下/上一个结果       ?xxx :后查找
history | less
less -N xxx.txt    # 显示行号
less -m xxx.txt    # 显示百分比
```



### sed 基于正则的流处理器

sed是一种在线编辑器, 每次处理一行内容. 处理时, 把当前行存储在临时缓冲区中, 称为"模式空间"(pattern space), 接着sed命令处理缓冲区中的内容. 处理完成后, 将缓冲区内容输出到屏幕. 然后接着处理下一行, 直至文件结束.

##### 打印   p参数

```bash
# 匹配fish并输出, 但是匹配到的行被输出了2遍, 这是因为sed会把待处理的信息也输出了
sed '/fish/p' input.txt

# 使用-n参数只输出匹配到的行
sed -n '/fish/p' input.txt
# 输出匹配 fish 或者 dog 的行
sed -n '/fish/,/dog/p' input.txt

# 从第一行打印到匹配fish的那一行
sed -n '1,/fish/p' input.txt

# 打印匹配到dog后的3行
sed -n '/dog/,+3p' input.txt
```

##### 文本替换  s参数

```bash
# 把一行上的每个(/g)单词 my 都替换成 your, 所有行都处理
# 但不会改动input.txt, 只是会输出每一行的内容(即使该行没被处理也输出)
sed 's/my/your/g' input.txt

# 如果想用处理后的内容覆盖原文件, 则需要加 -i 参数
sed -i 's/my/your/g' input.txt

# 在每行前面增加 # 
sed -i 's/^/#/g' input.txt
# 在每行后面增加 ===
sed -i 's/$/===/g' input.txt

# 指定处理的行范围(第3到6行)
sed -i '3,6s/my/your/g' input.txt
# 从匹配到的行开始后面连续3行, 在行首加#
sed -i '/my/,+3s/^/#/g' input.txt

# 只替换每行的第一个a
sed -i 's/a/A/1' input.txt
# 替换每行的第二个a
sed -i 's/a/A/2' input.txt
# 替换每行的第三个以后的所有a
sed -i 's/a/A/3g' input.txt

# 可以一次匹配多个模式, 先把第一到第三行的my替换成your, 再把第三行以后的This替换成That
sed '1,3s/my/your/g; 3,$s/This/That/g' input.txt
sed -e '1,3s/my/your/g' -e '3,$s/This/That/g' input.txt

# 可以使用&表示被匹配到的变量, 将单词my用[]包裹起来
sed 's/my/[&]/g' input.txt

# 使用正则中的分组功能, 圆括号定义分组, \1, \2表示匹配到的分组变量
sed 's/my (\w+)/\1/g' input.txt  # 输出my后面跟着的字符

# N命令可以把下一行内容当成缓冲区做匹配, 效果就是隔行处理
sed 'N;s/\n/,/' input.txt  # 把1,2  3,4  5,6 合并成一行, 用,分隔
```

##### 地址表示法

| 表达式      | 说明                                                 |
| ----------- | ---------------------------------------------------- |
| n           | 行号                                                 |
| $           | 最后一行                                             |
| /regexp/    | 所有匹配该正则的文本行                               |
| addr1,addr2 | 从addr1到addr2范围内的文本行, 包含addr2              |
| first~step  | 从first行开始, 每个step的文本行, 例: 1~2值每个奇数行 |
| addr1,+n    | 从addr1开始到后面的n行                               |
| addr!       | 除了addr之外的其他文本行                             |

##### 在行前插入一行   i参数

```bash
# 在第一行前插入一行 This is a test line
sed '1 i This is a test line' input.txt
```

##### 在行后插入一行   a参数

```bash
# 在第一行后插入一行 This is a test line
sed '1 a This is a test line' input.txt

# 匹配到fish就在这一行后面追加一行
sed '/fish/a This is a test line' input.txt
```

##### 替换匹配行    c参数

```bash
# 替换第二行
sed '2 c This is a test line' input.txt
```

##### 删除匹配行   d参数

```bash
# 删除1到3行
sed '1,3d' input.txt
# 从第2行到最后全部删除
sed '2,$d' input.txt

# 删除含有fish的行
sed '/fish/d' input.txt
```

##### 命令打包

```bash
# 对3到6行, 匹配到This就删除这一行
sed '3,6 {/This/d}' input.txt

# 对3到6行, 匹配This后再匹配fish, 匹配到之后删除这一行
sed '3,6 {/This/{/fish/d}}' input.txt

# 从第一行到最后, 如果匹配到This则删除, 如果前面有空格则去除空格
sed '1,$ {/This/d; s/^ *//g}' input.txt
```



### awk 格式化流处理器

```bash
# 打印第1列和第3列   $0表示整行
cat '{print $1, $4}' input.txt

# 格式化打印
awk '{printf "%-8s %-8s %-8s %-18s %-22s %-15s\n",$1,$2,$3,$4,$5,$6}' input.txt

#从file文件中找出长度大于80的行
awk 'length>80' file

#按连接数查看客户端IP
netstat -ntu | awk '{print $5}' | cut -d: -f1 | sort | uniq -c | sort -nr

#打印99乘法表
seq 9 | sed 'H;g' | awk -v RS='' '{for(i=1;i<=NF;i++)printf("%dx%d=%d%s", i, NR, i*NR, i==NR?"\n":"\t")}'
```

##### 过滤记录

```bash
# 第3列等于0 且 第6列等于LISTEN  还有 !=  <  <=  >  >=
awk '$3==0 && $6=="LISTEN"' input.txt

# 过滤后只输出第7列
awk '$3==0 && $6=="LISTEN" {print $7}' input.txt

# 同时输出表头, 引入内建变量NR
awk '$3==0 && $6=="LISTEN" || NR==1' input.txt
# 再加上格式化输出
awk '$3==0 && $6=="LISTEN" || NR==1 {printf "%-20s %-20s %s\n",$4,$5,$6}' input.txt
```

##### 内建变量

| 变量名   | 说明                                                         |
| -------- | ------------------------------------------------------------ |
| $0       | 当前记录, 即整行内容                                         |
| $1 ~ $n  | 当前记录的第n个字段, 字段间由FS分隔                          |
| FS       | 输入字段分隔符, 默认是空格或Tab                              |
| NF       | 当前记录的字段个数, 即有多少列                               |
| NR       | 已经读出的记录数, 即行号, 从1开始, 如果有多个文件, 该值会不断累加 |
| FNR      | 当前记录数, 与NR不同的是, 该值是各个文件自己的行号           |
| RS       | 输入的记录分隔符, 默认为换行符                               |
| OFS      | 输入字段分隔符, 默认为空格                                   |
| ORS      | 输出记录分隔符, 默认为换行符                                 |
| FILENAME | 当前输入文件的名称                                           |

```bash
awk '$1=="tcp6" && $6=="LISTEN" {printf "%s %-20s %s\n", NR,$4,$5}' input.txt
```

##### 指定分隔符

```bash
awk 'BEGIN{FS=":"} {print $1,$3,$6}' /etc/passwd
# 等价于
awk -F: '{print $1,$3,$6}' /etc/passwd
# 指定多个分隔符
awk -F '[;:]'  '{print $1,$3,$6}' /etc/passwd

# 指定输出分隔符
awk -F: '{print $1,$3,$6}' OFS="\t" /etc/passwd
```

##### 字符串匹配

```bash
# 过滤第6列匹配WAIT   ~ 表示模式开始   //中是模式
awk '$6 ~ /WAIT/ || NR==1 {print NR,$4,$5,$6}' OFS="\t" input.txt
# 条件 或
awk '$6 ~ /WAIT|FIN/ || NR==1 {print NR,$4,$5,$6}' OFS="\t" input.txt
# 取反
awk '$6 !~ /WAIT/ || NR==1 {print NR,$4,$5,$6}' OFS="\t" input.txt
```

##### 拆分文件

```bash
# 按第6列分隔文件   NR!=1表示不处理第一行
awk 'NR!=1{print > $6}' input.txt
# 只输出指定列到文件
awk 'NR!=1{print $4,$5 > $6}' input.txt

# awk就是个脚本解释器
awk 'NR!=1{if($6 ~ /TIME|ESTABLISHED/) print > "1.txt";
else if($6 ~ /LISTEN/) print > "2.txt";
else print > "3.txt"}' input.txt
```

##### 统计

```bash
# 计算文件大小总和
ls -l *.cpp *.c *.h | awk '{sum+=$5} END {print sum}'

# 统计每个用户的进程占用多少内存
ps aux | awk 'NR!=1{a[$1]+=$6;} END { for(i in a) print i ", " a[i]"KB"; }'
```

##### awk脚本

语法如下:

- BEGIN{ 这里面放的是执行前的语句 }
- {这里面放的是处理每一行时要执行的语句}
- END {这里面放的是处理完所有的行后要执行的语句 }

示例

```bash
$ cat score.txt
Marry   2143 78 84 77
Jack    2321 66 78 45
Tom     2122 48 77 71
Mike    2537 87 97 95
Bob     2415 40 57 62

$ cat cal.awk
#!/bin/awk -f
#运行前
BEGIN {
    math = 0
    english = 0
    computer = 0
    printf "NAME    NO.   MATH  ENGLISH  COMPUTER   TOTAL\n"
    printf "---------------------------------------------\n"
}
#运行中
{
    math+=$3
    english+=$4
    computer+=$5
    printf "%-6s %-6s %4d %8d %8d %8d\n", $1, $2, $3,$4,$5, $3+$4+$5
}
#运行后
END {
    printf "---------------------------------------------\n"
    printf "  TOTAL:%10d %8d %8d \n", math, english, computer
    printf "AVERAGE:%10.2f %8.2f %8.2f\n", math/NR, english/NR, computer/NR
}

$ awk -f cal.awk score.txt
NAME    NO.   MATH  ENGLISH  COMPUTER   TOTAL
---------------------------------------------
Marry  2143     78       84       77      239
Jack   2321     66       78       45      189
Tom    2122     48       77       71      196
Mike   2537     87       97       95      279
Bob    2415     40       57       62      159
---------------------------------------------
  TOTAL:       319      393      350
AVERAGE:     63.80    78.60    70.00
```

##### 环境变量

使用`-v`和`ENVIRON`与环境变量交互, 使用`ENVIRON`的环境变量需要**export**

```bash
$ x=5
$ y=10
$ export y
$ echo $x $y
5 10
$ awk -v val=$x '{print $1, $2, $3, $4+val, $5+ENVIRON["y"]}' OFS="\t" score.txt
Marry   2143    78      89      87
Jack    2321    66      83      55
Tom     2122    48      82      81
Mike    2537    87      102     105
Bob     2415    40      62      72
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
1. **16:41:15** - 系统当前时间;  **up 58 days,  4:44** - 运行时长, 期间没有重启过; **1user** - 当前有1个用户登录; **load average: 0.18, 0.23, 0.22** - 系统在过去***1*分钟**, ***5*分钟**, ***15*分钟**的负载情况
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
top -H -p 1234
```

#### 交互操作
`q` 退出top命令
`<Space>` 立即刷新
`s` 设置刷新时间间隔
**`c`** 显示命令完全模式
`t` 显示或隐藏进程和CPU状态信息
`m` 显示或隐藏内存状态信息
`l` 显示或隐藏uptime信息
`f` 增加或减少进程显示标志
`S` 累计模式，会把已完成或退出的子进程占用的CPU时间累计到父进程的MITE+
**`P`** 按%CPU使用率排行
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

### iptables 防火墙

**语法**

```shell
iptables(选项)(参数)
```

**选项**

```ini
-t, --table table 对指定的表 table 进行操作， table 必须是 raw， nat，filter，mangle 中的一个。如果不指定此选项，默认的是 filter 表。

# 通用匹配：源地址目标地址的匹配
-p：指定要匹配的数据包协议类型；
-s, --source [!] address[/mask] ：把指定的一个／一组地址作为源地址，按此规则进行过滤。当后面没有 mask 时，address 是一个地址，比如：192.168.1.1；当 mask 指定时，可以表示一组范围内的地址，比如：192.168.1.0/255.255.255.0。
-d, --destination [!] address[/mask] ：地址格式同上，但这里是指定地址为目的地址，按此进行过滤。
-i, --in-interface [!] <网络接口name> ：指定数据包的来自来自网络接口，比如最常见的 eth0 。注意：它只对 INPUT，FORWARD，PREROUTING 这三个链起作用。如果没有指定此选项， 说明可以来自任何一个网络接口。同前面类似，"!" 表示取反。
-o, --out-interface [!] <网络接口name> ：指定数据包出去的网络接口。只对 OUTPUT，FORWARD，POSTROUTING 三个链起作用。

# 查看管理命令
-L, --list [chain] 列出链 chain 上面的所有规则，如果没有指定链，列出表上所有链的所有规则。

# 规则管理命令
-A, --append chain rule-specification 在指定链 chain 的末尾插入指定的规则，也就是说，这条规则会被放到最后，最后才会被执行。规则是由后面的匹配来指定。
-I, --insert chain [rulenum] rule-specification 在链 chain 中的指定位置插入一条或多条规则。如果指定的规则号是1，则在链的头部插入。这也是默认的情况，如果没有指定规则号。
-D, --delete chain rule-specification -D, --delete chain rulenum 在指定的链 chain 中删除一个或多个指定规则。
-R num：Replays替换/修改第几条规则

# 链管理命令（这都是立即生效的）
-P, --policy chain target ：为指定的链 chain 设置策略 target。注意，只有内置的链才允许有策略，用户自定义的是不允许的。
-F, --flush [chain] 清空指定链 chain 上面的所有规则。如果没有指定链，清空该表上所有链的所有规则。
-N, --new-chain chain 用指定的名字创建一个新的链。
-X, --delete-chain [chain] ：删除指定的链，这个链必须没有被其它任何规则引用，而且这条上必须没有任何规则。如果没有指定链名，则会删除该表中所有非内置的链。
-E, --rename-chain old-chain new-chain ：用指定的新名字去重命名指定的链。这并不会对链内部造成任何影响。
-Z, --zero [chain] ：把指定链，或者表中的所有链上的所有计数器清零。

-j, --jump target <指定目标> ：即满足某条件时该执行什么样的动作。target 可以是内置的目标，比如 ACCEPT，也可以是用户自定义的链。
-h：显示帮助信息；
```

**基本参数**

|    参数     | 作用                                           |
| :---------: | :--------------------------------------------- |
|     -P      | 设置默认策略:iptables -P INPUT (DROP           |
|     -F      | 清空规则链                                     |
|     -L      | 查看规则链                                     |
|     -A      | 在规则链的末尾加入新规则                       |
|     -I      | num 在规则链的头部加入新规则                   |
|     -D      | num 删除某一条规则                             |
|     -s      | 匹配来源地址IP/MASK，加叹号"!"表示除这个IP外。 |
|     -d      | 匹配目标地址                                   |
|     -i      | 网卡名称 匹配从这块网卡流入的数据              |
|     -o      | 网卡名称 匹配从这块网卡流出的数据              |
|     -p      | 匹配协议,如tcp,udp,icmp                        |
| --dport num | 匹配目标端口号                                 |
| --sport num | 匹配来源端口号                                 |

**命令选项输入顺序**

```shell
iptables -t 表名 <-A/I/D/R> 规则链名 [规则号] <-i/o 网卡名> -p 协议名 <-s 源IP/源子网> --sport 源端口 <-d 目标IP/目标子网> --dport 目标端口 -j 动作
```

**工作机制**

规则链名包括(也被称为五个钩子函数（hook functions）)：

- **INPUT链** ：处理输入数据包。
- **OUTPUT链** ：处理输出数据包。
- **FORWARD链** ：处理转发数据包。
- **PREROUTING链** ：用于目标地址转换（DNAT），内到外。
- **POSTOUTING链** ：用于源地址转换（SNAT），外到内。

**防火墙策略**

防火墙策略一般分为两种，一种叫`通`策略，一种叫`堵`策略，通策略，默认门是关着的，必须要定义谁能进。堵策略则是，大门是洞开的，但是你必须有身份认证，否则不能进。所以我们要定义，让进来的进来，让出去的出去，`所以通，是要全通，而堵，则是要选择`。当我们定义的策略的时候，要分别定义多条功能，其中：定义数据包中允许或者不允许的策略，filter过滤的功能，而定义地址转换的功能的则是nat选项。为了让这些功能交替工作，我们制定出了“表”这个定义，来定义、区分各种不同的工作功能和处理方式。

现在用的比较多个功能有3个：
1. filter 定义允许或者不允许的，只能做在3个链上：INPUT ，FORWARD ，OUTPUT
2. nat 定义地址转换的，也只能做在3个链上：PREROUTING ，OUTPUT ，POSTROUTING
3. mangle功能:修改报文原数据，是5个链都可以做：PREROUTING，INPUT，FORWARD，OUTPUT，POSTROUTING

我们修改报文原数据就是来修改TTL的。能够实现将数据包的元数据拆开，在里面做标记/修改内容的。而防火墙标记，其实就是靠mangle来实现的。

**表名包括**：

- **raw** ：RAW表只使用在PREROUTING链和OUTPUT链上,因为优先级最高，从而可以对收到的数据包在连接跟踪前进行处理。一但用户使用了RAW表,在某个链上,RAW表处理完后,将跳过NAT表和ip_conntrack处理,即不再做地址转换和数据包的链接跟踪处理了.
- **mangle** ：拥有prerouting、FORWARD、postrouting三个规则链，除了进行网址转译工作会改写封包外，在某些特殊应用可能也必须去改写封包(ITL、TOS)或者是设定MARK(将封包作记号，以进行后续的过滤)这时就必须将这些工作定义在mangles规则表中
- **nat** ：拥有prerouting和postrouting两个规则链， 主要功能为进行一对一、一对多、多对多等网址转译工作（SNATDNAT）用于网关路由器。
- **filter** ：预设规则表，拥有 INPUT、FORWARD 和 OUTPUT 三个规则链，这个规则表顾名思义是用来进行封包过滤的理动作

4个表的优先级由高到低：raw-->mangle-->nat-->filter



**动作包括**：

- **ACCEPT** ：接收数据包。

- **DROP** ：丢弃数据包。

- **REDIRECT** ：重定向、映射、透明代理。

- **SNAT** ：源地址转换。

- **DNAT** ：目标地址转换。

- **MASQUERADE** ：IP伪装（NAT），用于ADSL。

- **LOG** ：日志记录。

- **SEMARK** : 添加SEMARK标记以供网域内强制访问控制（MAC）

  ![iptables](https://upload.wikimedia.org/wikipedia/commons/thumb/3/37/Netfilter-packet-flow.svg/2880px-Netfilter-packet-flow.svg.png)

#### 实例

**清空当前的所有规则和计数**

```shell
iptables -F  # 清空所有的防火墙规则
iptables -X  # 删除用户自定义的空链
iptables -Z  # 清空计数
```

**配置允许ssh端口连接**

```shell
iptables -A INPUT -s 192.168.1.0/24 -p tcp --dport 22 -j ACCEPT
# 22为你的ssh端口， -s 192.168.1.0/24表示允许这个网段的机器来连接，其它网段的ip地址是登陆不了你的机器的。 -j ACCEPT表示接受这样的请求
```

**允许本地回环地址可以正常使用**

```shell
iptables -A INPUT -i lo -j ACCEPT
#本地圆环地址就是那个127.0.0.1，是本机上使用的,它进与出都设置为允许
iptables -A OUTPUT -o lo -j ACCEPT
```

**设置默认的规则**

```sh
iptables -P INPUT DROP # 配置默认的不让进
iptables -P FORWARD DROP # 默认的不允许转发
iptables -P OUTPUT ACCEPT # 默认的可以出去
```

**配置白名单**

```sh
iptables -A INPUT -p all -s 192.168.1.0/24 -j ACCEPT  # 允许机房内网机器可以访问
iptables -A INPUT -p all -s 192.168.140.0/24 -j ACCEPT  # 允许机房内网机器可以访问
iptables -A INPUT -p tcp -s 183.121.3.7 --dport 3380 -j ACCEPT # 允许183.121.3.7访问本机的3380端口
```

**开启相应的服务端口**

```sh
iptables -A INPUT -p tcp --dport 80 -j ACCEPT # 开启80端口，因为web对外都是这个端口
iptables -A INPUT -p icmp --icmp-type 8 -j ACCEPT # 允许被ping
iptables -A INPUT -m state --state ESTABLISHED,RELATED -j ACCEPT # 已经建立的连接得让它进来
```

**保存规则到配置文件中**

```sh
# 任何改动之前先备份，请保持这一优秀的习惯
cp /etc/sysconfig/iptables /etc/sysconfig/iptables.bak 

iptables-save > /etc/sysconfig/iptables
cat /etc/sysconfig/iptables
```

**列出已设置的规则**

```sh
iptables -L -t nat                  # 列出 nat 上面的所有规则
#            ^ -t 参数指定，必须是 raw， nat，filter，mangle 中的一个
iptables -L -t nat  --line-numbers  # 规则带编号
iptables -L INPUT

iptables -L -nv  # 查看，这个列表看起来更详细

-n：以数字的方式显示ip，它会将ip直接显示出来，如果不加-n，则会将ip反向解析成主机名。
-v：显示详细信息
-vv
-vvv :越多越详细
-x：在计数器上显示精确值，不做单位换算
--line-numbers : 显示规则的行号
-t nat：显示所有的关卡的信息
```

**删除已添加的规则**

```sh
# 添加一条规则
iptables -A INPUT -s 192.168.1.5 -j DROP
# 将所有iptables以序号标记显示
iptables -L -n --line-numbers
# 删除INPUT里序号为8的规则
iptables -D INPUT 8
```

**开放指定的端口**

```sh
iptables -A INPUT -s 127.0.0.1 -d 127.0.0.1 -j ACCEPT     #允许本地回环接口(即运行本机访问本机)
iptables -A INPUT -m state --state ESTABLISHED,RELATED -j ACCEPT  #允许已建立的或相关连的通行
iptables -A OUTPUT -j ACCEPT         #允许所有本机向外的访问
iptables -A INPUT -p tcp --dport 22 -j ACCEPT    #允许访问22端口
iptables -A INPUT -p tcp --dport 80 -j ACCEPT    #允许访问80端口
iptables -A INPUT -j reject       #禁止其他未允许的规则访问
iptables -A FORWARD -j REJECT     #禁止其他未允许的规则访问
```

**屏蔽IP**

```sh
iptables -A INPUT -p tcp -m tcp -s 192.168.0.8 -j DROP  # 屏蔽恶意主机（比如，192.168.0.8
iptables -I INPUT -s 123.45.6.7 -j DROP       #屏蔽单个IP的命令
iptables -I INPUT -s 123.0.0.0/8 -j DROP      #封整个段即从123.0.0.1到123.255.255.254
```

**指定数据包出去的网络接口**

```sh
# 只对 OUTPUT，FORWARD，POSTROUTING 三个链起作用
iptables -A FORWARD -o eth0
```

**端口映射**

```sh
# 本机的 2222 端口映射到内网 虚拟机的22 端口
iptables -t nat -A PREROUTING -d 210.14.67.127 -p tcp --dport 2222  -j DNAT --to-dest 192.168.188.115:22
```





### ping 测试网络通断与延迟

```bash
ping www.baidu.com
ping -c 10 192.168.1.101    # 指定次数
ping -c 10 -i 0.5 192.168.1.101     # 指定次数和间隔(秒)
ping -i 3 -s 1024 -t 255 192.168.1.101  # -s 发送包大小为1024字节 -t TTL为255
```

### Tcpping

ping命令是基于ICMP协议, 遇到某些主机会禁用ICMP协议导致Ping命令失效, 这时可以使用基于tcp协议的工具, 比如: tcpping, tcping, psping, hping, paping等

tcpping安装步骤如下:

```bash
# 1. tcpping 脚本依赖 tcptraceroute 组件, 所以必须先安装 tcptraceroute
yum install -y tcptraceroute
# 2. 下载tcpping文件
wget http://www.vdberg.org/~richard/tcpping
# 3. 将 tcpping 文件移动到 /usr/bin 下并授权
mv tcpping /usr/bin/
cd /usr/bin
chmod 755 tcpping
# 4. -d:打印时间戳  -c:结果输出在一列  -C:调整输出格式  -r n:输出间隔为n秒(默认1) -x n:重复n次
tcpping www.baidu.com
```

hping是一个命令行下使用的 TCP/IP 数据包组装/分析工具, 它不仅能发送 ICMP 回应请求, 它还可以支持 TCP、UDP、ICMP 和 RAW-IP 协议

```bash
# 安装
yum install hping3
# 发送
hping3 -S -p 80 -c 3 www.baidu.com
```



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

# 显示网卡信息
ip a
ip address show
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
netstat -natp | grep 3306    
lsof -i :8080

# 各种tcp连接状态数量统计
netstat -tna | awk '{print $6}' | sort | uniq -c

netstat -n| awk '/^tcp/ {++S[$NF]} END {for(a in S) print a, S[a]}'
```

>- -a (all)显示所有选项，默认不显示 LISTEN 相关
>- -t (tcp)仅显示 tcp 相关选项
>- -u (udp)仅显示 udp 相关选项
>- -n 拒绝显示别名，能显示数字的全部转化成数字。
>- -l 仅列出有在 Listen (监听) 的服務状态
>- -p 显示建立相关链接的程序名
>- -r 显示路由信息，路由表
>- -e 显示扩展信息，例如 uid 等
>- -s 按各个协议进行统计
>- -c 每隔一个固定时间，执行该 netstat 命令。

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
$ netstat -st | grep SYN

$ netstat -st | grep overflowed
$ netstat -st | grep dropped
```

如果`xx times the listen queue of a socket overflowed` 和 `xx SYNs to LISTEN sockets dropped`出现持续的增长, 说明服务器的全队列过小, 全队列发生溢出, 后续的请求就会被丢弃, 服务端出现请求数量上不去的现象.

查看所有端口的统计信息，`-st`/`-su`：统计TCP/UDP	

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
ss dport > :1024
ss sport > :1024
ss sport < :32000
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

# 查看指定进程的所有 TCP 连接信息
lsof -p 4721 -nP | grep TCP
java    163379 root   14u     IPv6         2306092363       0t0        TCP *:8085 (LISTEN)
java    163379 root   29u     IPv6         2306098266       0t0        TCP 172.31.96.175:8085->172.31.103.83:52600 (ESTABLISHED)
java    163379 root   39u     IPv6         2306089478       0t0        TCP 172.31.96.175:8085->172.31.103.83:52610 (ESTABLISHED)
java    163379 root  270u     IPv6         2306091217       0t0        TCP *:21162 (LISTEN)
java    163379 root  272u     IPv6         2306089479       0t0        TCP 172.31.96.175:8085->172.31.103.83:52612 (ESTABLISHED)
java    163379 root  273u     IPv6         2306101388       0t0        TCP 172.31.96.175:8085->172.31.103.83:52624 (ESTABLISHED)
java    163379 root  274u     IPv6         2306077215       0t0        TCP 172.31.96.175:8085->172.31.103.83:52626 (ESTABLISHED)

# 列出指定主机上的指定端口相关的所有文件信息, 3秒刷新一次
lsof -i@172.31.46.2:6379,3306 -r 3
```



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



### tcpdump 抓包

抓取经过eth0接口请求8080端口的tcp请求

```bash
tcpdump tcp -i eth0 -s 0 -nn -vvv and dst port 8080 -w 8080dump.cap
```

- 类型: `host` `net` `port`  
- 方向: `src` `dst` `src or dst` `src and dst`
- 协议：`ip` `tcp` `udp` `arp` `icmp`....

```bash
# 监听特定网卡
tcpdump -i eth0

# 监听特定主机  监听与172.31.46.2主机间的通信，出入的包都会被监听
tcpdump host 172.31.46.2

tcpdump src host 172.31.46.2
tcpdump dst host 172.31.46.2

# 监听特定端口
tcpdump port 8080

# 监听来自172.31.46.2的请求本机8080端口的tcp通信
tcpdump tcp port 8080 and src host 172.31.46.2
```

```bash
tcpdump tcp -i eth0 -t -s 0 -c 100 -n and dst port ! 22 and src net 192.168.1.0/24 -w tcpdump.cap
```

- `tcp` : tcp udp ip icmp arp等这些选项都要放到第一个参数的位置，用来过滤数据报的类型
- `-i eth0`：只抓取接口eth0的包
- `-t`：不显示时间戳
- **`-s 0`**：默认只抓取68字节，加上`-s 0`后可以抓取完整的数据包
- `-c 100`：只抓取100个数据包
- `-n`：将地址以数字形式显示，否则显示为主机名
- **`-nn`**：除了有`-n`的作用，还把端口显示为数值，否则显示为端口服务名
- `dst port ! 22`：不抓取目标端口是22的数据包
- `src net 192.168.1.0/24`：数据包的源网络地址为192.168.1.0/24
- `-w tcpdump.cap`：抓取结果写入文件
- **`-X`**：以16进制和ASCII码显示包数据，**现场分析数据包内容必备**
- `-v` `-vv` `-vvv` ：输出结果详细程度递增



### wget 文件下载

```bash
wget -O file www.xx.com		# 保存到指定文件
wget -i urls.txt -P path/dic   # 从文件读取链接, 下载到指定目录

wget www.xxx.com/test.html	# 获取网页内容并保存到test.html文件
wget -a file www.yy.com     # 追加到文件
wget -o log www.xx.com      # 下载记录保存到log文件
wget -t 40 www.xx.com       # 最多重试40次(默认20)
wget -S www.xx.com/sdf      # 测试链接是否正确
wget --limit-rate=300k www.xx.com   # 限速下载
wget -b www.xx.com			# 后台下载
wget -c www.xx.com			# 断点续传
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
curl -G --data-urlencode 's=kk a' www.xx.com

# POST 
# 加 -d 后会自动加上请求头 Content-Type : application/x-www-form-urlencoded, 且请求转为 POST
curl -d '@data.txt' www.xx.com      # 读取文件内容作为请求体发送 
curl -d 'name=bob' www.xx.com/form  # post提交表单
curl -X POST --data "key=value" www.xx.com  # post发送参数
curl -X POST --date-urlencode "a=aa" www.xx.com        # 自动将发送的数据进行URL 编码
curl -d '{"a":"aa"}' -H 'Content-Type: application/json' www.xx.com   # 发送json请求
# 上传文件和其他参数 --form会导致请求类型为 multipart/form-data
curl -X POST '172.31.46.2:7777' \
--form 'file=@"/tmp/opencv-test/000136.jpg"' \
--form 'flags="2"' \
--form 'borderMode="1"'

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

### 文件传输RZ、SZ

xshell下可以使用，只适合小的文件传输

```bash
# 安装
$ yum install -y lrzsz.x86_64

# 下载
$ sz xxx
# 弹窗确认保存位置

# 上传
$ rz
# 弹窗选择文件
```



### mount 磁盘挂载

```bash
# 列出文件系统的整体磁盘空间使用情况
$ df -h
文件系统        容量  已用  可用 已用% 挂载点
/dev/sda2       223G  6.8G  216G    4% /
devtmpfs        126G     0  126G    0% /dev
tmpfs           126G     0  126G    0% /dev/shm
tmpfs           126G   19M  126G    1% /run
tmpfs           126G     0  126G    0% /sys/fs/cgroup
/dev/sda1       494M  158M  337M   32% /boot
/dev/sdb1       2.0T   42G  2.0T    3% /iflytek
tmpfs            26G     0   26G    0% /run/user/0

# 列出所有可用块设备的信息，显示他们之间的依赖关系
$ lsblk
NAME   MAJ:MIN RM   SIZE RO TYPE MOUNTPOINT
sda      8:0    0 223.1G  0 disk
├─sda1   8:1    0   500M  0 part /boot
└─sda2   8:2    0 222.6G  0 part /
sdb      8:16   0   7.3T  0 disk
└─sdb1   8:17   0     2T  0 part /iflytek
```

这里以CentOS下创建xfs为例

centos7.0开始默认文件系统是xfs，centos6是ext4，centos5是ext3

```bash
# 1 安装XFS系统工具集
# Fedora, CentOS, RHEL系统:
$ sudo yum install xfsprogs
# 很多Linux系统将XFS作为默认文件系统，所以无需安装, 比如Centos 7.3就无需安装
# Debian, Ubuntu , Linux Mint系统：
$ sudo apt-get install xfsprogs

# 2 创建xfs格式分区
# 查看下是否有分区
$ fdisk -l
Disk /dev/sdb: 323.2 GB, 323196289024 bytes, 631242752 sectors
Units = sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes

# 开始分区
$ fdisk /dev/sdb
Welcome to fdisk (util-linux 2.23.2).
 
Changes will remain in memory only, until you decide to write them.
Be careful before using the write command.
 
Device does not contain a recognized partition table
Building a new DOS disklabel with disk identifier 0xafc7c358.
# 输入n 添加分区 
Command (m for help): n
Partition type:
   p   primary (0 primary, 0 extended, 4 free)
   e   extended
# 输入p 指定为主分区   
Select (default p): p
# 输入分区编号
Partition number (1-4, default 1): 1
# 一路默认
First sector (2048-631242751, default 2048): 
Using default value 2048
Last sector, +sectors or +size{K,M,G} (2048-631242751, default 631242751): 
Using default value 631242751
Partition 1 of type Linux and of size 301 GiB is set
# 输入w 保存配置并退出
Command (m for help): w
The partition table has been altered!
 
Calling ioctl() to re-read partition table.
Syncing disks.

# 3 格式化分区为xfs 如果已有其他文件系统创建在此分区，必须加上"-f"参数来覆盖它。
$ mkfs.xfs -f  /dev/sdb
meta-data=/dev/sdb               isize=512    agcount=4, agsize=19726336 blks
         =                       sectsz=512   attr=2, projid32bit=1
         =                       crc=1        finobt=0, sparse=0
data     =                       bsize=4096   blocks=78905344, imaxpct=25
         =                       sunit=0      swidth=0 blks
naming   =version 2              bsize=4096   ascii-ci=0 ftype=1
log      =internal log           bsize=4096   blocks=38528, version=2
         =                       sectsz=512   sunit=0 blks, lazy-count=1
realtime =none                   extsz=4096   blocks=0, rtextents=0

# 4 挂载
$ mkdir /mysql
$ mount  -t  xfs  /dev/sdb  /mysql
# 验证XFS挂载是否成功
$ df -Th /dev/sdb
Filesystem     Type 1K-blocks  Used Available Use% Mounted on
/dev/sdb       xfs  315467264 32944 315434320   1% /mysql
$ df -h
Filesystem      Size  Used Avail Use% Mounted on
/dev/sda2        30G   22G  8.2G  73% /
devtmpfs        1.7G     0  1.7G   0% /dev
tmpfs           1.7G     0  1.7G   0% /dev/shm
tmpfs           1.7G   25M  1.7G   2% /run
tmpfs           1.7G     0  1.7G   0% /sys/fs/cgroup
/dev/sda1       497M   62M  436M  13% /boot
/dev/sdc1       133G  4.2G  122G   4% /mnt/resource
tmpfs           344M     0  344M   0% /run/user/1000
/dev/sdb        301G   33M  301G   1% /mysql

# 5 开机自动挂载
$ vim /etc/fstab
# 追加如下一行
/dev/sdb1		/iflytek 			xfs			defaults		0 0

# 可以使用UUID替换对应的设备
$ blkid /dev/sdb1
/dev/sdb1: UUID="23c36f8a-04cb-4cfc-8fdd-663379c4563c" TYPE="xfs"

$ vim /etc/fstab
UUID=23c36f8a-04cb-4cfc-8fdd-663379c4563c		/iflytek 	xfs		defaults	0 0
```

查看磁盘分区的文件系统类型

```bash
$ df -Th
文件系统       类型      容量  已用  可用 已用% 挂载点
/dev/sda2      xfs       223G  6.8G  216G    4% /
devtmpfs       devtmpfs  126G     0  126G    0% /dev
tmpfs          tmpfs     126G     0  126G    0% /dev/shm
tmpfs          tmpfs     126G   19M  126G    1% /run
tmpfs          tmpfs     126G     0  126G    0% /sys/fs/cgroup
/dev/sda1      xfs       494M  158M  337M   32% /boot
/dev/sdb1      xfs       2.0T   42G  2.0T    3% /iflytek
tmpfs          tmpfs      26G     0   26G    0% /run/user/0

# 或者
$ parted -l
Model: AVAGO INSPUR (scsi)
Disk /dev/sda: 240GB
Sector size (logical/physical): 512B/4096B
Partition Table: msdos
Disk Flags:

Number  Start   End    Size   Type     File system  标志
 1      1049kB  525MB  524MB  primary  xfs          启动
 2      525MB   240GB  239GB  primary  xfs


Model: AVAGO INSPUR (scsi)
Disk /dev/sdb: 8001GB
Sector size (logical/physical): 512B/4096B
Partition Table: msdos
Disk Flags:

Number  Start   End     Size    Type     File system  标志
 1      1049kB  2199GB  2199GB  primary  xfs

# 查看已格式化分区的UUID和文件系统
$ blkid
/dev/sda2: UUID="58e6f06c-55bf-4565-9ef3-50b1155c1e2a" TYPE="xfs"
/dev/sdb1: UUID="23c36f8a-04cb-4cfc-8fdd-663379c4563c" TYPE="xfs"
/dev/sda1: UUID="799fab66-2443-4490-a387-b3483162e23d" TYPE="xfs"

# 可以查看未挂载的文件系统类型，有些系统可能没有这个命令
$ lsblk -f
NAME   FSTYPE LABEL UUID                                 MOUNTPOINT
sda
├─sda1 xfs          799fab66-2443-4490-a387-b3483162e23d /boot
└─sda2 xfs          58e6f06c-55bf-4565-9ef3-50b1155c1e2a /
sdb
└─sdb1 xfs          23c36f8a-04cb-4cfc-8fdd-663379c4563c /iflytek
```

#### LVM(Logical volume Manager)逻辑卷管理相关概念

传统的磁盘管理机制，Linux操作系统和windows的差不多，绝大多数都是使用MBR(Master Boot Recorder)都是通过先对一个硬盘进行分区，然后再将该分区进行文件系统的格式化，在Linux系统中如果要使用该分区就将其挂载上去即可，windows的话其实底层也就是自动将所有的分区挂载好，然后我们就可以对该分区进行使用了。

在传统的磁盘管理机制中，我们的上层应用是直接访问文件系统，从而对底层的物理硬盘进行读取，而在LVM中，其通过对底层的硬盘进行封装，当我们对底层的物理硬盘进行操作时，其不再是针对于分区进行操作，而是通过一个叫做逻辑卷的东西来对其进行底层的磁盘管理操作。

LVM最大的特点就是可以对磁盘进行动态管理。因为逻辑卷的大小是可以动态调整的，而且不会丢失现有的数据。我们如果新增加了硬盘，其也不会改变现有上层的逻辑卷。

- 物理拓展(Physical Extend，PE)　　
- 物理卷（Physical Volume,PV）：也就是物理磁盘分区，如果想要使用LVM来管理这个分区，可以使用fdisk将其ID改为LVM可以识别的值，即8e。
- 卷组（Volume Group,VG）：PV的集合
- 逻辑卷（Logic Volume,LV）：VG中画出来的一块逻辑磁盘

**为什么要使用逻辑卷**：

- 业务上使用大容量的磁盘。举个例子，我们需要在/data下挂载30TB的存储，对于单个磁盘，是无法满足要求的，因为市面上没有那么大的单块磁盘。但是如果我们使用逻辑卷，将多个小容量的磁盘聚合为一个大的逻辑磁盘，就能满足需求。
- 扩展和收缩磁盘。在业务初期规划磁盘时，我们并不能完全知道需要分配多少磁盘空间是合理的，如果使用物理卷，后期无法扩展和收缩，如果使用逻辑卷，可以根据后期的需求量，手动扩展或收缩。

总结：

(1)物理磁盘被格式化为PV，空间被划分为一个个的PE

(2)不同的PV加入到同一个VG中，不同PV的PE全部进入到了VG的PE池内

(3)LV基于PE创建，大小为PE的整数倍，组成LV的PE可能来自不同的物理磁盘（当我们创建好我们的VG以后，这个时候我们创建LV其实就是从VG中拿出我们指定数量的PE）

(4)LV现在就直接可以格式化后挂载使用了

(5)LV的扩充缩减实际上就是增加或减少组成该LV的PE数量，其过程不会丢失原始数据

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

### 修改时间、时区、语言

```bash
# 查看当前时区
$ date -R
Sat, 15 May 2021 15:33:16 +0800

# 修改时区方法1 依次输入序号 Asia > China > Beijing Time
$ tzselect

# 或者
$ ln -sf /usr/share/zoneinfo/Asia/Shanghai /etc/localtime
```

```bash
# 查看时间日期
$ date
2021年 05月 15日 星期六 15:43:17 CST

# 设置日期  2021-05-15
$ date -s 15/05/2021
# 设置时间 15:45:30
$ date -s 15:45:30

# 或者使用datetimectl
$ timedatectl -h

# 将当前时间日期写入BIOS，避免重启后失效
$ hwclock -w
```

```bash
# 查看当前语言设置
$ echo $LANG
zh_CN.UTF-8

# 查看系统安装的语言包
$ locale -a | grep zh_CN

# 如果没有中文语言，可以执行如下指令，安装中文语言包
$ yum groupinstall chinese-support

# 临时修改
$ export LANG=zh_CN.UTF-8
# 永久修改
$ vim /etc/locale.conf  
# 改为 LANG="zh_CN.UTF-8"
# 或者
$ localectl  set-locale LANG=zh_CN.UTF8
```



## 应用

### yum

```bash
yum check-update	# 列出所有可更新的软件清单
yum update			# 更新所有软件
yum update xxx		# 更新指定软件 
yum install -y xxx	# 安装
yum list			# 列出可安装的软件列表
yum remove -y xxx	# 卸载
yum search xxx		# 查找
yum info xxx		#显示指定的rpm软件包的描述信息和概要信息
yum resovledep xx	#显示rpm软件包的依赖关系
yum localinstall xx	#安装本地的rpm软件包
yum deplist xxx		#显示rpm软件包的所有依赖关系

yum search pam*		# 通配符查找
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



### service



### systemctl



### init.d



### journalctl



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



### seq 生成连续的数字序列

```bash
# 输出1到5
$ seq 5
# 或者
$ seq 1 5
1
2
3
4
5

# 输出1到5，步长为2
$ seq 1 2 5
1
3
5

# 序列在单行，用指定分隔符进行分隔
$ seq -s " " 1 2 5
1 3 5

# 使用指定字符补齐长度
$ seq -s " " -f "%04g" 1 2 5
0001 0003 0005
```



### crontab 定时任务

`-u user` :  对指定用户的crontab服务进行处理

```bash
*   *   *   *   *
分  时  天   月  周(1-6 0)

# 列出crontab文件
crontab -l
# 编辑
crontab -e
# 删除crontab文件
crontab -r

# 每分钟执行一次
* * * * * myCommand
# 每小时执行一次
* */1 * * *  myCommand

# 每隔两天在上午8点到11的第3和第15分钟执行
3,15 8-11 */2  *  * myCommand
# 每周1上午8点到11的第3和第15分钟执行
3,15 8-11 * * 1 myCommand
# 每周六,周日的1:10重启smb
10 1 * * 6,0 /etc/init.d/smb restart
# 12月每天早上6点到12点, 每隔3个小时的0分钟执行
0 6-12/3 * 12 * myCommand
```

crontab的日志文件是`/var/log/cron`   查看cron服务状态`systemctl status crond`

**注意**

- 命令中`%`是特殊字符, 需要加反斜线`\`转义

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

nohup 是 no hang up 的缩写，就是不挂断的意思。该命令可以在你退出帐户/关闭终端时忽略SIGHUP信号，从而继续运行相应的进程。在缺省情况下该作业的所有输出都被重定向到当前目录的nohup.out的文件中, 如果当前目录的 nohup.out 文件不可写，输出重定向到 $HOME/nohup.out 文件中。

```bash
nohup java -jar xxx.jar > /dev/null 2>&1 &
```

`&` 在后台运行，但是关闭shell后，对应的服务也会关闭，因为对SIGHUP信号不免疫。此时配合nohup可以让应用继续运行。  `2>&1` 是将标准出错重定向到标准输, `/dev/null 2>&1` 将标准输出和错误输出全部重定向到/dev/null中,也就是将产生的所有信息丢弃



## 系统监控

下面大部分命令属于`sysstat`软件包的一部分, 执行`yum install -y sysstat`

### mpstat  - cpu监控

每隔1s打印所有cpu使用情况, 间隔1s，打印10次

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
  - **st** 管理程序(hypervisor 虚拟化)为另一个虚拟进程提供服务而等待虚拟 CPU 的百分比


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
$ iostat -x 1 3
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
`r_await` 平均每次设备读I/O操作的等待时间 (毫秒)。
`w_await` 平均每次设备写I/O操作的等待时间 (毫秒)。
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

- `u` cpu使用率
- `q` cpu负载
- `B` 内存使用率
- `R` 内存页
- `S` swap使用率
- `b` io
- `n`  网络
- `d ` 块设备
- `F` 挂载设备
- `m` 电源管理

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

### fdisk 磁盘列表

```bash
$ fdisk -l
磁盘 /dev/sda：239.5 GB, 239511535616 字节，467795968 个扇区
Units = 扇区 of 1 * 512 = 512 bytes
扇区大小(逻辑/物理)：512 字节 / 4096 字节
I/O 大小(最小/最佳)：65536 字节 / 65536 字节
磁盘标签类型：dos
磁盘标识符：0x0004b739

   设备 Boot      Start         End      Blocks   Id  System
/dev/sda1   *        2048     1026047      512000   83  Linux
/dev/sda2         1026048   467795967   233384960   83  Linux

磁盘 /dev/sdb：8001.0 GB, 8001020755968 字节，15626993664 个扇区
Units = 扇区 of 1 * 512 = 512 bytes
扇区大小(逻辑/物理)：512 字节 / 4096 字节
I/O 大小(最小/最佳)：65536 字节 / 65536 字节
磁盘标签类型：dos
磁盘标识符：0x7356a9d5

   设备 Boot      Start         End      Blocks   Id  System
/dev/sdb1            2048  4294967294  2147482623+  83  Linux
```



### df  查询磁盘可用空间

```bash
$ df -lh
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

### iftop 带宽使用监控



### mtr 网络测试工具

mtr（My traceroute）几乎是所有Linux发行版本预装的网络测试工具，集成了`tracert`与`ping`这两个命令的图形界面，功能十分强大。`ping`送出封包到指定的服务器。如果服务器有回应就会传送回封包，并附带返回封包来回的时间。`tracert`返回从用户的电脑到指定的服务器中间经过的所有节点（路由）以及每个节点的回应速度。

mtr默认发送ICMP数据包进行链路探测，通过“-u”参数指定UDP数据包用于探测。相对于traceroute只做一次链路跟踪测试，mtr会对链路上的相关节点做持续探测并给出相应的统计信息。mtr能避免节点波动对测试结果的影响，所以其测试结果更正确，建议优先使用。

```bash
jcy-dev (0.0.0.0)                                         Fri Feb 26 09:48:16 2021
Keys:  Help   Display mode   Restart statistics   Order of fields   quit
                                          Packets               Pings
 Host                                   Loss%   Snt   Last   Avg  Best  Wrst StDev
 1. 172.31.46.1                          0.0%    18    0.9   0.9   0.7   2.2   0.0
 2. 10.253.13.65                         0.0%    18    0.8   0.7   0.6   0.9   0.0
 3. 10.253.13.70                         0.0%    18    1.0   0.9   0.8   1.1   0.0
 4. 100.65.0.9                           0.0%    18    0.4   0.4   0.3   0.5   0.0
 5. 36.7.109.2                           0.0%    18    1.3   1.3   1.0   2.4   0.2
 6. 100.64.23.1                          0.0%    18    0.8   1.6   0.7  11.4   2.4
 7. 10.12.0.161                          0.0%    18    2.0   1.7   1.3   2.1   0.0
 8. DZL-CORE-S-GE2-1.MAN.HF.AH.CN        0.0%    18    1.2   1.2   1.1   1.7   0.0
 9. 202.102.207.101                      0.0%    17    8.6   4.4   1.3  12.8   3.9
10. 202.97.100.1                         0.0%    17    8.6   8.4   8.3   9.0   0.0
11. 101.95.218.246                      81.2%    17   10.9   9.7   9.0  10.9   1.0
12. 101.95.209.70                        0.0%    17    9.8   9.8   9.3  13.3   0.8
13. 180.163.38.30                        0.0%    17    9.2  10.0   9.0  17.4   2.1
14. 42.120.241.42                        0.0%    17   11.5  11.5   9.5  35.9   6.3
15. 116.251.116.97                       0.0%    17   10.9  16.5  10.8  41.6   8.7
16. ???
17. ???
18. public1.alidns.com                   0.0%    17   10.0  10.1  10.0  10.3   0.0
```

#### 常见可选参数说明

- -r或--report：以报告模式显示输出。
- -p或--split：将每次追踪的结果分别列出来，而非--report统计整个结果。
- -s或--psize：指定ping数据包的大小。
- -n或--no-dns：不对IP地址做域名反解析。
- -a或--address：设置发送数据包的IP地址。用于主机有多个IP的情况。
- -4：只使用IPv4协议。
- -6：只使用IPv6协议。

在mtr运行过程中，您也可以输入相应字母来快速切换模式，各字母的含义如下。
- ?或h：显示帮助菜单。
- d：切换显示模式。
- n：切换启用或禁用DNS域名解析。
- u：切换使用ICMP或UDP数据包进行探测。

#### 返回结果说明
默认配置下，返回结果中各数据列的说明如下。

- 第一列（Host）：节点IP地址和域名。按 **n** 键可切换显示。
- 第二列（Loss%）：节点丢包率。
- 第三列（Snt）：每秒发送数据包数。默认值是10，可以通过“-c”参数指定。
- 第四列（Last）：最近一次的探测延迟。
- 第五、六、七列（Avg、Best、Worst）：分别是探测延迟的平均值、最小值和最大值。
- 第八列（StDev）：标准偏差。越大说明相应节点越不稳定。

#### 分析链路测试结果

- 结合Avg（平均值）和StDev（标准偏差），判断各节点是否存在异常。

  - 若StDev很高，则同步观察相应节点的Best和Worst，来判断相应节点是否存在异常。

  - 若StDev不高，则通过Avg来判断相应节点是否存在异常。

    >**注意**：上述StDev高或者不高，并没有具体的时间范围标准。而需要根据同一节点其它列的延迟值大小来进行相对评估。比如，如果Avg为30ms，那么，当StDev为25ms，则认为是很高的偏差。而如果Avg为325ms，则同样的StDev为25ms，反而认为是不高的偏差。

- 查看节点丢包率，若“Loss%”不为零，则说明这一跳路由的网络可能存在问题。导致节点丢包的原因通常有两种。
  - 人为限制了节点的ICMP发送速率，导致丢包。
  - 节点确实存在异常，导致丢包。

- 确定当前异常节点的丢包原因。

  - 若随后节点均没有丢包，说明当前节点丢包是由于运营商策略限制所致，可以忽略。如前文链路测试结果示例图中的第2跳路由的网络所示。

  - 若随后节点也出现丢包，说明当前节点存在网络异常，导致丢包。如前文链路测试结果示例图中的第5跳路由的网络所示。

    > **说明**：前述两种情况可能同时发生，即相应节点既存在策略限速，又存在网络异常。对于这种情况，若当前节点及其后续节点连续出现丢包，而且各节点的丢包率不同，则通常以最后几跳路由的网络的丢包率为准。

- 通过查看是否有明显的延迟，来确认节点是否存在异常。通过如下两个方面进行分析。

  - 若某一跳路由的网络之后延迟明显陡增，则通常判断该节点存在网络异常。如前文链路测试结果示例图所示，从第5跳路由的网络之后的后续节点延迟明显陡增，则推断是第5跳路由的网络节点出现了网络异常。

    >  注：高延迟并不一定完全意味着相应节点存在异常，延迟大也有可能是在数据回包链路中引发的，建议结合反向链路测试一并分析。

  -  ICMP策略限速也可能会导致相应节点的延迟陡增，但后续节点通常会恢复正常。



[Linux工具快速教程](https://linuxtools-rst.readthedocs.io/zh_CN/latest/index.html)

[CentOS7.5 系统最小化安装与初始化配置](https://www.cnblogs.com/tssc/p/11041464.html)