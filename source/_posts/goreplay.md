---
title: goreplay - http流量录制
tags:
  - linux
  - performance
categories:
  - linux
date: 2021-12-14 11:14:29
---

使用go语言开发的goreplay可以对运行中的服务端口进行流量录制, 并进行回放. 目前支持http协议请求

## 安装

1. 安装golang环境

```bash
# 解压安装
tar -C /usr/local -xzf go1.17.5.linux-amd64.tar.gz

# 配置到系统路径
vim /etc/profile
export PATH=$PATH:/usr/local/go/bin
source /etc/profile

# 验证
go version
```



2. 安装goreplay

```bash
# 解压安装
tar xvf gor_1.3.3_x64.tar.gz

# 复制到系统路径下(解压后就一个可执行文件 gor)
mv ./gor /usr/local/bin/

sudo ./gor --input-raw :8000 --output-stdout
```



## 使用

- 拦截请求并输出到控制台, 用于实时监控, 多个请求之间使用 **`\n🐵🙈🙉\n`** 进行分割

  ```bash
  gor --input-raw :8880 --output-stdout
  ```

- 录制请求到文件, 用于后续重放

  ```bash
  sudo ./gor --input-raw :8000 --output-file=requests.gor
  ```

- 录制流量, 直接发送到服务器(端口相同有bug?)

  ```bash
  sudo ./gor --input-raw :8000 --output-http="http://localhost:8001"
  ```

- 发送到多个位置, 并且随机分配请求

  ```bash
  gor --input-raw :80 --output-http "http://staging.com"  --output-http "http://dev.com" --split-output true
  ```

- 请求过滤

  ```bash
  gor --input-raw :8080 --output-http staging.com --http-allow-url /api
  gor --input-raw :8080 --output-http staging.com --http-disallow-url /api
  gor --input-raw :8080 --output-http staging.com --http-allow-header api-version:^1\.0\d
  gor --input-raw :80 --output-http "http://staging.server" \
      --http-allow-method GET \
      --http-allow-method OPTIONS
  ```

- 从文件重播流量

  ```
  ./gor --input-file requests.gor --output-http="http://localhost:8001"
  ```

#### 其他参数

- 捕获响应 `--output-http-track-response`

- http转发超时设置, 默认5s `--output-http-timeout 30s`

- 修改http响应体大小限制, 默认200k `--output-http-response-buffer 4096`

- 文件分块存储  `--output-file %Y%m%d.log`  支持`%Y` `%m` `%d` `%H` `%M` `%S`   

  ```
  20140608_0.log
  20140608_1.log
  20140609_0.log
  20140609_1.log
  ```

- 多个子块文件合并  `--output-file-append`

  ```
  20140608.log
  20140609.log
  ```

- 只按文件大小进行分块, 默认32M, 队列256, 输出文件进行gz压缩

  ```
  gor --input-raw :80 --output-file %Y-%m-%d.gz --output-file-size-limit 256m --output-file-queue-limit 0
  ```

- 输入文件通配符 `--input-file logs-2016-05-*`

- 
