---
title: tmux
tags:
  - linux
  - tmux
categories:
  - linux
date: 2022-03-07 14:32:42
---

通常使用ssh远程登录服务器进行管理, 但是当网络突然断开, 再次登录时, 上次的会话已经终止, 里面的进程消失了(比如正在下载文件, 运行服务). 这就是**会话**的特点: 窗口和其中启动的进程是绑定在一起的. 为了解绑, 选择使用**Tmux**



### Tmux的作用

1. 允许单个窗口中同时访问多个会话.
2. 可以让新窗口接入已经存在的会话
3. 允许每个会话有多个连接窗口, 因此可以多人实时共享会话
4. 支持窗口任意的垂直或水平拆分



## 基本用法

### 安装

```bash
# Ubuntu 或 Debian
$ sudo apt-get install tmux

# CentOS 或 Fedora
$ sudo yum install tmux

# Mac
$ brew install tmux
```

### 启动退出

```bash
# 启动
tmux

# 退出
exit
# 或者按下 ctrl + d
```

### 前缀键

Tmux中有大量的快捷键, 所有的快捷键都通过前缀建唤起. 默认的前缀键是`Ctrl+b`. 举例来说, 帮助命令的快捷键是`Ctrl+b ?`, 即在Tmux窗口中, 先按下`Ctrl b`, 再按下`?`就会显示帮助信息. 然后按下ESC或`q`, 就可以退出帮助.



## 会话管理

### 新建会话

第一个启动的Tmux窗口, 编号是0, 但是编码不好区分, 建议给会话起个名字.

```bash
tmux new -s 会话名 -n 窗口名

Ctrl+b  :new sessionName
```

### 分离会话

```bash
Ctrl+b  d
tmux detach
```

执行上面的命令后, 就会退出当前Tmux窗口, 但是会话和里面的进程仍然在后台运行

### 会话查看

```bash
tmux ls
tmux list-session
```

### 接入会话

```bash
tmux attach -t 0
tmux attach -t 会话名
```

### 杀死会话

```bash
tmux kill-session -t 0
tmux kill-session -t 会话名
```

### 切换会话

```bash
Ctrl+b  s
tmux switch -t 0
tmux switch -t 回话名
```

### 重命名回话

```bash
Ctrl+b  $
tmux rename-session -t 0 新名称
```

### 会话快捷键

```
ctrl+b d: 分离会话
ctrl+b s: 列出所有会话
ctrl+b $: 重命名当前会话
```

### 列出当前所有 Tmux 会话的信息

```bash
 $ tmux info
```



## 窗口管理

### 新建窗口

```bash
Ctrl+b c
tmux new-window
tmux new-window -n 名称
```

### 切换窗口

```bash
# 切换到上一个窗口
Ctrl+b p
# 切换到下一个窗口
Ctrl+b n
# 切换到指定编号的窗口
Ctrl+b <number>
# 从列表中选择窗口	tmux select-window -t 名称或编号
Ctrl+b w
```

### 重命名窗口

```bash
# 窗口重命名		tmux rename-window 新名称
Ctrl+b ,
```

### 关闭窗口

```bash
Ctrl+b  &
```



## 窗格操作

### 拆分窗格

```bash
# 拆分成上下两个窗格	tmux split-window
Ctrl+b "

# 拆分成左右两个窗格	tmux split-window -h
Ctrl+b %
```

### 移动光标

```bash
# 切换到上方窗格		tmux select-pane -U
Ctrl+b 上

# 切换到下方窗格		tmux select-pane -D
Ctrl+b 下

# 切换到左侧窗格		tmux select-pane -L
Ctrl+b 左

# 切换到右侧窗格		tmux select-pane -R
Ctrl+b 右

# 光标切换到上一个窗格
Ctrl+b ;  

# 光标切换到下一个窗格
Ctrl+b o
```

### 交换窗格位置

```bash
# 当前窗格上移
tmux swap-pane -U

# 当前窗格下移
tmux swap-pane -D

# 当前窗格与上一个窗格交换位置  
Ctrl+b {
# 当前窗格与下一个窗格交换位置	
Ctrl+b }
```

### 其他窗格快捷键

```
Ctrl+b x：关闭当前窗格。

Ctrl+b z：当前窗格全屏显示，再使用一次会变回原来大小。
Ctrl+b Ctrl+<arrow key>：按箭头方向调整窗格大小。

Ctrl+b Ctrl+o：所有窗格向前移动一个位置，第一个窗格变成最后一个窗格。
Ctrl+b Alt+o：所有窗格向后移动一个位置，最后一个窗格变成第一个窗格。

Ctrl+b !：将当前窗格拆分为一个独立窗口。
Ctrl+b q：显示窗格编号。
```



## 快捷键

按下前缀键(默认`ctrl b`), 再按下面的按键

| 模块             | 快捷键                                                       |
| ---------------- | ------------------------------------------------------------ |
| 会话             | :new<回车>  启动新会话 <br />s           列出所有会话<br />$           重命名当前会话 |
| 窗口 (标签页)    | c  创建新窗口<br />w  列出所有窗口<br />n  后一个窗口 <br />p  前一个窗口 <br />f  查找窗口 <br />,  重命名当前窗口 <br />&  关闭当前窗口 |
| 调整窗口排序     | swap-window -s 3 -t 1  交换 3 号和 1 号窗口 <br />swap-window -t 1       交换当前和 1 号窗口 <br />move-window -t 1       移动当前窗口到 1 号 |
| 窗格（分割窗口） | %  垂直分割<br/>"  水平分割<br/>o  交换窗格<br/>x  关闭窗格<br/>⍽  左边这个符号代表空格键 - 切换布局<br/>q 显示每个窗格是第几个，当数字出现的时候按数字几就选中第几个窗格<br/>{ 与上一个窗格交换位置<br/>} 与下一个窗格交换位置<br/>z 切换窗格最大化/最小化 |

