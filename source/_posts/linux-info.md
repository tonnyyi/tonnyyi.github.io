---
title: Linux系统及硬件信息查看
tags:
  - linux
  - command
categories:
  - linux
  - Hardware
date: 2016-09-02 15:34:31
---

> 物理服务器型号: **Dell R720**

### 服务器
### 命令
- `dmidecode | grep "Product Name"`: 查看服务器型号(使用root执行)
  
    ```
    [root@cluster143 mcc]# dmidecode | grep "Product Name"
    Product Name: PowerEdge R720
    Product Name: 068CDY
    ```
<!-- more -->

`hostnamectl`是systemd的一部分, 可用于查询和更改系统主机名和相关设置

```bash
$ hostnamectl
   Static hostname: localhost.localdomain
         Icon name: computer-server
           Chassis: server
        Machine ID: 688adf6374604cd19edf34943d350fc1
           Boot ID: 7b21be9fca42471ab3405cf111a9fb08
  Operating System: Kylin Linux Advanced Server V10 (Azalea)
       CPE OS Name: cpe:/o:kylin:enterprise_linux:V10:GA:server
            Kernel: Linux 4.19.90-11.ky10.aarch64
      Architecture: arm64

$ hostnamectl
   Static hostname: jzcpx-ocr
Transient hostname: jzcpx-no.01.novalocal
         Icon name: computer-server
           Chassis: server
        Machine ID: 374becb2f4034c0a92ed8a6dcfcf3d76
           Boot ID: 380d45e18482430cad8ce0b8fc264290
  Operating System: CentOS Linux 7 (Core)
       CPE OS Name: cpe:/o:centos:centos:7
            Kernel: Linux 3.10.0-514.el7.x86_64
      Architecture: x86-64
```



### CPU
#### 概念
- **物理CPU**: 实际插槽上的CPU个数, 可以通过查看`cat /proc/cpuinfo`中不重复的*physical id*个数确定
- **逻辑CPU**: 一颗CPU可能有多个核, 如果CPU支持Intel的超线程(HT)则核又多出一倍, 所以`逻辑CPU个数` = `物理CPU个数` \* `单颗CPU的核数 [* 2 (如果CPU支持并开启HT)]`
使用`top`命令看到的就是逻辑cpu个数

#### 命令
- `lscpu`: 查看cpu统计信息

    ```bash
    [mcc@localhost ~]$ lscpu
    Architecture:          x86_64                       # CPU架构
    CPU op-mode(s):        32-bit, 64-bit
    Byte Order:            Little Endian                # 小端
    CPU(s):                72                           # 逻辑CPU个数
    On-line CPU(s) list:   0-71
    Thread(s) per core:    2                            # 每个cpu核能支持2个线程，即支持超线程
    Core(s) per socket:    18                           # 每个cpu有18个物理核
    座：                 2                               # 共2个物理CPU
    NUMA 节点：         2
    厂商 ID：           GenuineIntel                     # CPU产商为Intel
    CPU 系列：          6
    型号：              85
    型号名称：        Intel(R) Xeon(R) Gold 6140 CPU @ 2.30GHz
    步进：              4
    CPU MHz：             2803.394
    BogoMIPS：            4604.79
    虚拟化：           VT-x                             # 支持CPU虚拟化技术
    L1d 缓存：          32K
    L1i 缓存：          32K
    L2 缓存：           1024K
    L3 缓存：           25344K
    NUMA 节点0 CPU：    0-17,36-53
    NUMA 节点1 CPU：    18-35,54-71
    ```
    
- `cat /proc/cpuinfo`
    - CPU型号

        ```bash
        $ cat /proc/cpuinfo | grep 'model name' | sort | uniq
        model name	: Intel(R) Xeon(R) Gold 6140 CPU @ 2.30GHz
        ```
        
    - CPU数量
      
        ```bash
        $ cat /proc/cpuinfo | grep 'physical id' | sort | uniq | wc -l
        2
        ```
    
- `arch` 机器架构信息, 输出结果有：i386、i486、i586、alpha、sparc、arm、m68k、mips、ppc、i686等。
  
    ```bash
    $ arch
    x86_64
    ```
    
