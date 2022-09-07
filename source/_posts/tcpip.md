---
title: TCP/IP协议
tags:
  - null
categories:
  - null
date: 2020-12-10 10:25:32
---

## IPv4

<img src="https://tonnyblog.oss-cn-beijing.aliyuncs.com/img/20201210105402.png" alt="image-20201210105352347" style="zoom: 50%;" />

|      |          地址范围           |            私有地址             | 适用网络类型 |
| :--: | :-------------------------: | :-----------------------------: | :----------: |
| A类  |  1.0.0.1 - 126.255.255.254  |    10.0.0.0 - 10.255.255.255    | 大型规模网络 |
| B类  | 128.0.0.1 - 191.255.255.254 | 172.16.0.0 - 172.31.255.255.255 | 中型规模网络 |
| C类  | 192.0.0.1 - 223.255.255.254 |  192.168.0.0 - 192.168.255.255  | 小型规模网络 |
| D类  | 224.0.0.1 - 239.255.255.254 |                                 | 多路广播网络 |
| E类  | 240.0.0.1 - 255.255.255.255 |                                 |   保留地址   |



![图片](https://tonnyblog.oss-cn-beijing.aliyuncs.com/img/20210806164159)



# linux 网络

![图片](https://tonnyblog.oss-cn-beijing.aliyuncs.com/img/640.png)

#### RingBuffer监控与调优

```bash
# 查看网卡RingBuffer设置  最大允许4096, 实际设置为512
[root@172 server]# ethtool -g eno3
Ring parameters for eno3:
Pre-set maximums:
RX:             4096
RX Mini:        0
RX Jumbo:       0
TX:             4096
Current hardware settings:
RX:             512
RX Mini:        0
RX Jumbo:       0
TX:             512

# 查看由于RingBuffer装不下导致的丢包(tx_fifo_errors字段可能没有) 突然的大流量 或  应用层未及时处理
# 可以查看ifconfig的overruns指标增长
[root@172 server]# ethtool -S eno3
NIC statistics:
......
rx_fifo_errors: 0
tx_fifo_errors: 0

```

