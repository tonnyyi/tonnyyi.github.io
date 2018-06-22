---
title: Linux之hostname那些事
tags:
  - host
categories:
  - Linux
  - Hostname
date: 2018-02-28 16:35:16
---




#### 查看当前hostname

```bash
hostname
uname -n
cat /proc/sys/kernel/hostname
sysctl kernel.hostname
```
<!-- more -->
#### `hostname`命令

root权限, 立即生效, 对所有新连接有效, 重启失效

#### /etc/hosts

本地dns, ::1是IPv6格式的localhost, 当ping一个host时会先查找该文件, 如果没有则查询DNS服务器

#### /etc/hostname

首先读取该文件, 只能保存hostname, 而且只能是有一个hostname

#### /etc/sysconfig/network



#### /proc/sys/kernel/hostname



启动时读取/etc/hostname赋值到内核参数kernel.hostname, 该内核参数保存在/proc/sys/kernel/hostname. 运行时更新/etc/hostname, sysctl kernel.hostname=xxx, hostname xxx都会更新/proc/sys/kernel/hostname, 即系统内核hostname参数被更新.



不同的Linux发行版都有一个脚本, 用来在系统启动时设置hostname. 不同发行版**永久**更改hostname的方式如下:

- **Debian** 有一个脚本 `/etc/init.d/hostname.sh`, 该脚本会在启动时读取`/etc/hostname`的内容并赋值为hostname. 可以将要修改的hostname值写入`/etc/hostname`, 然后使用root执行`/etc/init.d/hostname.sh`.

- **Ubuntu** 和Debian一样, 也是使用`/etc/hostname`保存hostname, 但是需要用root执行`service hostname start`来使更改生效.

- **Slackware** 使用的是`/etc/HOSTNAME`,设置好hostname以后使用root执行`hostname -F /etc/HOSTNAME`.

- **Red Hat** 会在`/etc/sysconfig/network`中查询这样一行

  ```
  HOSTNAME=gauss

  ```

修改完以后则需要重启, 但可以执行`hostname gauss`来临时修改, 这样临时和重启之后hostname就都更改过来了.



----

#### 参考:

- [深入理解Linux修改hostname](http://www.cnblogs.com/kerrycode/p/3595724.html)
- [Linux Hostname Configuration](http://jblevins.org/log/hostname)
- ​