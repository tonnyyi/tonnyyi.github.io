---
title: wireshark
tags:
  - tcp
  - http
  - wireshark
categories:
  - network
date: 2022-07-04 09:57:29
---



## 抓包

使用`tcpdump`命令抓包

```bash
tcpdump tcp -i eth0 -s 0 -nn -vvv and port 8080 -w tcpdump.pcap

# 使用wireshark实时抓包分析
ssh root@172.31.47.13 tcpdump -i enp95s0f0 tcp -s 0 -nn -w - 'port 8085' | /Applications/Wireshark.app/Contents/MacOS/Wireshark -k -i -
```

## 分析

### 封包详细信息

![image-20220704110641431](https://tonnyblog.oss-cn-beijing.aliyuncs.com/img/image-20220704110641431.png)

**MTU** 指的是在三层（L3）上传输的最大数据报单元, 而 MTU 的大小一般由数据链路层（L2）设备决定. 比如使用最广泛的以太网(Ethernet, IEEE 802.3)的帧大小是 1518 字节, 根据 Ethernet Frame 的定义, L2 Frame 由 14 字节 Header 和 4 字节 Trailer 组成, 所以 L3 层(也就是 IP 层)最多只能填充 1500 字节大小, 这就是 MTU 的由来。
　　MTU即物理接口（数据链路层）提供给其上层（通常是IP层）最大一次传输数据的大小；以普遍使用的以太网接口为例，缺省MTU=1500 Byte，这是以太网接口对IP层的约束，如果IP层有<=1500 byte 需要发送，只需要一个IP包就可以完成发送任务；如果IP层有> 1500 byte 数据需要发送，需要分片才能完成发送，这些分片有一个共同点，即IP Header ID相同。

**MSS** (Maximum Segment Size)是 TCP Layer (*L4*) 的属性, MSS 指的是 TCP payload 的长度. 当在 MTU 1500 的网络上传输时, MSS 为 1460 (即 1500 减去 20 字节 IP 头, 20 字节 TCP 头).
　　MSS是TCP提交给IP层最大分段大小，不包含TCP Header和 TCP Option，只包含TCP Payload ，MSS是TCP用来限制application层最大的发送字节数，是tcp能发送的分组的最大长度。MSS是系统默认的，就是系统TCP/IP栈所能允许的最大包。在建立连接时，这个值已经被确定了，这个值并不是客观的值，而是由tcp/ip的实现确定的。
　　如果底层物理接口MTU= 1500 byte，则 MSS = 1500- 20(IP Header) -20 (TCP Header) = 1460 byte，如果application 有2000 byte发送，需要两个segment才可以完成发送，第一个TCP segment = 1460，第二个TCP segment = 540。实际场景下，TCP包头会带有12字节的时间戳，于是单个TCP包实际传输的最大量就会缩减为1448字节，1448=1500-20字节（IP头）-32字节（20字节TCP头和12字节TCP时间戳）

**TCP Window Size**: 如果A发送给B window size = 8192，意思是：B最多可以连续发送8192 byte 给A，在A ACK这8192 byte之前。那A的这个8192byte 怎么来的呢？ 一般来说，8192byte就是A的接收缓区，A_Receive_Buffer= 8192，如果B不小心发送超过8192 byte，如果application没有及时取走，则超过8192 byte 数据可能会因为A_Receive_Buffer满而被丢弃，所以B会严格遵守A的 advertised window size，如果A通告的window = 0，则B一定不会发送数据。
　　Window Size 占两个byte，最大值为65535，看完下文你会发现它对对方发送速率影响很大。如果window size 是高速网络带宽的瓶颈，可以采用TCP Option scaling window 这个选项。

### TCP包结构

![image-20220704113459669](https://tonnyblog.oss-cn-beijing.aliyuncs.com/img/image-20220704113459669.png)

MTU(Maximum Transmission Unit)，即最大传输单元，意思是在网络通信协议里规定最大传输的字节数量，通常是1500字节(不同网络大小不一样，可使用ifconfig查看)，最小是46。但是抓包软件抓出来会发现有些数据包长度会大于MTU，理论上当IP包大于MTU时，会对包进行分片。既然会分片，那么这个包大于MTU是几个意思？原因是这个分片不是发生在IP协议上，而是在TCP协议上，有个东西叫tso(tcp segment offload)，意思是如果网卡支持tso，操作系统发送大的tcp包时，不需要消耗CPU来计算分片，而是将整个包发送到网卡，由**网卡的NPU来进行分片处理。而抓包软件抓的仅仅是CPU处理后的信息**，也就是说在发送方抓的时候还没到网卡就被CPU抓了，而真正的分片是由后面的网卡才分片｛但是在接收端抓就会抓到已经分片的包了｝

```bash
# 查看网卡是否支持tso
ethtool --show-offload eth0
# ...
# tcp-segmentation-offload: on
# ...

# 禁用tso, 之后再抓包就不会大于MTU了
ethtool -K eth0 tso off
```

　　

#### TCP三次握手

选中1条http记录, 右键点击"Follow TCP Stream", 这样做的目的是为了得到与浏览器打开网站相关的数据包

![image-20220704122257609](https://tonnyblog.oss-cn-beijing.aliyuncs.com/img/image-20220704122257609.png)

![image-20220704122430026](https://tonnyblog.oss-cn-beijing.aliyuncs.com/img/image-20220704122430026.png)

![image-20220704122549337](https://tonnyblog.oss-cn-beijing.aliyuncs.com/img/image-20220704122549337.png)

### 过滤器

wireshark 提供了两种过滤器: 捕获过滤器 和 显示过滤器

#### 1. 捕获过滤器

在抓包之前设置好过滤条件, 然后只会抓取符合条件的数据包

![image-20220704100618506](https://tonnyblog.oss-cn-beijing.aliyuncs.com/img/image-20220704100618506.png)

语法示例

```
(host 10.4.1.12 or src net 10.6.0.0/16) and tcp dst portrange 200-10000 and dst net 10.0.0.0/8
```





#### 2. 显示过滤器

在已捕获的数据包中设置过滤条件, 只显示符合条件的数据包

![image-20220704101422123](https://tonnyblog.oss-cn-beijing.aliyuncs.com/img/image-20220704101422123.png)

> 这两种过滤语法是完全不同的, 因为捕捉网卡的其实是WinPcap(Windows)或libpcap(Linux), 所以要按它们的规则来配置, 而显示过滤器则就是Wireshark对数据进行过滤

### 显示过滤器语法

当使用关键字作为值时，需使用反斜杠“/”。

Comparison operators （比较运算符）:

| 英文写法： | C语言写法： | 含义：   |
| ---------- | ----------- | -------- |
| eq         | ==          | 等于     |
| ne         | !=          | 不等于   |
| gt         | >           | 大于     |
| lt         | <           | 小于     |
| ge         | >=          | 大于等于 |
| le         | <=          | 小于等于 |

Logical expressions（逻辑运算符）:

| 英文写法： | C语言写法： | 含义：   |
| ---------- | ----------- | -------- |
| and        | &&          | 逻辑与   |
| or         | \|\|        | 逻辑或   |
| xor        | ^^          | 逻辑异或 |
| not        | !           | 逻辑非   |

#### 1. 根据IP过滤

`src`: 源地址   `dst`: 目的地址

```bash
# ip过滤
ip.addr == 10.43.54.65
# 等同于
ip.src == 10.43.54.65 or ip.dst == 10.43.54.65

ip.addr != 10.43.54.65
# 等同于
ip.src != 10.43.54.65 or ip.dst != 10.43.54.65

!(ip.addr == 10.43.54.65)
# 等同于 这个更常用
!(ip.src == 10.43.54.65 or ip.dst == 10.43.54.65)

# 网段过滤
ip.src==192.168.0.0/16 and ip.dst==192.168.0.0/16

ip.addr in {10.0.0.5 .. 10.0.0.9 192.168.1.1..192.168.1.9}
```

#### 2. 针对协议过滤

```bash
# 获取某种协议的数据包时, 仅需输入协议名称即可
http

# 捕获多种协议
http or telnet

# 排查
!arp
# 或
not tcp
```

#### 3. 根据端口过滤

```bash
# 捕获单个端口
tcp.port == 25

# 捕获多端口
tcp.port in {80 443 8080}
tcp.port == 443 || (tcp.port >= 4430 && tcp.port <= 4434)

# 根据源/目的端口过滤
tcp.dstport == 23
tcp.srcport == 23
```

#### 4. 针对长度和内容过滤

```bash
# 针对长度过滤(这里的长度指的是数据段的长度)
tcp.length < 20

# 搜索按条件过滤udp的数据段payload（数字8是表示udp头部有8个字节，数据部分从第9个字节开始udp[8:]）
# (14是十六进制0x14)匹配payload第一个字节0x14的UDP数据包
udp[8]==14
# 可以udp[8:2]==1405，且只支持2个字节连续，三个以上须使用冒号：分隔表示十六进制。 (相当于 udp[8]==14 and udp[9]==05,1405是0x1405)
udp[8:2]==14:05 

# 搜索按条件过滤tcp的数据段payload（数字20是表示tcp头部有20个字节，数据部分从第21个字节开始tcp[20:]）
# 等同http matches "^GET [ -~]*HTTP/1.1\\x0d\\x0a"
tcp[20:] matches "^GET [ -~]*HTTP/1.1\\x0d\\x0a"

# 针对uri内容的过滤
http.request.uri contains "user"
```

#### 5. HTTP协议过滤

```bash
# 主机过滤
http.host == "www.baidu.com"
http contains "HTTP/1.1 200 OK" && http contains "application/json"

# 过滤请求或响应
http.request
http.response

# 请求过滤
http.request.version == "HTTP/1.1"
http.request.uri contains "login"
http.request.full_uri== "http://www.baidu.com/user/login"
# http uri结尾字符
http.request.uri matches ".jpg$"
# ①只需要http的包；②过滤掉jpg；③过滤掉png；④过滤掉zip
http.request and !((http.request.full_uri matches "http://.*\.jpg.*") or(http.request.full_uri matches "http://.*\.png.*") or(http.request.full_uri matches "http://.*\.zip.*"))
http.referer == "10.1.100.123"

# 响应过滤
http.response.code == 200
http.response.phrase == "OK"
# 过滤所有含有http头中含有server字段的数据包
http.server
http.server contains "nginx"

# 内容过滤
http.content_type == "application/json"
http.content_encoding == "gzip"
http.transfer_encoding == "chunked"
http.content_length == 279
http.content_length_header == "279"

# 其他常用条件
http.accept
http.location
http.cookie contains "token"
http.request.method == "post"
```

#### 5. 根据MAC过滤

```bash
# 过滤目标mac
eth.dst == A0:00:00:04:C5:84

# 过滤来源mac
eth.src eq A0:00:00:04:C5:84 

# 过滤来源MAC和目标MAC都等于A0:00:00:04:C5:84的
eth.addr eq A0:00:00:04:C5:84 

# 搜索过滤MAC地址前3个字节是0x001e4f的数据包。
eth.addr[0:3]==00:1e:4f 
```
