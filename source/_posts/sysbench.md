---
title: sysbench 性能基准测试
tags:
  - linux
  - performance
categories:
  - performance
date: 2021-08-09 16:20:37
---

## 一、 安装

1. 安装`epel`软件源(Centos7)

   ```bash
   rpm -Uvh http://mirror.centos.org/centos/7/extras/x86_64/Packages/epel-release-7-11.noarch.rpm
   ```

2. 安装`sysbench`

   ```bash
   yum install -y sysbench
   ```

>安装时如果如下错误
>
>** 发现 2 个已存在的 RPM 数据库问题， 'yum check' 输出如下：
>2:postfix-2.10.1-6.el7.x86_64 有缺少的需求 libmysqlclient.so.18()(64bit)
>2:postfix-2.10.1-6.el7.x86_64 有缺少的需求 libmysqlclient.so.18(libmysqlclient_18)(64bit)
>
>
>
>解决方法如下:
>
>```bash
>wget https://downloads.mysql.com/archives/get/p/23/file/mysql-community-libs-compat-5.7.34-1.el7.x86_64.rpm
>rpm -ivh mysql-community-libs-compat-5.7.34-1.el7.x86_64.rpm
>```



## 二、使用

### 基础知识

语法格式: `sysbench [通用选项]... --test=<测试名称> [测试选项]... 命令`

常用通用选项:

- `--thread=N` 线程数, 默认 1
- `--events=N` 执行的请求数, 默认 0
- `--time=N` 运行时长限制, 单位秒, 默认 10
- `--rate=N` 0:无限制, 默认 0
- `--report-interval=N` 报告输出间隔, 单位秒, 默认0
- ``
- ``
- ``

### 系统基准测试

#### CPU



#### 内存



#### 文件IO

1. prepare

   ```bash
   sysbench --test=fileio --file-num=100 --file-total-size=500M prepare
   ```

2. run

   ```bash
   sysbench --test=fileio --file-total-size=500M --file-test-mode=rndrw --max-time=180 --max-requests=1000000 --num-threads=32 --file-num=100 --file-extra-flags=direct --file-fsync-freq=0 --file-block-size=16384 run
   ```

   测试结果:

   ```
   File operations:
       reads/s:                      290.52
       writes/s:                     193.57
       fsyncs/s:                     17.75
   
   Throughput:
       read, MiB/s:                  4.54
       written, MiB/s:               3.02
   
   General statistics:
       total time:                          180.2397s
       total number of events:              87254
   
   Latency (ms):
            min:                                    0.16
            avg:                                   66.06
            max:                                 1779.62
            95th percentile:                      277.21
            sum:                              5764074.78
   
   Threads fairness:
       events (avg/stddev):           2726.6875/68.73
       execution time (avg/stddev):   180.1273/0.07
   ```

3. cleanup

   ```bash
   sysbench --test=fileio --file-num=100 --file-total-size=500M cleanup
   ```

   

#### 多线程





### mysql基准测试