- 单颗CPU中物理核数
  
    ```bash
    $ cat /proc/cpuinfo | grep "cores" | uniq
    cpu cores	: 18
    ```
    
    - `getconf LONG_BIT `: 查看当前CPU运行模式是32位还是64位
    
        ```bash
        $ getconf LONG_BIT
        64
        ```

- 查看总的逻辑核数
	```bash
	$ cat /proc/cpuinfo| grep "processor"| wc -l
	72
	```

### 内存
#### 命令
- `free -h`: 查看内存的概要使用信息

    ```
    [root@localhost mcc]# free -h
                 total       used       free     shared    buffers     cached
    Mem:           62G        62G       654M       2.9G         0B        37G
    -/+ buffers/cache:        24G        38G
    Swap:          97G       560M        97G
    ```
    
- `cat /proc/meminfo`: 查看内存详细使用信息

    ```
    [root@localhost mcc]# cat /proc/meminfo
    MemTotal:       65760532 kB
    MemFree:          671632 kB
    MemAvailable:   41169156 kB
    Buffers:               0 kB
    Cached:         42351124 kB
    SwapCached:        48188 kB
    Active:         32159900 kB
    Inactive:       30565408 kB
    Active(anon):   21018736 kB
    Inactive(anon):  2415272 kB
    Active(file):   11141164 kB
    Inactive(file): 28150136 kB
    ...
    ```
- `dmidecode -t memory`: 查看内存硬件信息(需用`root`执行)

    ```
    [root@localhost mcc]# dmidecode -t memory
    # dmidecode 2.12
    SMBIOS 2.7 present.
    
    Handle 0x1000, DMI type 16, 23 bytes
    Physical Memory Array
    	Location: System Board Or Motherboard
    	Use: System Memory
    	Error Correction Type: Multi-bit ECC
    	Maximum Capacity: 1536 GB
    	Error Information Handle: Not Provided
    	Number Of Devices: 24          # 24个内存插槽
    
    Handle 0x1100, DMI type 17, 34 bytes
    Memory Device
    	Array Handle: 0x1000
    	Error Information Handle: Not Provided
    	Total Width: 72 bits
    	Data Width: 64 bits
    	Size: 8192 MB              # 插着一条8192 MB内存
    	Form Factor: DIMM
    	Set: 1
    	Locator: DIMM_A1
    	Bank Locator: Not Specified
    	Type: DDR3                 # DDR3类型
    	Type Detail: Synchronous Registered (Buffered)
    	Speed: 1333 MHz
    	Manufacturer: 00CE04B300CE         # 生产商
    	Serial Number: 835D9
    	Asset Tag: 02104611
    	Part Number: M393B1K7
    	Rank: 2
    	Configured Clock Speed: 1333 MHz   # 内存频率
    ...	
    ```

### 磁盘
`smartctl -a /dev/sda`: 查看硬盘型号

```
[root@localhost mcc]# smartctl -a /dev/sda
smartctl 6.2 2013-07-26 r3841 [x86_64-linux-3.10.0-123.el7.x86_64] (local build)
Copyright (C) 2002-13, Bruce Allen, Christian Franke, www.smartmontools.org
    
=== START OF INFORMATION SECTION ===
Vendor:               DELL
Product:              PERC H310
Revision:             2.12
User Capacity:        999,653,638,144 bytes [999 GB]
Logical block size:   512 bytes
Logical Unit id:      0x6b083fe0c44c3e001bcc591f05b11119
Serial number:        001911b1051f59cc1b003e4cc4e03f08
Device type:          disk
Local Time is:        Fri Sep  2 15:28:58 2016 CST
SMART support is:     Unavailable - device lacks SMART capability.
    
=== START OF READ SMART DATA SECTION ===
    
Error Counter logging not supported
    
Device does not support Self Test logging
```

