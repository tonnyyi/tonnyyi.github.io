---
title: 为什么MySQL单表数据量不要超过2000w?
tags:
  - MySQL
  - SQL
  - 性能
categories:
  - MySQL
date: 2022-08-10 15:15:25
---



### 页的数据结构

每张表的数据都存储在一个`表名.ibd`的文件中, 也叫表空间;  在这个文件中, 数据被分成**16K**大小的数据页.

![图片](https://tonnyblog.oss-cn-beijing.aliyuncs.com/img/640.png)

> 除了页, 还有段、区、组等概念

页的结构

![图片](https://tonnyblog.oss-cn-beijing.aliyuncs.com/img/640-20220810151951030.png)

当插入数据时, 会从`Free Space`中划出一部分给`UserRecords`用来存储数据. 当`Free Space`被用完时, 就需要申请新的页了.

![图片](https://tonnyblog.oss-cn-beijing.aliyuncs.com/img/640-20220810152213850.png)

### 索引的数据结构

索引页与数据页的结构几乎相同, 但索引页中记录的是页(数据页/索引页)的最小主键id和页号, 并且在索引页中增加了层级信息

![图片](https://tonnyblog.oss-cn-beijing.aliyuncs.com/img/640-20220810160112260.png)

### 2000w的依据

![图片](https://tonnyblog.oss-cn-beijing.aliyuncs.com/img/640-20220810160246326.png)

比如要查找id为6的行数据, 则需要查找10 -> 30 -> 60共3个页, InnoDb采用的是聚簇索引, 也就是说叶子节点存储的是实际数据

**假设**

- 非叶子节点内指向其他页的数量为 x
- 叶子节点内能存储的数据行数为 y
- B+ 树的层数为 z

因此总行数 Total = x^(z - 1) * y

x是多大呢? 页的结构包含了:  File Header (38 byte)、Page Header (56 Byte)、Infimum + Supermum（26 byte）、File Trailer（8byte）, 再加上页目录，大概 1k 左右, 剩下的15K用于存储数据，在索引页中主要记录的是主键与页号，主键我们假设是 Bigint (8 byte), 而页号也是固定的（4Byte）, 那么索引页中的一条数据也就是 12byte; 所以 x=15*1024/12≈1280 行。

y是多大呢? 和索引页一样, 数据页能存放数据的空间也是15K, 假设一行数据1K, 那一页就能存下 15 条，Y≈15。

如果z=2, Total = (1280 ^ 1) * 15 = 19200

如果z=3, Total = (1280 ^ 2) * 15 = 24576000(约2.45kw)

一般 B+ 数的层级最多也就是 3 层

所以实际的表行数限制不一定非得是2000w, 跟每行数据的大小也有关系, 比如, 如果每行数据大小为5k, 则Total = (1280 ^ 2) * 3 = 4915200(约500w)
