---
title: MySQL备份与还原
tags:
  - MySQL
categories:
  - MySQL
date: 2022-08-12 09:26:25
---

## mysqldump命令

### 导出

只导出结构

```bash
mysqldump --opt -d 数据库名 -u root -p > xxx.sql
```

只导出数据

```bash
mysqldump -t 数据库名 -u root -p > xxx.sql
```

导出结构和数据

```bash
mysql
```



```
--add-drop-database
--add-drop-table
```



### 导入

```bash
source xx.sql
```