#### 命令
- `lsblk`: 查看硬盘以及分区

    ```
    [root@localhost mcc]# lsblk
    NAME            MAJ:MIN RM   SIZE RO TYPE MOUNTPOINT
    sda               8:0    0   931G  0 disk
    ├─sda1            8:1    0  1000M  0 part /boot
    └─sda2            8:2    0   930G  0 part
      ├─centos-swap 253:0    0  97.7G  0 lvm  [SWAP]
      └─centos-root 253:1    0 832.4G  0 lvm  /
    sr0              11:0    1  1024M  0 rom
    ```
- `fdisk -l`: 查看磁盘详细信息

    ```
    [root@localhost mcc]# fdisk -l
    
    磁盘 /dev/sda：999.7 GB, 999653638144 字节，1952448512 个扇区
    Units = 扇区 of 1 * 512 = 512 bytes
    扇区大小(逻辑/物理)：512 字节 / 512 字节
    I/O 大小(最小/最佳)：512 字节 / 512 字节
    磁盘标签类型：dos
    磁盘标识符：0x000cc668
    
       设备 Boot      Start         End      Blocks   Id  System
    /dev/sda1   *        2048     2050047     1024000   83  Linux
    /dev/sda2         2050048  1952448511   975199232   8e  Linux LVM
    
    磁盘 /dev/mapper/centos-swap：104.9 GB, 104857600000 字节，204800000 个扇区
    Units = 扇区 of 1 * 512 = 512 bytes
    扇区大小(逻辑/物理)：512 字节 / 512 字节
    I/O 大小(最小/最佳)：512 字节 / 512 字节
    
    
    磁盘 /dev/mapper/centos-root：893.7 GB, 893743267840 字节，1745592320 个扇区
    Units = 扇区 of 1 * 512 = 512 bytes
    扇区大小(逻辑/物理)：512 字节 / 512 字节
    I/O 大小(最小/最佳)：512 字节 / 512 字节
    ```
- `df -h`: 查看各分区使用情况

    ```
    [root@localhost mcc]# df -h
    文件系统                 容量  已用  可用 已用% 挂载点
    /dev/mapper/centos-root  832G  250G  583G   31% /
    devtmpfs                  32G     0   32G    0% /dev
    tmpfs                     32G   80K   32G    1% /dev/shm
    tmpfs                     32G  3.1G   29G   10% /run
    tmpfs                     32G     0   32G    0% /sys/fs/cgroup
    /dev/sda1                997M  126M  872M   13% /boot
    ```
- `du -sh <目录名>`: 查看指定命令的大小

    ```bash
    [root@localhost mcc]# du -sh /tmp/
    2.3G	/tmp/
    
    # 查看目录下文件/文件夹占用空间
$ du -sh /home/*
    12K	/home/centos
    9.1M	/home/elastic
    1.3G	/home/git
    114M	/home/iflytek
    ```
    
    
    ​    
### 网卡
#### 命令
- `lspci | grep -i 'eth'` 或 `dmesg | grep -i eth`: 查看网卡硬件信息

    ```
    [root@localhost mcc]# lspci | grep -i 'eth'
    01:00.0 Ethernet controller: Broadcom Corporation NetXtreme BCM5720 Gigabit Ethernet PCIe
    01:00.1 Ethernet controller: Broadcom Corporation NetXtreme BCM5720 Gigabit Ethernet PCIe
    02:00.0 Ethernet controller: Broadcom Corporation NetXtreme BCM5720 Gigabit Ethernet PCIe
    02:00.1 Ethernet controller: Broadcom Corporation NetXtreme BCM5720 Gigabit Ethernet PCIe
    ```
- `ifconfig -a`: 查看所有网络接口, `em1-4`使用重命名过的, 通过`dmesg | grep -i eth`可用看出来

    ```
    [root@localhost mcc]# ifconfig -a
    bond0: flags=5187<UP,BROADCAST,RUNNING,MASTER,MULTICAST>  mtu 1500
    ...
    em1: flags=6211<UP,BROADCAST,RUNNING,SLAVE,MULTICAST>  mtu 1500
    ...
    em2: flags=6211<UP,BROADCAST,RUNNING,SLAVE,MULTICAST>  mtu 1500
    ...
    em3: flags=6211<UP,BROADCAST,RUNNING,SLAVE,MULTICAST>  mtu 1500
    ...
    em4: flags=6211<UP,BROADCAST,RUNNING,SLAVE,MULTICAST>  mtu 1500
    ...
    lo: flags=73<UP,LOOPBACK,RUNNING>  mtu 65536
    ```
