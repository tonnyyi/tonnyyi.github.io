---
title: shell
tags:
  - null
categories:
  - null
date: 2019-01-10 08:36:03
---
## 变量
变量声明方式: `变量名=变量值`, `=`左右不要有空格. 变量名命名规则:
- 只能使用数字, 大小写字母, 下划线, 首字符不能为数字
- 不能有空格, 可以使用`_`
- 不能使用base里的关键字(使用`help`命令查看关键字)

```shell
boo
_var="hello"
# 除了显示设置变量, 还可以用语句给变量赋值
for file in `ls /etc`
或
for file in $(ls /etc)

# 使用变量只需要在变量名前加美元符号即可
your_name="Tom"
echo $your_name
echo ${your_name}
```
**推荐给所有变量加上花括号**
已定义的变量, 可以被重新赋值, 如:
```shell
your_name="John"
echo $your_name
your_name="alibaba"
echo $your_name
```
第二次赋值的时候不能写\$your_name="alibaba", **使用变量的时候才加美元符（$）**

### 只读变量
使用 readonly 命令可以将变量定义为只读变量, 只读变量的值不能被改变.
```shell
#!/bin/bash
myUrl="http://www.google.com"
readonly myUrl
myUrl="http://www.runoob.com"
```
运行时会报错
```bash
/bin/sh: NAME: This variable is read only.
```

### 删除变量
使用 `unset` 命令可以删除变量
```shell
#!/bin/sh
myUrl="http://www.runoob.com"
unset myUrl
echo $myUrl
```
以上实例执行将没有任何输出

### 变量类型
* **局部变量** 局部变量在脚本或命令中定义，仅在当前shell实例中有效，其他shell启动的程序不能访问局部变量。
* **环境变量** 所有的程序，包括shell启动的程序，都能访问环境变量，有些程序需要环境变量来保证其正常运行。必要的时候shell脚本也可以定义环境变量。
* **shell变量** shell变量是由shell程序设置的特殊变量。shell变量中有一部分是环境变量，有一部分是局部变量，这些变量保证了shell的正常运行

### 字符串

