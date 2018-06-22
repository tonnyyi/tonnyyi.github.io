---
title: shadowsocks
date: 2018-04-30 19:55:16
tags:
    - shadowsocks
categories:
    - shadowsocks
---

#### 安装系统依赖
```bash
yum install -y openssl-devel gcc swig python-devel autoconf libtool
yum groupinstall "Development Tools"
```

#### 安装谷歌BBR
网络拥塞算法, 该脚本会自动升级内核
```bash
wget --no-check-certificate https://github.com/teddysun/across/raw/master/bbr.sh
chmod +x bbr.sh
./bbr.sh
```
<!-- more -->
#### 系统优化
```bash
vi /etc/sysctl.conf
```
内容如下:

```
# Kernel sysctl configuration file for Red Hat Linux
#
# For binary values, 0 is disabled, 1 is enabled.  See sysctl(8) and
# sysctl.conf(5) for more details.
#
# Use '/sbin/sysctl -a' to list all possible parameters.

# Controls IP packet forwarding
# net.ipv4.ip_forward = 0

# Controls source route verification
net.ipv4.conf.default.rp_filter = 1

# Do not accept source routing
net.ipv4.conf.default.accept_source_route = 0

# Controls the System Request debugging functionality of the kernel
kernel.sysrq = 0

# Controls whether core dumps will append the PID to the core filename.
# Useful for debugging multi-threaded applications.
kernel.core_uses_pid = 1

# Controls the use of TCP syncookies
net.ipv4.tcp_syncookies = 1

# Controls the default maxmimum size of a mesage queue
kernel.msgmnb = 65536

# Controls the maximum size of a message, in bytes
kernel.msgmax = 65536

# Controls the maximum shared segment size, in bytes
kernel.shmmax = 68719476736

# Controls the maximum number of shared memory segments, in pages
kernel.shmall = 4294967296

# Accept IPv6 advertisements when forwarding is enabled
net.ipv6.conf.all.accept_ra = 2
net.ipv6.conf.eth0.accept_ra = 2

net.core.default_qdisc = fq
net.ipv4.tcp_congestion_control = bbr


# max open files
fs.file-max = 1024000
# max read buffer
net.core.rmem_max = 67108864
# max write buffer
net.core.wmem_max = 67108864
# default read buffer
net.core.rmem_default = 65536
# default write buffer
net.core.wmem_default = 65536
# max processor input queue
net.core.netdev_max_backlog = 4096
# max backlog
net.core.somaxconn = 4096

# reuse timewait sockets when safe
net.ipv4.tcp_tw_reuse = 1
# turn off fast timewait sockets recycling
# net.ipv4.tcp_tw_recycle = 0
# short FIN timeout
net.ipv4.tcp_fin_timeout = 30
# short keepalive time
net.ipv4.tcp_keepalive_time = 1200
# outbound port range
net.ipv4.ip_local_port_range = 10000 65000
# max SYN backlog
net.ipv4.tcp_max_syn_backlog = 4096
# max timewait sockets held by system simultaneously
net.ipv4.tcp_max_tw_buckets = 5000
# TCP receive buffer
net.ipv4.tcp_rmem = 4096 87380 67108864
# TCP write buffer
net.ipv4.tcp_wmem = 4096 65536 67108864
# turn on path MTU discovery
net.ipv4.tcp_mtu_probing = 1

# for high-latency network
# forward ipv4
net.ipv4.ip_forward = 1

# turn on TCP Fast Open on both client and server side
net.ipv4.tcp_fastopen = 3
```
保存生效
```bash
sysctl -p
```
打开防火墙端口
```bash
service iptables status
iptables -I INPUT -p tcp --dport 443 -j ACCEPT
ptables -I INPUT -p udp --dport 443 -j ACCEPT
```

#### 脚本一键安装
https://teddysun.com/486.html
```bash
wget --no-check-certificate https://raw.githubusercontent.com/teddysun/shadowsocks_install/master/shadowsocksR.sh
chmod +x shadowsocksR.sh
./shadowsocksR.sh 2>&1 | tee shadowsocksR.log
```

#### 配置
```bash
vi /etc/shadowsocks.json
```
文件内容
```
{
    "server":"0.0.0.0",
    "server_ipv6":"[::]",
    "server_port":443,
    "local_address":"127.0.0.1",
    "local_port":1080,
    "password":"密码",
    "timeout":120,
    "method":"aes-256-cfb", # 加密方法
    "obfs":"http_simple",   # 混淆算法
    "protocol":"auth_chain_a",  # 协议
    "obfs_param":"",
    "redirect":"",
    "dns_ipv6":false,
    "fast_open":true,
    "workers":1
}
```

#### 管理
```bash
# 重启
/usr/local/shadowsocks/server.py -c /etc/shadowsocks.json -d restart

# 查看流量
yum install -y iftop
iftop
```