- `ethtool <网络接口>`: 查看网络接口的详细信息

    ```
    [root@cluster143 mcc]# ethtool em1
    Settings for em1:
    	Supported ports: [ TP ]
    	Supported link modes:   10baseT/Half 10baseT/Full
    	                        100baseT/Half 100baseT/Full
    	                        1000baseT/Half 1000baseT/Full  # 支持千兆半双工，全双工模式
    	Supported pause frame use: No
    	Supports auto-negotiation: Yes         # 支持自适应模式，一般都支持
    	Advertised link modes:  10baseT/Half 10baseT/Full
    	                        100baseT/Half 100baseT/Full
    	                        1000baseT/Half 1000baseT/Full
    	Advertised pause frame use: Symmetric
    	Advertised auto-negotiation: Yes       # 默认使用自适应模式
    	Link partner advertised link modes:  10baseT/Half 10baseT/Full
    	                                     100baseT/Half 100baseT/Full
    	                                     1000baseT/Full
    	Link partner advertised pause frame use: No
    	Link partner advertised auto-negotiation: Yes
    	Speed: 1000Mb/s            # 现在网卡的速度是1000Mb
    	Duplex: Full               # 全双工
    	Port: Twisted Pair
    	PHYAD: 1
    	Transceiver: internal
    	Auto-negotiation: on
    	MDI-X: off
    	Supports Wake-on: g
    	Wake-on: d
    	Current message level: 0x000000ff (255)
    			       drv probe link timer ifdown ifup rx_err tx_err
    	Link detected: yes         # 表示有网线连接，和路由是通的

    ```

## 操作系统
- `uname -a`: 查询系统内核

    ```
    [root@cluster143 mcc]# uname -a
    Linux cluster143 3.10.0-123.el7.x86_64 #1 SMP Mon Jun 30 12:09:22 UTC 2014 x86_64 x86_64 x86_64 GNU/Linux
    ```
- `ls /etc | grep release`: 查看发行文件

    ```
    [root@cluster143 mcc]# ls /etc|grep release
    centos-release
    os-release
    redhat-release
    system-release
    system-release-cpe
    ```
    查看内容

    ```
    [root@cluster143 mcc]# cat /etc/centos-release
    CentOS Linux release 7.0.1406 (Core)
    ```
    所以当前系统为`CentOS 7.0.1406`, 内核版本为`3.10.0-123.el7.x86_64`
    
## 其他

```bash
# 硬件
dmidecode |grep -A16 "System Information$"      # 查看主板型号
dmidecode |grep -A16 "Memory Device$"           # 查看内存型号

# 系统
hostname               # 查看计算机名
lspci -tv              # 列出所有PCI设备
lsusb -tv              # 列出所有USB设备
lsmod                  # 列出加载的内核模块
env                    # 查看环境变量

# 服务
chkconfig --list       # 列出所有系统服务

# 用户
w                      # 查看活动用户
id <用户名>             # 查看指定用户信息
last                   # 查看用户登录日志
cut -d: -f1 /etc/passwd   # 查看系统所有用户
cut -d: -f1 /etc/group    # 查看系统所有组
crontab -l             # 查看当前用户的计划任务

# 进程
ps -ef                 # 查看所有进程
top                    # 实时显示进程状态

# 网络
route -n               # 查看路由表
iptables -L            # 查看防火墙设置
netstat -lntp          # 查看所有监听端口
netstat -antp          # 查看所有已经建立的连接
netstat -s             # 查看网络统计信息

```

-----
### 参考
- [Linux下区分物理CPU、逻辑CPU和CPU核数](http://blog.csdn.net/dba_waterbin/article/details/8644626)
- [Linux查看CPU信息、机器型号等硬件信息](http://www.51testing.com/html/38/225738-210333.html)
- [Linux 查看系统硬件信息(实例详解)](http://www.cnblogs.com/ggjucheng/archive/2013/01/14/2859613.html)