[linux shell 字符串操作详解 （长度，读取，替换，截取，连接，对比，删除，位置 ）](https://blog.csdn.net/dongwuming/article/details/50605911)
[Reading output of a command into an array in Bash](https://stackoverflow.com/questions/11426529/reading-output-of-a-command-into-an-array-in-bash)
```shell
your_name='runoob'
str="Hello, I know you are \"$your_name\"! \n"
echo -e $str
```
输出结果
```
Hello, I know you are "runoob"!
```

单引号字符串的限制：
* 单引号里的任何字符都会原样输出，单引号字符串中的变量是无效的；
* 单引号字串中不能出现单独一个的单引号（对单引号使用转义符后也不行），但可成对出现，作为字符串拼接使用。

双引号的优点：
* 双引号里可以有变量
* 双引号里可以出现转义字符

#### 操作

| 表达式 | 含义 |
| :-: | :-: |
| `{{"${#string}"}}` | `$string`的长度  |
| `${string:position}` |	在`$string`中, 从位置`$position`开始提取子串 |
| `${string:position:length}` |	在`$string`中, 从位置`$position`开始提取长度为`$length`的子串	  |
| `${string#substring}` |	从变量`$string`的开头, 删除最短匹配`$substring`的子串 |
| `${string##substring}` |	从变量`$string`的开头, 删除最长匹配`$substring`的子串 |
| `${string%substring}` |	从变量`$string`的结尾, 删除最短匹配`$substring`的子串 |
| `${string%%substring}` |	从变量`$string`的结尾, 删除最长匹配`$substring`的子串 |
| `${string/substring/replacement}` |	使用`$replacement`, 来代替第一个匹配的`$substring` |
| `${string//substring/replacement}` |	使用`$replacement`, 代替所有匹配的`$substring` |
| `${string/#substring/replacement}` |	如果`$string`的前缀匹配`$substring`, 那么就用`$replacement`来代替匹配到的`$substring` |
| `${string/%substring/replacement}` |	如果`$string`的后缀匹配`$substring`, 那么就用`$replacement`来代替匹配到的`$substring` |

```shell
# 字符串长度 ${#变量名}得到字符串长度
hi="abcd"
echo ${#hi} #输出 4

# 截取字串  ${变量名:起始:长度}得到子字符串
test='I love china'
echo ${test:5}     # e china
echo ${test:5:10}  # e china

# 字符串删除
# ${变量名#substring正则表达式}从字符串开头开始配备substring,删除匹配上的表达式
# ${变量名%substring正则表达式}从字符串结尾开始配备substring,删除匹配上的表达式
# 注意：${test##*/},${test%/*} 分别是得到文件名，或者目录地址最简单方法
test='c:/windows/boot.ini'
echo ${test#/}      # c:/windows/boot.ini
echo ${test#*/}     # windows/boot.ini
echo ${test##*/}    # boot.ini

echo ${test%/*}     # c:/windows
echo ${test%%/*}

# 字符串替换
test='c:/windows/boot.ini'
echo ${test/\//\\}      # c:\windows/boot.ini
echo ${test//\//\\}     # c:\windows\boot.ini
```

##### 拼接
```shell
your_name="runoob"
# 使用双引号拼接
greeting="hello, "$your_name" !"
greeting_1="hello, ${your_name} !"
echo $greeting  $greeting_1 # hello, runoob ! hello, runoob !
# 使用单引号拼接
greeting_2='hello, '$your_name' !'
greeting_3='hello, ${your_name} !'
echo $greeting_2  $greeting_3   # hello, runoob ! hello, ${your_name} !
```

##### 子字符串
```shell
# 提取子字符串
string="runoob is a great site"
echo ${string:1:4} # 输出 unoo

# 查找子字符串
string="runoob is a great site"
echo `expr index "$string" io`  # 输出 4
```

### 数组
```shell
array_name=(value0 value1 value2 value3)
#或者
array_name=(
value0
value1
value2
value3
)

# 还可以单独定义数组的各个分量：
array_name[0]=value0
array_name[1]=value1
array_name[n]=valuen

# 读取长度元素 ${数组名[下标]}
val=${array_name[1]}

# 使用 @ 符号可以获取数组中的所有元素
echo ${array_name[@]}

# 取得数组元素的个数
length=${#array_name[@]}
# 或者
length=${#array_name[*]}
# 取得数组单个元素的长度
lengthn=${#array_name[n]}
```

### 注释
```shell
# 以 # 开头的行就是注释, 会被解释器忽略
```


## 参数
脚本通过`$n`获取参数
```shell
#!/bin/bash

echo "Shell 传递参数实例！";
echo "执行的文件名：$0";
echo "第一个参数为：$1";
echo "第二个参数为：$2";
echo "第三个参数为：$3";
```
执行脚本
```bash
$ ./test.sh 1 2 3
Shell 传递参数实例！
执行的文件名：./test.sh
第一个参数为：1
第二个参数为：2
第三个参数为：3
```

#### 特殊参数列表
| 变量 | 含义 |
| :-: | :-- |
| `$0` | 是脚本本身的名字 |
| `$n` | 添加到sehll的各参数值.`$1`是第一个参数,`$2`是第二个参数,以此类推 |
| `$#` | 传给脚本的参数个数 |
| `$@` | 传给脚本的所有参数的列表, 即被扩展为`$1` `$2` `$3`等 |
| `$*` | 以一个单字符串显示所有向脚本传递的参数, 即被扩展成`\$1c\$2c\$3`, 其中c是IFS的第一个字符 |
| `$$` | 脚本运行的当前进程ID号 |
| `$?` | 显示最后命令的退出状态, 0表示没有错误, 其他表示有错误 |
| `$!` | 脚本最后运行的后台Process的进程id |
| `$-` | 显示Shell使用的当前选项，与set命令功能相同。 |

`$*` 与 `$@` 区别：
* 相同点：都是引用所有参数。
* 不同点：只有在双引号中体现出来。假设在脚本运行时写了三个参数 1、2、3，，则 " * " 等价于 "1 2 3"（传递了一个参数），而 "@" 等价于 "1" "2" "3"（传递了三个参数）。

```shell
echo "-- \$* 演示 ---"
for i in "$*"; do
    echo $i
done

echo "-- \$@ 演示 ---"
for i in "$@"; do
    echo $i
done
```
执行脚本，输出结果如下所示
```bash
$ ./test.sh 1 2 3
-- $* 演示 ---
1 2 3
-- $@ 演示 ---
1
2
3
```

### getopt getopts
[Bash Shell中命令行选项/参数处理](https://www.cnblogs.com/FrankTan/archive/2010/03/01/1634516.html)

## 运算符

### 算数运算符

| 运算符 | 说明 |
| :-: | --- |
| + | 加法 |
| - | 减法 |
| * | 乘法 |
| / | 除法, 取整 |
| % | 取余 |
| = | 赋值  |
| == | 相等, 相同返回true |
| != | 不等, 不同返回false |

**注意 :**
* 条件表达式要放在方括号之间, 并且要有空格, 例如: `[$a==$b]` 是错误的, 必须写成 `[ $a == $b ]`
* 乘号(*)前边必须加反斜杠(\)才能实现乘法运算, 即写成`expr $a \* $b`
*  MAC 中 shell 的 expr 语法是: `$((表达式))`, 此处表达式中的`*`不需要转义符号`\`

原生bash不支持简单的数学运算, 但可以通过一下几种方式实现:
1. `let`

    ```shell
    var=1;
    let "var+=1";
    echo $var
    ```
    * let几乎支持所有的运算符
    * 方幂运算应使用“**”
    * 变量在表达式中直接访问，不必加$
    * 一般情况下算数表达式可以不加双引号，但是若表达式中有bash中的关键字则需加上
    * let后的表达式只能进行整数运算

2. `(())`
    `(())`的使用方法与let关键字完全相同
    ```shell
    var=1;
    ((var+=1));
    echo $var
    ```

3. `$[]`

    ```shell
    var=1;
    var=$[$var+1];
    echo $var
    ```
    * `$[]`将中括号内的表达式作为数学运算先计算结果再输出
    * 对`$[]`中的变量进行访问时前面需要加`$`
    * `$[]`支持的运算符与`let`相同，但也只支持整数运算

4. `expr`

    ```shell
    var=1;
    var=`expr $var + 1`;
    echo $var
    ```
    * expr后的表达式符号间需用空格隔开
    * expr支持的操作符有： |、&、< >=、<、+、-、*、/、%
    * expr支持的操作符中在使用时需用\进行转义的有：|、&、< >=、<、*
    * expr同样只支持整数运算

5. `bc`
    bc是linux下的一个简单计算器，支持浮点数计算，在命令行下输入bc即进入计算器程序，而我们想在程序中直接进行浮点数计算时，利用一个简单的管道即可解决问题。
    ```shell
    var=1;
    var=`echo "$var+1"|bc`;
    echo $var
    ```
    * bc支持除位操作运算符之外的所有运算符。
    * bc中要使用scale进行精度设置，如scale=2设置小数点2位精度

6. `awk`

    ```shell
        var=1;
        var=`echo "$var 1"|awk '{printf("%g",$1+$2)}'`;
        echo $var
    ```  
    * awk支持除位操作运算符之外的所有运算符
    * awk内置有log、sqr、cos、sin等等函数

### 关系运算符
关系运算符只支持数字, 不支持字符串, 除非字符串的值是数字

| 表达式 | 含义 |
| :-: | --- |
| `int1 -eq int2` | 两数值相等(equal) |
| `int1 -ne int2` | 两数值不等(not equal) |
| `int1 -gt int2` | n1大于n2(greater than) |
| `int1 -lt int2` | n1小于n2(less than) |
| `int1 -ge int2` | n1大于等于n2(greater than or equal) |
| `int1 -le int2` | n1小于等于n2(less than or equal) |

示例:
```shell
a=100
b=100
if [ $a -eq $b ]
then
    echo '$a -eq $b : a 等于 b'
else
    echo '$a -eq $b : a 不等于 b'
fi

# 10 -eq 10: a 等于 b
```

### 布尔运算符

| 运算符 | 说明 |
| :-: | :-: |
| ! | 非运算 |
| -o | 或运算 |
| -a | 与运算 |

示例:
```shell
a=10
b=20

if [ $a != $b ]
then
   echo "$a != $b : a 不等于 b"
else
   echo "$a != $b: a 等于 b"
fi
# 10 != 20 : a 不等于 b

if [ $a -lt 100 -a $b -gt 15 ]
then
   echo "$a 小于 100 且 $b 大于 15 : 返回 true"
else
   echo "$a 小于 100 且 $b 大于 15 : 返回 false"
fi
# 10 小于 100 且 20 大于 15 : 返回 true
```

### 逻辑运算符

| 运算符 | 说明 |
| :-: | :-: |
| && | AND |
| \|\| | OR |

```shell
a=10
b=20

if [[ $a -lt 100 && $b -gt 100 ]]
then
   echo "返回 true"
else
   echo "返回 false"
fi
# 返回 false

if [[ $a -lt 100 || $b -gt 100 ]]
then
   echo "返回 true"
else
   echo "返回 false"
fi
# 返回 true
```
**注意使用的是双括号**

### 字符串运算符

| 运算符 | 说明 |
| :-: | :-: |
| = | 检测两个字符串是否相等 |
| != | 检测两个字符串是否不相等 |
| -z | 检测字符串长度是否为0 |
| -n | 检测字符串长度是否不为0 |
| str | 检测字符串是否不为空 |
| `str1 > str2` | str1字母顺序是否大于str2, 若大于, 则返回true |
| `str1 < str2` | str1字母顺序是否小于str2, 若小于, 则返回true |

```shell
a="abc"
b="efg"

if [ $a != $b ]
then
   echo "$a != $b : a 不等于 b"
else
   echo "$a != $b: a 等于 b"
fi
# abc != efg : a 不等于 b

if [ -z $a ]
then
   echo "-z $a : 字符串长度为 0"
else
   echo "-z $a : 字符串长度不为 0"
fi
# -z abc : 字符串长度不为 0

if [ -n "$a" ]
then
   echo "-n $a : 字符串长度不为 0"
else
   echo "-n $a : 字符串长度为 0"
fi
# -n abc : 字符串长度不为 0

if [ $a ]
then
   echo "$a : 字符串不为空"
else
   echo "$a : 字符串为空"
fi
# abc : 字符串不为空
```

### 文件测试运算符
文件类检测:

| 表达式 | 含义 |
| :-: | --- |
| `-e file` | 文件是否存在(exist) |
| `-f file` | 文件是否存在且为普通文件(file) |
| `-d file` | 文件是否存在且为目录(directory) |
| `-b file` | 文件是否存在且为块设备block device |
| `-c file` | 文件是否存在且为字符设备character device |
| `-S file` | 文件是否存在且为套接字文件Socket |
| `-p file` | 文件是否存在且为命名管道文件FIFO(pipe) |
| `-L file` | 文件是否存在且是一个链接文件(Link) |

文件属性检测:

| 表达式 | 含义 |
| :-: | --- |
| `-r file` | 文件是否存在且当前用户可读 |
| `-w file` | 文件是否存在且当前用户可写 |
| `-x file` | 文件是否存在且当前用户可执行 |
| `-u file` | 文件是否存在且设置了SUID |
| `-g file` | 文件是否存在且设置了SGID |
| `-k file` | 文件是否存在且设置了sbit(sticky bit) |
| `-s file` | 文件是否存在且大小大于0字节，即用于检测文件是否为非空白文件 |
| `-N file` | 文件是否存在，且自上次read后是否被modify |

示例:
```shell
if [ -x $file ]
then
   echo "文件可执行"
else
   echo "文件不可执行"
fi

if [ -f $file ]
then
   echo "文件为普通文件"
else
   echo "文件为特殊文件"
fi

if [ -s $file ]
then
   echo "文件不为空"
else
   echo "文件为空"
fi

if [ -e $file ]
then
   echo "文件存在"
else
   echo "文件不存在"
fi
```

两个文件之间比较:

| 表达式 | 含义 |
| :-: | --- |
| `file1 -nt file2` | (newer than)判断file1是否比file2新 |
| `file1 -ot file2` | (older than)判断file1是否比file2旧 |
| `file1 -ef file2` | (equal file)判断file2与file2是否为同一文件, 可用在判断hard link的判定上|

示例:
```shell
cd /bin
if test -e ./bash
then
    echo '文件已存在!'
else
    echo '文件不存在!'
fi
```
## echo printf 命令
echo显示换行:
```shell
echo -e "OK! \n" # -e 开启转义
echo "It is a test"
```

结果:
```
OK!

It is a test
```

echo显示不换行:
```shell
echo -e "OK! \c" # -e 开启转义 \c 不换行
echo "It is a test"
```
结果:
```
OK! It is a test
```

默认`printf`不会像`echo`自动添加换行符, 我们可以手动添加 `\n`
```shell
$ echo "Hello, Shell"
Hello, Shell
$ printf "Hello, Shell\n"
Hello, Shell
```

使用格式替换符:
```shell
printf "%-10s %-8s %-4s\n" 姓名 性别 体重kg  
printf "%-10s %-8s %-4.2f\n" 郭靖 男 66.1234
printf "%-10s %-8s %-4.2f\n" 杨过 男 48.6543
printf "%-10s %-8s %-4.2f\n" 郭芙 女 47.9876
```
结果:
```
姓名     性别   体重kg
郭靖     男      66.12
杨过     男      48.65
郭芙     女      47.99
```
`%s` `%c` `%d` `%f`都是格式替代符
`%-10s` 指一个宽度为10个字符(`-`表示左对齐, 没有则表示右对齐), 任何字符都会被显示在10个字符宽的字符内, 如果不足则自动以空格填充, 超过也会将内容全部显示出来
`%-4.2f` 指格式化为小数, 其中.2指保留2位小数
```shell
# format-string为双引号
printf "%d %s\n" 1 "abc"        
#1 abc

# 单引号与双引号效果一样
printf '%d %s\n' 1 "abc"        
#1 abc

# 没有引号也可以输出
printf %s abcdef            
# 格式只指定了一个参数，但多出的参数仍然会按照该格式输出，format-string 被重用
printf %s abc def               
printf "%s\n" abc def
#abcdefabcdefabc
#def

printf "%s %s %s\n" a b c d e f g h i j
#a b c
#d e f
#g h i
#j  

# 如果没有 arguments，那么 %s 用NULL代替，%d 用 0 代替
printf "%s and %d \n"
# and 0
```

## test命令
`test`命令用于检查某个条件是否成立, 它可以进行数字, 字符串和文件三种测试

示例:
```shell
# 两个整数之间的比较, 支持正负数, 但不支持小数.
num1=100
num2=100
if test $[num1] -eq $[num2]
then
    echo '两个数相等！'
else
    echo '两个数不相等！'
fi

str1="ru1noob"
str2="runoob"
if test $str1 = $str2
then
    echo '两个字符串相等!'
else
    echo '两个字符串不相等!'
fi

cd /bin
if test -e ./notFile -o -e ./bash
then
    echo '至少有一个文件存在!'
else
    echo '两个文件都不存在'
fi
```

## 各种括号的作用() (()) [] [[]]
```shell
$ type [ [[ test
[ is a shell builtin
[[ is a shell keyword
test is a shell builtin

if [[ $a != 1 && $a != 2 ]]
if [ $a -ne 1] && [ $a != 2 ]
# 或者
if [ $a -ne 1 -a $a != 2 ]
```

### test 和 [] 用法

它们根据参数的个数来完成测试, 例:
- 不带任何参数, 返回false.

    ```shell
    $ [  ];echo $?
    1
    ```

- 只有一个参数时, 仅当参数非空是返回true

    ```shell
    $ test "";echo $?
    1
    $ test "a";echo $?
    0
    ```

- 两个参数时, 有几种情况:
    - 第一个参数是单目运算符, 包括文件测试, 如: `[ -e file ]` 和 `[ -n str ]`
    - 第一个参数是`!`时, 则只能是`[ ! str ]`, 等价于`[ ! -n str ]`
    - 第一个参数不是任何有效的操作符, 则直接报错
- 三个参数时, 也有几种情况:
    - 使用了双目预运算符, 如: `[ file1 -nt file2 ]`, `[ int1 -eq int2 ]`
    - 使用了逻辑运算符, 如: `[ ! -e file ]`, `[ ! -z str ]`
    - 使用了括号, 则只能是`[ (string) ]`


### `[[]] `用法
用法与`[]`基本相同, 但要注意一下几点:
- 当条件表达式中使用`==`或`!=`时, 该运算符的右边会被当做pattern被匹配, `==`表示能匹配成功则返回0, `!=`则相反. 但只是通配符匹配, 不支持正则. 通配符包括`*`, `?` 和 `[...]`

    ```shell
    $ [[ abc == a* ]];echo $?
    0
    $ [[ abc == a*d ]];echo $?
    1
    $ [[ abc == a[bB]c ]];echo $?
    0
    ```

- 当条件表达式中使用的运算符是"=~"时, 该运算符的右边会被当做正则表达式的pattern被匹配

    ```shell
    $ [[ abc =~ aa* ]];echo $?
    0
    $ [[ abc =~ aa.* ]];echo $?
    1
    ```

- 除了可以使用逻辑运算符`!`和`()`, 还可以使用`&&`, `||`, 等价于`[]`的`-a`和`-o`, 但是`[[]]`不在支持`-a`和`-o`

    ```shell
    $ [[ 3 -eq 3 && 5 -eq 5 ]];echo $?
    0
    ```

### 使用建议
1. 无论是`[]`还是`[[]]`, 建议对其中的变量, 字符串使用双引号包裹. 能做字符串比对是, 不要使用数组比较.

    ```shell
    name="Tom Hanks"
    [ $name = "Tom Hanks" ]
    ```
    上面的语句会报错, 因为在变量替换阶段, `$name`会被替换为Tom Hanks, 但它们没有在引号内, 于是进行单词拆分, 导致等价于执行`[ Tom Hanks = "Tom Hanks" ]`. 所以应该这么写:

    ```shell
    [ "$name" = "Tom Hanks" ]
    ```

2. 数值比对时, 建议双方都加0, 避免变量为空时报错.

    ```shell
    $ [ $a -eq 7 ]
    -bash: [: -eq: unary operator expected

    $ [ $((a+0)) -eq 7 ]; echo $?  # 或者用字符串方式 [ "$a" = "7" ]
    1
    ```

3. 在变量可能为空时, 建议在变量的基础上增加其他辅助字符串. 这比上面的方法更安全

    ```shell
    $ [ "a$a" = "a7" ]   # 判断a是否为7
    $ [ "a$a" = "a" ]    # 判断a是否为空
    $ [ ! -z "$a" -a "a$a" = "a7" ]  # a不为空且a=7时才为真
    ```

## 流程控制

### if else
语法格式如下:
```shell
if commands1
then
    commands2
elif
    commands3
else
    commands4
fi
```
如果执行`commands1`后的退出码是0, 则执行`then`部分的`commands2`. 退出码非0则执行`elif`部分的`commands3`, 以此类推. `commands1`部分有如下几种写法:
- `if [ condition ]`: 等价于`test`, 在所有的POSIX shell上都能用. 注意`condition`与括号间必须有空格
- `if [[ condition ]]`: 基本等价于`[]`, 但支持更多的条件表达式, 且允许使用逻辑运算符"&&" "||" "!" 和 "()", 还支持正则表达式匹配. 较新的bash和zsh都支持.
- `if ((condition))`:
- `if (command)`: 以子shell的方式运行`command`
- `if command`: 根据命令执行后的返回码为判断依据

### for
语法及示例:
```shell
for bar in item1 item2 ... itemN
do
    command1
    command2
done

# 写成1行
for bar in item1 item2 ... itemN; do command1; command2;.. done;

# 遍历数字, 4种方式
for loop in 1, 2 3 4;
# for ((loop=1; loop<=4; loop++));
# for loop in $(seq 1 4);
# for loop in {1..4};
do
    echo "value is: $loop"
done
# 输出
value is: 1
value is: 2
value is: 3

# 遍历字符串
str="This is a string"
for s in $str; do
    echo $s "--"
done
# 输出
This --
is --
a --
string --

# 遍历命令输出
for i in `ls`;  
do   
echo $i is file name\! ;  
done

# 遍历文件夹下文件
for file in ./*; do
    if [ -f $file ]; then
        echo $file "是文件"
    elif [ -d $file ]; then
        echo $file "是目录"
    else
        echo $file
    fi
done

# 无限循环
for ((;;)); do
    command
done
```

### while
```shell
while condition
do
    command
done

# 循环5次
i=0
while [ $i -lt 5 ]; do
    echo $i
    i=$(($i+1))
done
0
1
2
3
4

# 无限循环
while ((1)); do
    command
done
# 方式2
while [ 1 ]; do
    command
done
# 方式3
while :; do
    command
done
# 方式4
while true; do
    command
done

# 读取文件
while read line; do
    echo $line
done < "/etc/passwd"
# 也可以这样写
cat /etc/passwd | {
while read line; do
    echo $line
done
}
```

### until
语法格式:
```shell
until condition
do
    command
then
```
`until`循环执行一系列命令, 直到`condition`为`true`时停止. 和`while`循环刚好相反.
```shell
#!/bin/bash

a=0

until [ ! $a -lt 10 ]
do
   echo $a
   a=`expr $a + 1`
done
```

### case
语法格式:
```shell
case 值 in
模式1)
    command
    ;;
模式2)
    command
    ;;
*)
    command
    ;;        
esac    
```
示例:
```shell
echo '输入 1 到 3 之间的数字:'
echo '你输入的数字为:'
read aNum
case $aNum in
    1)  echo '你选择了 1'
    ;;
    2)  echo '你选择了 2'
    ;;
    3)  echo '你选择了 3'
    ;;
    *)  echo '你没有输入 1 到 4 之间的数字'
    ;;
esac

name=`basename $0 .sh`
case $1 in
 s|start)
        echo "start..."
        ;;
 stop)
        echo "stop ..."
        ;;
 reload)
        echo "reload..."
        ;;
 *)
        echo "Usage: $name [start|stop|reload]"
        exit 1
        ;;
esac
exit 0

# 判断输入文件是文件还是目录
option="${1}"  
case ${option} in   
    -f) file="${2}"  
    echo "file name is $file"  
    ;;  
    -d) dir="${2}"  
    echo "dir name is $dir"  
    ;;  
    *)echo "basename ${0} :usage:[-f file ]| [-d  directory]"  
    exit 1  
    ;;  
esac  
```
注意:
- `*)`相当于其他语言中的`default`
- 除了`*)`模式, 其他分支中的`;;`是必须的, 相当于其他语言中的`break`
- `|`分隔多个模式, 相当于`or`

### break continue
```shell
while :
do
    echo -n "输入 1 到 5 之间的数字:"
    read aNum
    case $aNum in
        1|2|3|4|5) echo "你输入的数字为 $aNum!"
        ;;
        *) echo "你输入的数字不是 1 到 5 之间的! 游戏结束"
            break
        ;;
    esac
done
```
跳出多次循环时使用`break n`, n表示要跳出的循环层数, 默认为1.
```shell
a=1
while [ $a -le 5 ]
do
   echo "Outer loop:a=$a"
   a=$[$a+1]
   for val in 1 2 3 4 5
   do
        if [ $val -eq 3 ]
        then
            break 2
        fi
        echo "	Inner loop:val=$val"
        val=$[$val+1]
   done
done
```
continue仅跳出本次循环
```shell
a=1
for val in 1 2 3 4 5
do
  if [ $val -le 3 ]
  then
       continue
  fi
  echo "val=$val"
done
```

## 函数
函数定义格式如下:
```shell
[function] funcName [()]
{
    //do something
    [return int;]
}
```
说明:
- 可以使用`function foo()`定义, 也可以直接`foo()`定义
- 可以加`return xx;`返回, xx的值为0-255. 如果没有return, 将以最后一条命令运行结果作为返回值.

```shell
#!/bin/bash
foo() {
    return $((1 + 2));
}
foo
echo "函数返回值为 $?"
```
### 函数参数
```shell
paramFunc() {
    echo "第一个参数为 $1 !"
    echo "第二个参数为 $2 !"
    echo "第十个参数为 $10 !"
    echo "第十个参数为 ${10} !"
    echo "第十一个参数为 ${11} !"
    echo "参数总数有 $# 个!"
    echo "作为一个字符串输出所有参数 $* !"
}
paramFunc 1 2 3 4 5 6 7 8 9 34 73
```
输出:
```
第一个参数为 1 !
第二个参数为 2 !
第十个参数为 10 !
第十个参数为 34 !
第十一个参数为 73 !
参数总数有 11 个!
作为一个字符串输出所有参数 1 2 3 4 5 6 7 8 9 34 73 !
```
注意: **当n>=10时, 需要使用${n}来获取参数**.

当脚本有main函数, 你希望脚本运行时传递的所有命令行参数都能被main函数捕捉到, 可以这么写:
```shell
#!/bin/sh

main() {
    for p in "${@}"; do
        echo $p;
    done
}

main "${@}"
```

执行结果: 
```shell
$ ./test.sh a 12 -x
a
12
-x
```

同理, 当A函数调用B函数时, 希望把调用A时传递的所有参数都传递给B, 也可以这么写
```shell
A() {
    B "${@}"
}

B() {

}
```

## 输入/输出重定向
一个命令通常从标准输入读取输入, 并将其输出写入标准输出. 通常都是终端. 重定向命令列表如下:

| 命令 | 说明 |
| :-: | :-: |
| command > file | 将输出重定向到file |
| command < file | 将输入重定向到file |
| command >> file | 将输出已追加方式重定向到file |
| n > file | 将文件描述符为n的文件重定向到file |
| n >> file | 将文件描述符为n的文件已追加方式重定向到file |
| n >& m | 将输出文件m和n合并 |
| n <& m | 将输入文件m和n合并 |
| << tag | 将开始标记tag和结束标记tag之间的内容作为输入 |

> 文件描述符 0 通常是标准输入(STDIN), 1 是标准输出(STDOUT), 2 是标准错误输出(STDERR)

#### 实例
```shell
$ who > users.txt
$ cat users.txt
tonnyyi  console  Dec 28 09:17
tonnyyi  ttys000  Dec 28 09:28
tonnyyi  ttys004  Dec 29 14:44

# 输出重定向会覆盖文件内容, 如果不希望被覆盖, 可以使用追加的方式
$ echo "hello" >> users.txt
$ cat users.txt
tonnyyi  console  Dec 28 09:17
tonnyyi  ttys000  Dec 28 09:28
tonnyyi  ttys004  Dec 29 14:44
hello

# 输入重定向可以将原本需要从键盘获取输入的命令转移到从文件读取
$ grep tonnyyi < user.txt
tonnyyi  console  Dec 28 09:17
tonnyyi  ttys000  Dec 28 09:28
tonnyyi  ttys004  Dec 29 14:44

# 同时替换输入和输出, 执行command, 从infile读取内容, 然后将输出写入到outfile
$ command < infile > outfile
```
一般情况下, 每个Unix/Linux命令运行时都会打开3个文件
- 标准输入文件(stdin): 其文件描述符为**0**, Unix程序默认从中读取数据
- 标准输出文件(stdout): 其文件描述符为**1**, Unix程序默认向其输出数据
- 标准错误文件(stderr): 其文件描述符为**2**, Unix程序会向stderr流中写入错误信息

如果希望stderr重定向到file, 可以这样写:
```shell
$ command 2 > file
```
如果希望将stdout和stderr合并后重定向到file, 可以这样写:
```shell
$ command > file 2>$1
```
#### /dev/null 文件
`/dev/null`是一个特殊文件, 写入到它的内容都会被丢弃; 如果尝试从该文件读取内容, 则什么也读不到. 将命令的输出重定向到它, 会起到"禁止输出"的效果. 如果希望屏蔽stdout和stderr, 可以这样写:
```shell
$ command > /dev/null 2>&1
```

## 文件包含
和其他语言一样, Shell 也可以包含外部脚本, 这样可以很方便的封装一些公用的代码作为一个独立的文件. 语法格式如下:
```shell
. filename      # 点号与文件名中间有一个空格
或
source filename
```
#### 实例
`test1.sh`内容如下:
```shell
#!/bin/bash

url="http://www.baidu.com"
```
`test2.sh`内容如下:
```shell
#!/bin/bash

. ./test1.sh   # 或者 source ./test1.sh
echo "百度地址: $url"
```

## 其他
### 遍历命令结果
字段分隔符(Internal Field Separator, `IFS`)是shell脚本中的一个重要概念, 处理文本数据的时候非常的有用, 是把单个数据流划分成不同数据元素的定界符. 系统环境默认的IFS是空白字符(换行符, 制表符或者空格)
```shell
$ jps -l
88483 ailegal-case.jar
85666 ailegal-center.jar
85797 ailegal-gateway.jar

pros=($(jps -l))
for pro in ${pros[@]}; do
echo $pro
done
```
输出:
```
88483
ailegal-case.jar
85666
ailegal-center.jar
85797
ailegal-gateway.jar
```
如果想以换行符作为分隔符, 则可以这么做:
在Bash 4.0(`bash -version`)之前:
```shell
ls -l | while read x; do echo $x; done
#或者
(IFS='
'
for x in `ls -l`; do echo $x; done)
#或者
while read x; do echo $x; done << EOF
$(ls -l)
EOF
```
如果是在函数内, 可以使用`local`命令, 避免全局性的修改IFS
```shell
#!/bin/sh
test() {
   local IFS=$'\n'
   local lines=$(ls -l)
   for line in $lines; do
       echo $line
   done
}

test
```

4.0以后可以使用`mapfile`:
```shell
mapfile -t files < <(ls -l)
for file in "${files[@]}"; do
    echo $file
done
```
