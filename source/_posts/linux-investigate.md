---
title: linux系能瓶颈排查
tags:
  - linux
  - performance
categories:
  - linux
date: 2021-08-23 16:53:44
---

## 系统

### CPU负载
- top   1/5/15分钟负载  M/P:按内存/CPU排序
- sar -q 1 3     CPU负载
- lscpu     查看CPU物理颗数/核数/逻辑数

### CPU使用率
负载高但使用率低, 说明IO较多
- top
- sar -u 1 3

### 内存  
- free -h
- vmstat -Sm 1 3    内存统计(物理/swap)
- sar -r 1 3    内存使用率
- sar -B 1 3    内存页换入换出缺页
- sar -W 1 3    swap换入换出   
- ps aux | awk 'NR!=1{a[$1]+=$6;} END { for(i in a) print i ", " a[i]"KB"; }'   查看各用户所占用内存

### 磁盘IO
- fdisk -l  查看磁盘详细信息
- df -lh    查看磁盘挂载点可用空间
- lsblk   查看磁盘, 分区容量和挂载点
- du -sh PATH/*  查看各文件/目录占用磁盘大小
- iostat -x -k -d 1 3    io读写数据量统计
- iotop -P     需要安装, 查看进程io情况
- sar -b 1 3    IO请求数及读写块数
- sar -d 1 3    块设备使用情况

### 文件        
- lsof FILE | awk 'NR!=1 {print $2}' | uniq     文件被谁占用
- ls -l *.cpp *.c | awk '{sum+=$5} END {print sum}'     统计文件大小

### 网络
- ip a              全部网卡信息
- netstat -tna | awk '{print $6}' | sort | uniq -c    各状态tcp连接统计
- ethtool enp95s0f0 | grep Speed    查看网卡速率
- sar -n DEV 1 3    收发包速度
- sar -v EDEV 1 3   网络设备通信失败情况
- sar -n IP 1 3     IP层统计
- sar -n EIP 1 3    IP层错误统计
- sar -n TCP 1 3    TCP层统计
- sar -n ETCP 1 3   TCP层错误统计
- sar -n SOCK 1 3   socket连接信息
- ss -s      各种连接(tcp/udp...)统计
- ss -nat    显示所有的tcp连接
- ss -nt dst :8080   限制连接到指定端口的socket
- ss src IP    显示来自指定IP的连接
- cat /var/log/messages     查看内核日志, 确定是否丢包
- watch more /proc/net/dev   定位丢包/错包情况, 重点drop和发出去总量
- watch -n 1 'nstat -z -t 1 | grep -e TcpExtTCPSynRetrans -e TcpRetransSegs  -e TcpOutSegs -e TcpInSegs'    每秒TCP重传报文数(/proc/net/snmp和/proc/net/netstat)



## 进程

### CPU
- pidstat -w -p PID     进程上下文切换情况

### 内存

### 磁盘
- lsof -p PID   查看进程打开的文件
- pidstat -d -p PID    查看进程读写磁盘速度

### 网络
- lsof  占用的网络端口
- lsof -i tcp -naP -p PID      查看进程打开的tcp连接  
- nethogs       需要安装, 按进程实时统计带宽使用率
- iftop -nNB -m 10M     需要安装, 按p:显示端口 t:合并接收发送 L:显示刻度 T:显示总量 B:根据最近2/10/40s排序

### 线程
- top -H -p PID     查看进程的所有线程
- pstack PID 进程堆栈
- pidstat -t -p PID     进程的线程信息
- strace COMMAND    跟踪进程系统调用
