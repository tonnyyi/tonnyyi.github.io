---
title: MariaDB安装与配置
tags:
- Mysql
- MariaDB

categories:
- Database
- MariaDB

---

## 环境
- 硬件
- 操作系统: CentOS
- **MariaDB**: 10.1.14

## 安装
https://mariadb.com/kb/en/mariadb/
### 准备
#### 修改yum源
#### 添加yum仓库

## 配置
/usr/share/mysql/
`mysqld --help --verbose`: 展示编译时的添加的默认配置以及启动时读取的配置
`mysqld --no-defaults --verbose --help`: 只展示编译时添加的默认配置, 不展示启动时读取的配置
`mysqld --print-defaults`: 只展示启动时读取的配置, 不展示编译时添加的默认配置

https://mariadb.com/kb/en/mariadb/server-system-variables
https://mariadb.com/kb/en/mariadb/mysqld-configuration-files-and-groups/
https://mariadb.com/kb/en/mariadb/mysqld-options/
http://dev.mysql.com/doc/refman/5.6/en/option-files.html
### my.cnf
执行`mysql --help`, 在输出的内容中说明了配置文件的加载位置和顺序:

```
Default options are read from the following files in the given order:
/etc/my.cnf ~/.my.cnf
```


https://mariadb.com/kb/en/mariadb/server-system-variables/

[mysqld]
http://dev.mysql.com/doc/refman/5.6/en/mysqld-option-tables.html
https://mariadb.com/kb/en/mariadb/mysqld-options/
https://mariadb.com/kb/en/mariadb/full-list-of-mariadb-options-system-and-status-variables/

[galera]
https://mariadb.com/kb/en/mariadb/galera-cluster-system-variables/

[mariadb]
https://mariadb.com/kb/en/mariadb/full-list-of-mariadb-options-system-and-status-variables/

[mariadb-10.1]
https://mariadb.com/kb/en/mariadb/system-and-status-variables-added-by-major-release/

#### groups
mysqld server mysqld-10.1 mariadb mariadb-10.1 client-server galera

## 启动停止
https://mariadb.com/kb/en/mariadb/starting-and-stopping-mariadb-automatically/
mysqld --print-defaults

## 集群
### Galera
https://mariadb.com/kb/en/mariadb/galera-cluster-system-variables/

## 测试

## 问题
