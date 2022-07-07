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

重要的参数有

**网卡流量** Mbps

**每秒收发包数量** pss

**丢包数**

 `netstat -i` 间歇地看RX-DRP RX-OVR

##### 全队列数

全队列满了以后, 后续TCP连接会被丢弃

```bash
# 查看全连接队列信息(listen状态)
# Recv-Q：当前全连接队列的大小，也就是当前已完成三次握手并等待服务端 accept() 的 TCP 连接；
# Send-Q：当前全连接最大队列长度，下面的输出结果说明的是监听8088端口的TCP服务的最大全连接长度
ss -lnt | grep 8080

# 查看全连接队列信息(非listen状态, 即连接已建立)
# Recv-Q：已收到但未被应用进程读取的字节数；
# Send-Q：已发送但未收到确认的字节数；
ss -nt | grep 8080

# 查看全连接队列溢出情况(隔几秒执行一次, 一直涨的话说明全队列在溢出)
netstat -s | grep overflowed
```

##### 半队列数

```bash
# 查看半连接数
netstat -natp | grep SYN_RECV | wc -l

# 查看半连接队列溢出导致TCP连接丢弃情况(隔几秒执行一次, 一直涨的话说明半队列在溢出)
netstat -s | grep 'SYNs to LISTEN'
```



- ip a              全部网卡信息

- netstat -tna | awk '{print $6}' | sort | uniq -c    各状态tcp连接统计

- ethtool enp95s0f0 | grep Speed    查看网卡速率

- **sar -n DEV 1 5**    收发包速度, 网络流量

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

- **iftop -nNB** 查看各IP流量

- cat /var/log/messages     查看内核日志, 确定是否丢包

- watch more /proc/net/dev   定位丢包/错包情况, 重点drop和发出去总量

- watch -n 1 'nstat -z -t 1 | grep -e TcpExtTCPSynRetrans -e TcpRetransSegs  -e TcpOutSegs -e TcpInSegs'    每秒TCP重传报文数(/proc/net/snmp和/proc/net/netstat)

- tcpdump tcp -i eth0 -s 0 -nn -vvv and port 8080 -w tcpdump.cap  抓包8080端口

- tcpdump -i eth0 tcp -s 0 -S -nn -tttt -vv and port 8080  实时查看抓包数据

