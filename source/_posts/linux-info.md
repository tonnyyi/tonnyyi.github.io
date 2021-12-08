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

### TCP连接数限制

1. 可用端口范围 

   创建TCP连接时, 需要占用本机的一个端口号作为源端口, 本机可用端口号范围查询方式:

   ```bash
   $ cat /proc/sys/net/ipv4/ip_local_port_range
   32768	60999
   
   # 修改方式
   # 1. 临时
   echo "15000 64000" > /proc/sys/net/ipv4/ip_local_port_range
   # 2. 永久
   vim /etc/sysctl.conf 
   # 添加一行  net.ipv4.ip_local_port_range = 32768 59000
   sysctl -p /etc/sysctl.conf 	#生效
   ```

   > 同一源端口可以对多个不同的(目标IP+端口)建立TCP连接, 所以一台主机最多建立65536个TCP连接的说法不正确

2. 文件描述符
   - **系统级** `cat /proc/sys/fs/file-max` 所有进程打开的文件描述符总和, Centos7默认是794168
   - **用户级** `cat /etc/security/limits.conf` 和 `ulimit -n` 每个用户创建的进程打开的文件描述符总和
   - **进程级** `cat /proc/sys/fs/nr_open` 单个进程可用的最大文件描述符数量
   
   #### 查看文件句柄使用统计

    ```bash
    # 系统级
    $ cat /proc/sys/fs/file-nr
    4950	0	26114240
    已分配 已分配未使用 最大数量
   
    # 进程级 看Max open files行
    $ cat /proc/进程ID/limits
    Limit                     Soft Limit           Hard Limit           Units
    Max cpu time              unlimited            unlimited            seconds
    Max file size             unlimited            unlimited            bytes
    Max data size             unlimited            unlimited            bytes
    Max stack size            8388608              unlimited            bytes
    Max core file size        0                    unlimited            bytes
    Max resident set          unlimited            unlimited            bytes
    Max processes             1028915              1028915              processes
    Max open files            5000                 5000                 files
    Max locked memory         65536                65536                bytes
    Max address space         unlimited            unlimited            bytes
    Max file locks            unlimited            unlimited            locks
    Max pending signals       1028915              1028915              signals
    Max msgqueue size         819200               819200               bytes
    Max nice priority         0                    0
    Max realtime priority     0                    0
    Max realtime timeout      unlimited            unlimited            us
   
    # 实际进程已使用描述符数量 
    $ ll /proc/进程ID/fd | wc -l 
    115
    ```

    #### 修改方式

    ```bash
    # 系统级
    # 1. 临时
    sysctl -w fs.file-max=26114247
    # 或者
    echo "26114247" > /proc/sys/fs/file-max
    # 2. 永久
    vim /etc/sysctl.conf
    # 添加如下一行
    fs.file-max=26114247
    # 生效
    sysctl -p
   
    # 用户级
    # 1. 临时 
    ulimit -HSn 102400
    # 2. 永久 重新登录即生效
    vim /etc/security/limits.conf
    # 添加如下两行
    * hard nofile 102400
    * soft nofile 102400
   
    # 进程级
    # 1. 临时
    sysctl -w fs.nr_open=1048576
    # 或者
    echo 1000000 > /proc/sys/fs/nr_open
    # 2. 永久
    vim /etc/sysctl.conf
    # 添加如下一行
    fs.nr_open=1048576
    # 生效
    sysctl -p
    ```

    > - `file-max` 限制不了 `limits.conf`
    >
    > - `ulimit -n` 是设置当前shell以及当前shell启动的进程能打开的最大文件数量, 默认1024, 但`limits.conf`优先级更高
    >
    > - 如果每个线程处理一个TCP连接, 当线程数太多时, 服务器可能由于忙于上下文切换而导致崩溃(这就是C10K问题), 此时可以用**多路IO复用**, 一个线程管理多个TCP连接

3. 内存溢出

   每个TCP连接都会用到缓冲区, 都是需要占用一定的内存

4. CPU资源

### 设备唯一性

- `/etc/machine-id`文件

  系统安装或首次启动后生成并一致保持不变

  > **docker容器中该文件与宿主机内容一致**

  ```bash
  $ cat /etc/machine-id
  036aad1b88ec4e80906d5d27ed9cef1d
  ```

- `/sys/class/dmi/id/product_uuid`文件

  > **docker容器中该文件与宿主机内容一致**

  ```bash
  $ cat /sys/class/dmi/id/product_uuid
  036aad1b-88ec-4e80-906d-5d27ed9cef1d
  ```

- `/etc/fstab`文件

  >**docker容器中没有该文件**

  ```bash
  $ cat /etc/fstab
  
  #
  # /etc/fstab
  # Created by anaconda on Wed Mar 17 09:09:33 2021
  #
  # Accessible filesystems, by reference, are maintained under '/dev/disk'
  # See man pages fstab(5), findfs(8), mount(8) and/or blkid(8) for more info
  #
  /dev/mapper/klas-root   /                       xfs     defaults        0 0
  UUID=018c9363-2d14-4daf-ba6a-d12b294cfd82 /boot                   ext4    defaults        1 2
  UUID=7497-92C5          /boot/efi               vfat    umask=0077,shortname=winnt 0 0
  /dev/mapper/klas-home   /home                   xfs     defaults        0 0
  /dev/mapper/klas-swap   swap                    swap    defaults        0 0
  
  # 查看boot分区的UUID
  $ grep UUID /etc/fstab | grep -v /boot/ | awk '{print $1}'
  UUID=018c9363-2d14-4daf-ba6a-d12b294cfd82
  ```

- `/sys/class/dmi/id/board_serial`文件

  该文件内容为主板序列号, 虚拟机内没有该文件

  ```bash
  $ cat /sys/class/dmi/id/board_serial
  MBJ705S21125A90
  ```

- `hostid`命令

  无法确保全局唯一性, 且可以通过命令行配置

  ```bash
  $ hostid
  1facab67
  ```

- 根据根目录`/`挂载的磁盘id

  ```bash
  $ sudo /sbin/blkid | grep "$(df -h / | sed -n 2p | cut -d" " -f1):" | grep -o "UUID=\"[^\"]*\" " | sed "s/UUID=\"//;s/\"//"
  ef4de747-8828-4e4e-97db-05d47a5e1e60
  c6e4c0f0-576e-46f0-953a-da00dfa060fb
  ```

  

-----
### 参考
- [Linux下区分物理CPU、逻辑CPU和CPU核数](http://blog.csdn.net/dba_waterbin/article/details/8644626)
- [Linux查看CPU信息、机器型号等硬件信息](http://www.51testing.com/html/38/225738-210333.html)
- [Linux 查看系统硬件信息(实例详解)](http://www.cnblogs.com/ggjucheng/archive/2013/01/14/2859613.html)
