---
title: 网络通信原理
tags:
  - network
categories:
  - network
date: 2022-09-07 17:52:52
---



## ARP(Address Resolution Protocol)

如果目的主机与发送主机在同一网段(ip+子网掩码), 但是发送主机的ARP表中并无目的主机对应的记录, 则发送主机会以目的主机IP地址为内容, 广播ARP请求以期获得目的主机的MAC地址. 交换机会向所有端口(除到达口)上的主机转发, 这就是`洪泛`(Flooding). 只有目的主机接收到此ARP请求后, 会将自己的MAC地址和IP地址装入ARP应答后回复给发送主机, 发送主机收到应答后, 从中提取目的主机的MAC地址并在ARP表中增加一条IP地址到MAC地址的记录. 之后再向目的主机发送数据时, 就可以将数据封装成帧, 通过二次设备(如交换机)转发到本网段内的目的主机, 完成通信.

```bash
$ ifconfig -a
enp94s0f0: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
        inet 172.31.47.14  netmask 255.255.255.0  broadcast 172.31.47.255
        inet6 fe80::2e0:edff:fe94:f5ae  prefixlen 64  scopeid 0x20<link>
        ether 00:e0:ed:94:f5:ae  txqueuelen 1000  (Ethernet)
        RX packets 1044337254  bytes 664915187036 (619.2 GiB)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 951156674  bytes 388614409789 (361.9 GiB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0

# 查看arp地址缓存表
$ arp -a
? (172.31.47.5) at b4:05:5d:1a:b5:4b [ether] on enp94s0f0
? (172.31.47.13) at 6c:92:bf:69:a8:fe [ether] on enp94s0f0
gateway (172.31.47.1) at 7c:1e:06:2c:7e:ab [ether] on enp94s0f0
? (172.31.47.8) at 00:e0:ed:94:f5:a2 [ether] on enp94s0f0
? (172.31.47.16) at b4:05:5d:1a:b9:c7 [ether] on enp94s0f0

```