- iperf3 

  ```bash
  wget https://iperf.fr/download/source/iperf-3.1.3-source.tar.gz
  tar zxvf iperf-3.1.3-source.tar.gz
  cd iperf-3.1.3
  ./configure
  make
  make install
  
  # 1. 在服务器A上启动服务端 
  # -s    表示服务器端；
  # -p    定义端口号；
  # -i    输出测试报告的时间间隔设置每次报告之间的时间间隔，单位为秒，默认值为零
  $ iperf3 -s
  
  # 2. 在服务器BB上执行测试
  # -c    表示服务器的IP地址；
  # -p    表示服务器的端口号；
  # -t    参数可以指定传输测试的持续时间,Iperf在指定的时间内，重复的发送指定长度的数据包，默认是10秒钟.
  # -i    输出测试报告的时间间隔设置每次报告之间的时间间隔，单位为秒，默认值为零
  # -w    设置套接字缓冲区为指定大小，对于TCP方式，此设置为TCP窗口大小，对于UDP方式，此设置为接受UDP数据包的缓冲区大小，限制可以接受数据包的最大值.
  # --logfile    参数可以将输出的测试结果储存至文件中.
  # -J  来输出JSON格式测试结果.
  # -R  反向传输,缺省iperf3使用上传模式：Client负责发送数据，Server负责接收；如果需要测试下载速度，则在Client侧使用-R参数即可.
  $ iperf3 -c 172.31.47.14
  Connecting to host 172.31.47.14, port 5201
  [  4] local 172.31.160.210 port 64246 connected to 172.31.47.14 port 5201
  [ ID] Interval           Transfer     Bandwidth       Retr  Cwnd
  [  4]   0.00-1.00   sec   114 MBytes   954 Mbits/sec    0    359 KBytes
  [  4]   1.00-2.00   sec   113 MBytes   945 Mbits/sec    0    359 KBytes
  [  4]   2.00-3.00   sec   112 MBytes   938 Mbits/sec    0    359 KBytes
  [  4]   3.00-4.00   sec   113 MBytes   945 Mbits/sec    0    359 KBytes
  [  4]   4.00-5.00   sec   112 MBytes   938 Mbits/sec    0    359 KBytes
  [  4]   5.00-6.00   sec   113 MBytes   945 Mbits/sec    0    359 KBytes
  [  4]   6.00-7.00   sec   112 MBytes   938 Mbits/sec    0    359 KBytes
  [  4]   7.00-8.00   sec   113 MBytes   945 Mbits/sec    0    359 KBytes
  [  4]   8.00-9.00   sec   112 MBytes   938 Mbits/sec    0    359 KBytes
  [  4]   9.00-10.00  sec   113 MBytes   945 Mbits/sec    0    359 KBytes
  - - - - - - - - - - - - - - - - - - - - - - - - -
  [ ID] Interval           Transfer     Bandwidth       Retr
  [  4]   0.00-10.00  sec  1.10 GBytes   943 Mbits/sec    0             sender
  [  4]   0.00-10.00  sec  1.10 GBytes   942 Mbits/sec                  receiver
  
  iperf Done.
  
  # 同时服务端也有报告输出
  -----------------------------------------------------------
  Server listening on 5201
  -----------------------------------------------------------
  Accepted connection from 172.31.160.210, port 64244
  [  5] local 172.31.47.14 port 5201 connected to 172.31.160.210 port 64246
  [ ID] Interval           Transfer     Bandwidth
  [  5]   0.00-1.00   sec   108 MBytes   906 Mbits/sec
  [  5]   1.00-2.00   sec   112 MBytes   941 Mbits/sec
  [  5]   2.00-3.00   sec   112 MBytes   942 Mbits/sec
  [  5]   3.00-4.00   sec   112 MBytes   941 Mbits/sec
  [  5]   4.00-5.00   sec   112 MBytes   942 Mbits/sec
  [  5]   5.00-6.00   sec   112 MBytes   941 Mbits/sec
  [  5]   6.00-7.00   sec   112 MBytes   941 Mbits/sec
  [  5]   7.00-8.00   sec   112 MBytes   941 Mbits/sec
  [  5]   8.00-9.00   sec   112 MBytes   942 Mbits/sec
  [  5]   9.00-10.00  sec   112 MBytes   941 Mbits/sec
  [  5]  10.00-10.04  sec  4.36 MBytes   942 Mbits/sec
  - - - - - - - - - - - - - - - - - - - - - - - - -
  [ ID] Interval           Transfer     Bandwidth
  [  5]   0.00-10.04  sec  0.00 Bytes  0.00 bits/sec                  sender
  [  5]   0.00-10.04  sec  1.10 GBytes   938 Mbits/sec                  receiver
  -----------------------------------------------------------
  Server listening on 5201
  -----------------------------------------------------------
  ```

  

## 进程

### CPU
- pidstat -w -p PID     进程上下文切换情况

### 内存

- 

### 磁盘
- lsof -p PID   查看进程打开的文件
- pidstat -d -p PID    查看进程读写磁盘速度

### 网络
- lsof  占用的网络端口
- `lsof -i tcp -naP -p PID`      查看进程打开的tcp连接  
- `lsof -i:9981 -P -t -sTCP:LISTEN`  根据端口查找进程ID
- nethogs       需要安装, 按进程实时统计带宽使用率
- iftop -nNB -m 10M     需要安装, 按p:显示端口 t:合并接收发送 L:显示刻度 T:显示总量 B:根据最近2/10/40s排序

### 线程
- top -H -p PID     查看进程的所有线程
- pstack PID 进程堆栈
- pidstat -t -p PID     进程的线程信息
- strace COMMAND    跟踪进程系统调用
