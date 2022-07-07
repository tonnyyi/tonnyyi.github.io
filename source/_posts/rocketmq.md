---
title: rocketmq
tags:
  - rocketmq
  - java
  - mq
categories:
  - java
  - MQ
date: 2022-03-15 10:05:43
---

## 1. 单机部署

### 1.1 前置条件

- 服务安装JDK1.8+
- 下载部署包[4.9.3.zip](https://dlcdn.apache.org/rocketmq/4.9.3/rocketmq-all-4.9.3-bin-release.zip)

### 1.2 启动Namesrv

启动命令如下

```bash
nohup sh bin/mqnamesrv &
```

启动后, 查看日志

```bash
tail -f ~/logs/rocketmqlogs/namesrv.log
```

> 如果想修改日志文件位置, 可以修改`conf/logback_namesrv.xml`

如果目录下出现`hs_err_pid{进程ID}.log`文件说明启动有问题(比如内存不足), 查看内容会发现里面写了失败原因以及解决方法.

```bash
cat hs_err_pid{进程ID}.log

#
# There is insufficient memory for the Java Runtime Environment to continue.
# Native memory allocation (mmap) failed to map 1402470400 bytes for committing reserved memory.
# Possible reasons:
#   The system is out of physical RAM or swap space
#   In 32 bit mode, the process size limit was hit
# Possible solutions:
#   Reduce memory load on the system
#   Increase physical memory or swap space
#   Check if swap backing store is full
#   Use 64 bit Java on a 64 bit OS
#   Decrease Java heap size (-Xmx/-Xms)
#   Decrease number of Java threads
#   Decrease Java thread stack sizes (-Xss)
#   Set larger code cache with -XX:ReservedCodeCacheSize=
# This output file may be truncated or incomplete.
#
#  Out of Memory Error (os_linux.cpp:2627), pid=1392, tid=140458871453440
...
```

### 1.3 启动Broker

在 `conf` 目录下，RocketMQ 提供了多种 Broker 的配置文件：

- `broker.conf` ：单主，异步刷盘。
- `2m/` ：双主，异步刷盘。
- `2m-2s-async/` ：两主两从，异步复制，异步刷盘。建议方式：[搭建文档]。([https://www.cnblogs.com/powerx/p/13345054.html](https://links.jianshu.com/go?to=https%3A%2F%2Fwww.cnblogs.com%2Fpowerx%2Fp%2F13345054.html))
- `2m-2s-sync/` ：两主两从，同步复制，异步刷盘。
- `dledger/` ：[Dledger 集群](https://links.jianshu.com/go?to=https%3A%2F%2Fgithub.com%2Fapache%2Frocketmq%2Fblob%2Fmaster%2Fdocs%2Fcn%2Fdledger%2Fdeploy_guide.md)，至少三节点。

这里只启动一个Broker服务

```bash
nohup sh bin/mqbroker -c conf/broker.conf -n localhost:9876 &
```

查询如下日志, 表示启动成功

```log
2022-03-15 11:05:28 INFO brokerOutApi_thread_1 - register broker[0]to name server localhost:9876 OK
2022-03-15 11:05:28 INFO main - The broker[broker-a, 127.0.0.1:10911] boot success. serializeType=JSON and name server is localhost:9876
2022-03-15 11:05:37 INFO BrokerControllerScheduledThread1 - dispatch behind commit log 0 bytes
```

> - `-c`  指定Broker配置
> - `-n`  指定Namesrv地址

**127.0.0.1:10911**这个broker地址后面会用到

### 1.4 测试

#### 1.4.1 测试发送消息

```bash
# 设置Namesrv地址
export NAMESRV_ADDR=localhost:9876

# 发送测试消息
sh bin/tools.sh org.apache.rocketmq.example.quickstart.Producer
```

如果发送成功, 会看到大量的发送日志

```
SendResult [sendStatus=SEND_OK, msgId=7F000001064F070DEA4E4A7FACC70000, offsetMsgId=7F00000100002A9F000000000001E08A, messageQueue=MessageQueue [topic=TopicTest, brokerName=broker-a, queueId=2], queueOffset=0]
....
```

如果状态为**sendStatus=SEND_OK**, 则说明消息都发送成功了

#### 1.4.2 测试消费消息

```bash
# 设置Namesrv地址
export NAMESRV_ADDR=localhost:9876

# 消费测试消息
sh bin/tools.sh org.apache.rocketmq.example.quickstart.Consumer
```

如果消费成功, 会看到大量日志

```
...
ConsumeMessageThread_please_rename_unique_group_name_4_9 Receive New Messages: [MessageExt [brokerName=broker-a, queueId=1, storeSize=192, queueOffset=236, sysFlag=0, bornTimestamp=1647313882570, bornHost=/127.0.0.1:28148, storeTimestamp=1647313882571, storeHost=/127.0.0.1:10911, msgId=7F00000100002A9F000000000004A65C, commitLogOffset=304732, bodyCRC=1948249169, reconsumeTimes=0, preparedTransactionOffset=0, toString()=Message{topic='TopicTest', flag=0, properties={MIN_OFFSET=0, MAX_OFFSET=250, CONSUME_START_TIME=1647314037931, UNIQ_KEY=7F000001064F070DEA4E4A7FB1CA03B3, CLUSTER=DefaultCluster, TAGS=TagA}, body=[72, 101, 108, 108, 111, 32, 82, 111, 99, 107, 101, 116, 77, 81, 32, 57, 52, 55], transactionId='null'}]]
```

等到控制台不在输出时, 按`Ctrl+c`退出

### 1.5 关闭服务器

```bash
# 关闭broker
sh bin/mqshutdown broker

# 关闭namesrv
sh bin/mqshutdown namesrv
```

**关闭顺序要先关闭broker, 再关闭namesrv**

关闭成功后会显示如下日志

```
[root@localhost rocketmq-4.9.3]# sh bin/mqshutdown broker
The mqbroker(66635) is running...
Send shutdown request to mqbroker(66635) OK
[root@localhost rocketmq-4.9.3]# sh bin/mqshutdown namesrv
The mqnamesrv(63900) is running...
Send shutdown request to mqnamesrv(63900) OK
[2]+  退出 143              nohup sh bin/mqbroker -c conf/broker.conf -n localhost:9876
```



### 1.6 常用命令

1. 查看集群状态: `bin/mqadmin clusterList -n 127.0.0.1:9876`
2. 查看broker状态: `bin/mqadmin brokerStatus -n 127.0.0.1:9876 -b 172.31.169.140:10911`
3. 查看topic列表: `bin/mqadmin topicList -n 127.0.0.1:9876`
4. 查看topic状态: `bin/mqadmin topicStatus -n 127.0.0.1:9876 -t TopicTest`  TopicTest换成自己的主题名
5. 查看topic路由: `bin/mqadmin topicStatus -n 127.0.0.1:9876 -t TopicTest`  TopicTest换成自己的主题名

## 2. 集群部署

- 如果对高性能有比较强的诉求，使用两主两从，异步复制，异步刷盘。
- 如果对可靠性有比较强的诉求，建议使用Dledger 集群，至少三节点。



## 3. Rocket Dashboard 控制台

在 RocketMQ 拓展项目([rocketmq-externals](https://links.jianshu.com/go?to=https%3A%2F%2Fgithub.com%2Fapache%2Frocketmq-externals)) 中，包含了 [RocketMQ Console](https://links.jianshu.com/go?to=https%3A%2F%2Fgithub.com%2Fapache%2Frocketmq-externals%2Ftree%2Fmaster%2Frocketmq-console) 项目，是 RocketMQ 的图形化管理控制台，提供 Broker 集群信息查看，Topic 管理，Producer、Consumer 信息展示，消息查询等等常用功能。现在改名为**RocketMQ Dashboard**。转移后的仓库为：https://github.com/apache/rocketmq-dashboard

虽然说，我们也可以使用 RocketMQ 提供的 [CLI Admin Tool](https://links.jianshu.com/go?to=http%3A%2F%2Frocketmq.apache.org%2Fdocs%2Fcli-admin-tool%2F) 工具，实现上述的查询与管理的功能，但是命令行的方式对操作人员的要求稍高一些。当然，在 RocketMQ Console 无法满足我们更精细化的管理的需求的时候，我们还是会使用 CLI Admin Tool 工具。

### 安装

```bash
git clone https://github.com/apache/rocketmq-dashboard.git

# 源码运行
mvn spring-boot:run

# 构建后运行
mvn clean package -Dmaven.test.skip=true
java -jar target/rocketmq-dashboard-1.0.1-SNAPSHOT.jar
```

- 如果你使用rocketmq < 3.5.8，请在启动rocketmq-dashboard时添加-Dcom.rocketmq.sendMessageWithVIPChannel=false（或者你可以在ops页面更改）
- 更改 `src/main/resources/application.yml`, 修改默认端口, Namesrv配置, 或者您可以在 ops 页面中更改Namesrv

### 启动

```bash
nohup java -jar rocketmq-dashboard-1.0.1-SNAPSHOT.jar &
```



> **参考**
>
> 1. [RocketMQ 安装和启动](https://www.jianshu.com/p/3733903440d1)

