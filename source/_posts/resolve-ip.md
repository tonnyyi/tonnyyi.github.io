---
title: my public ip
tags:
  - ip
categories:
  - fun
date: 2018-04-30 21:40:54
---

### 服务端
远程vps
开启端口
```bash
iptables -I INPUT -p udp --dport 1234 -j ACCEPT
iptables -I INPUT -p tcp --dport 1234 -j ACCEPT
```
<!-- more -->
```python
# !/usr/bin/env python3
# coding=utf-8

import sys
import socket

server = socket.socket(socket.AF_INET,socket.SOCK_STREAM)

host='45.77.244.133'
port=1234

server.bind((host,port))
print("start at %s:%s" % (host, port))

## 设置最大连接数,超过后排队
server.listen(5)

while True:
    s, addr = server.accept()
    d = s.recv(1024)
    print("connected from: %s, msg: %s" % (addr[0], d.decode('utf-8')))

    s.send(addr[0].encode("utf-8"))
    s.close()
```
运行
```bash
python3 ./server.py  > server.log 2>&1 &
```

### 客户端
```python
#!/usr/bin/env python
# encoding: utf-8

"""


    @author: mails100@163.com
    @time: 2017/12/27 15:06
"""

import socket

ip = '45.77.244.133'
port = 1234

with socket.socket(socket.AF_INET, socket.SOCK_STREAM) as s:
    s.connect((ip, port))
    s.send('[rb-ip]'.encode('utf-8'))

    d = s.recv(1024)
    if d:
        print("my public ip:", d.decode('utf-8'))
```

配置cron定时任务
1. `vi ~/.profile`, 添加`EDITOR=vi; export EDITOR`
2. 创建`root_cron`文件
    ```
    # get current public ip by requesting vps
    */1 * * * * python3 /home/pi/myIP.py >> /home/pi/ip.log 2>&1
    ```
3. crontab root_cron
4. 调整任务`crontab -e`, 删除任务`crontab -r`, 查看`crontab -l`
