---
title: mysql-default-database
tags:
  - mysql
  - schema
categories:
  - mysql
date: 2021-11-11 14:31:48
---



# infomation_schema数据库

## CHARACTER_SETS表

该表提供查询MySQL支持的可用字符集有哪些, 该表是Memory引擎临时表

```bash
mysql> select * from character_sets;
+--------------------+----------------------+---------------------------------+--------+
| CHARACTER_SET_NAME | DEFAULT_COLLATE_NAME | DESCRIPTION                     | MAXLEN |
+--------------------+----------------------+---------------------------------+--------+
| big5               | big5_chinese_ci      | Big5 Traditional Chinese        |      2 |
| latin1             | latin1_swedish_ci    | cp1252 West European            |      1 |
| ascii              | ascii_general_ci     | US ASCII                        |      1 |
| gb2312             | gb2312_chinese_ci    | GB2312 Simplified Chinese       |      2 |
| gbk                | gbk_chinese_ci       | GBK Simplified Chinese          |      2 |
| utf8               | utf8_general_ci      | UTF-8 Unicode                   |      3 |
| latin7             | latin7_general_ci    | ISO 8859-13 Baltic              |      1 |
| utf8mb4            | utf8mb4_general_ci   | UTF-8 Unicode                   |      4 |
| gb18030            | gb18030_chinese_ci   | China National Standard GB18030 |      4 |
...
+--------------------+----------------------+---------------------------------+--------+
41 rows in set (0.00 sec)
```

### 字段含义

- **CHARACTER_SET_NAME**: 字符集名称
- **DEFAULT_COLLATE_NAME**: 字符集应对的默认校验规则
- **DESCRIPTION**: 字符集描述信息
- **MAXLEN**: 字符集单个字符占用的最大字节数



## COLLATION_CHARACTER_SET_APPLICABILITY表

该表提供查询MySQL中某中字符集所适用的校对规则, 该表是Memory引擎临时表

```bash
mysql> select * from collation_character_set_applicability;
+--------------------------+--------------------+
| COLLATION_NAME           | CHARACTER_SET_NAME |
+--------------------------+--------------------+
| gbk_chinese_ci           | gbk                |
| gbk_bin                  | gbk                |
| utf8_general_ci          | utf8               |
| utf8_bin                 | utf8               |
| utf8_unicode_ci          | utf8               |
...
| utf8_unicode_520_ci      | utf8               |
| utf8_vietnamese_ci       | utf8               |
| utf8mb4_general_ci       | utf8mb4            |
| utf8mb4_bin              | utf8mb4            |
| utf8mb4_unicode_ci       | utf8mb4            |
...
| utf8mb4_unicode_520_ci   | utf8mb4            |
| utf8mb4_vietnamese_ci    | utf8mb4            |
| gb18030_chinese_ci       | gb18030            |
| gb18030_bin              | gb18030            |
| gb18030_unicode_520_ci   | gb18030            |
+--------------------------+--------------------+
222 rows in set (0.00 sec)
```

### 字段含义

- **COLLATION_NAME**: 校对规则名称
- **CHARACTER_SET_NAME**: 校对规则对应的字符集名称



## COLLATIONS表

该表提供查询MySQL支持的可用校对规则有哪些

```bash
mysql> select * from COLLATIONS;
+--------------------------+--------------------+-----+------------+-------------+---------+
| COLLATION_NAME           | CHARACTER_SET_NAME | ID  | IS_DEFAULT | IS_COMPILED | SORTLEN |
+--------------------------+--------------------+-----+------------+-------------+---------+
...
| utf8_general_ci          | utf8               |  33 | Yes        | Yes         |       1 |
| utf8_bin                 | utf8               |  83 |            | Yes         |       1 |
| utf8_unicode_ci          | utf8               | 192 |            | Yes         |       8 |
...
| utf8mb4_general_ci       | utf8mb4            |  45 | Yes        | Yes         |       1 |
| utf8mb4_bin              | utf8mb4            |  46 |            | Yes         |       1 |
| utf8mb4_unicode_ci       | utf8mb4            | 224 |            | Yes         |       8 |
...
+--------------------------+--------------------+-----+------------+-------------+---------+
222 rows in set (0.00 sec)
```

### 字段含义

- **COLLACTION_NAME**: 校对规则名称
- **CHARACTER_SET_NAME**: 校对规则对应的字符集名称
- **ID**: 校对规则的ID号
- **IS_DEFAULT**: 是否是字符集默认的校对规则
- **IS_COMPILED**: 校对规则是否被编译进Server中, 不过不是Yes, 则该规则处于不可用状态
- **SORTLEN**: 最大排序字节长度, 与字符集对应的字符串在排序时所占用的内存大小有关



## COLUMN_PRIVILEGES表

该表提供查询关于列(字段), 表中的内容来自`mysql.column_priv`列权限表(需要针对一个表的列单独授权之后才会有内容)

### 字段含义

- GRANTEE：PRIVILEGE_TYPE 列值的权限对应的授予者(账户名)

- TABLE_SCHEMA：PRIVILEGE_TYPE 列值的权限关联的表对应的库名

- TABLE_NAME：PRIVILEGE_TYPE 列值的权限关联的表名

- COLUMN_NAME：PRIVILEGE_TYPE 列值的权限关联的字段名

- PRIVILEGE_TYPE：具体的列权限名称，注意：该字段值只显示一个权限名称，即，如果一个字段拥有多个可授予的列权限值，则在该表中会记录多行记录，每行PRIVILEGE_TYPE列值仅对应一个权限名称

- IS_GRANTABLE：如果GRANTEE列值表示的授予者还同时拥有grant option权限，则该列值为YES，否则为NO\

  

 该表中的信息还可以通过show语句方式查询(select和show方式虽然都能查询该表中的列权限信息，但是查询的结果展示方式有所不同)

```bash
# 语法
SHOW GRANTS;
SHOW GRANTS FOR CURRENT_USER;
SHOW GRANTS FOR CURRENT_USER();

# 示例
root@localhost : information_schema 09:39:10> show grants for 'xx'@'%';
+-------------------------------------------------------------------------------+
| Grants for xx@%                                                              |
+-------------------------------------------------------------------------------+
| GRANT USAGE ON *.* TO 'xx'@'%'                                                |
| GRANT SELECT (id), INSERT (id), UPDATE (id) ON `sbtest`.`sbtest1` TO 'xx'@'%' |
+-------------------------------------------------------------------------------+
```



## COLUMNS表

该表提供查询表中的列(字段)信息

```bash
mysql> select * from columns where table_name = 'test' limit 1 \G;
*************************** 1. row ***************************
           TABLE_CATALOG: def
            TABLE_SCHEMA: test
              TABLE_NAME: test
             COLUMN_NAME: id
        ORDINAL_POSITION: 1
          COLUMN_DEFAULT: NULL
             IS_NULLABLE: NO
               DATA_TYPE: int
CHARACTER_MAXIMUM_LENGTH: NULL
  CHARACTER_OCTET_LENGTH: NULL
       NUMERIC_PRECISION: 10
           NUMERIC_SCALE: 0
      DATETIME_PRECISION: NULL
      CHARACTER_SET_NAME: NULL
          COLLATION_NAME: NULL
             COLUMN_TYPE: int(11) unsigned
              COLUMN_KEY: PRI
                   EXTRA: auto_increment
              PRIVILEGES: select,insert,update,references
          COLUMN_COMMENT: 
   GENERATION_EXPRESSION: 
1 row in set (0.00 sec)
```

### 字段含义

- TABLE_SCHEMA：显示列对应的库名
- TABLE_NAME：显示列对应的表名
- COLUMN_NAME：显示列名称
- ORDINAL_POSITION：显示表列在表中的列创建顺序
- COLUMN_DEFAULT：显示表列默认值
- IS_NULLABLE：显示表列是否带有NULL属性
- DATA_TYPE：显示列的数据类型，不包含列的数据类型定义的额外属性
- CHARACTER_MAXIMUM_LENGTH：显示字符类型字段的定义长度
- CHARACTER_OCTET_LENGTH：显示字符类型字段的定义长度对应的字节数，如果是单字节字符集，则该字段值与CHARACTER_MAXIMUM_LENGTH字段值相同(多字节字符集除外)
- NUMERIC_PRECISION：显示数字类型字段的精度（定义长度）
- NUMERIC_SCALE：显示数字类型字段的标度（小数位数）
- DATETIME_PRECISION：显示时间类型字段的精度（5.6版本之后，datetime时间类型字段在存储引擎层存储时都当作int类型处理，但存储时会比timestamp多一个字节）
- CHARACTER_SET_NAME：显示表列的字符集，如果使用SHOW FULL COLUMNS语句查看，那么可以从结果集的Collation列中看到字符集类型，例如：Collation值为latin1_swedish_ci，则该字符集就是latin1
- COLLATION_NAME：显示列的校对规则
- COLUMN_TYPE：显示表列的定义类型，包含列数据类型定义的额外属性（在show columns语句的结果集中该字段信息显示在Type列），例如：varchar(32)，该字段为 "MySQL extension" 列
- COLUMN_KEY：如果字段是索引列，则这里会显示出索引的类型
- EXTRA：显示生成列的类型，有效值为：VIRTUAL GENERATED或VIRTUAL STORED，该字段为 "MySQL extension" 列
- PRIVILEGES：显示列的可授予权限列表，未列出的权限无法使用grant语句授予
- COLUMN_COMMENT：显示列的注释信息
- GENERATION_EXPRESSION：显示生成列的计算表达式，该字段为 "MySQL extension" 列



还可以通过`show columns`语句查询表的列信息

```bash
mysql> show columns from test.test;
+-------+------------------+------+-----+---------+----------------+
| Field | Type             | Null | Key | Default | Extra          |
+-------+------------------+------+-----+---------+----------------+
| id    | int(11) unsigned | NO   | PRI | NULL    | auto_increment |
| order | float            | YES  |     | NULL    |                |
+-------+------------------+------+-----+---------+----------------+
2 rows in set (0.00 sec)
```

查询某个表所有字段的可授予权限, 还可以使用`show full columns`语句

```bash
mysql> show full columns from test.test;
+-------+------------------+-----------+------+-----+---------+----------------+---------------------------------+---------+
| Field | Type             | Collation | Null | Key | Default | Extra          | Privileges                      | Comment |
+-------+------------------+-----------+------+-----+---------+----------------+---------------------------------+---------+
| id    | int(11) unsigned | NULL      | NO   | PRI | NULL    | auto_increment | select,insert,update,references |         |
| order | float            | NULL      | YES  |     | NULL    |                | select,insert,update,references |         |
+-------+------------------+-----------+------+-----+---------+----------------+---------------------------------+---------+
2 rows in set (0.00 sec)
```



## ENGINES表

该笔提供查询MySQL支持的引擎相关信息

```bash
mysql> select * from engines;
+--------------------+---------+----------------------------------------------------------------+--------------+------+------------+
| ENGINE             | SUPPORT | COMMENT                                                        | TRANSACTIONS | XA   | SAVEPOINTS |
+--------------------+---------+----------------------------------------------------------------+--------------+------+------------+
| InnoDB             | DEFAULT | Supports transactions, row-level locking, and foreign keys     | YES          | YES  | YES        |
| MRG_MYISAM         | YES     | Collection of identical MyISAM tables                          | NO           | NO   | NO         |
| MEMORY             | YES     | Hash based, stored in memory, useful for temporary tables      | NO           | NO   | NO         |
| BLACKHOLE          | YES     | /dev/null storage engine (anything you write to it disappears) | NO           | NO   | NO         |
| MyISAM             | YES     | MyISAM storage engine                                          | NO           | NO   | NO         |
| CSV                | YES     | CSV storage engine                                             | NO           | NO   | NO         |
| ARCHIVE            | YES     | Archive storage engine                                         | NO           | NO   | NO         |
| PERFORMANCE_SCHEMA | YES     | Performance Schema                                             | NO           | NO   | NO         |
| FEDERATED          | NO      | Federated MySQL storage engine                                 | NULL         | NULL | NULL       |
+--------------------+---------+----------------------------------------------------------------+--------------+------+------------+
9 rows in set (0.00 sec)
```

### 字段含义

- ENGINE：引擎名称
- SUPPORT：表示ENGINE字段显示的引擎在MySQL Server中是否支持
- COMMENT：表示ENGINE字段显示的引擎相关的解释信息，例如："Supports transactions, row-level locking, and foreign keys"，表示支持事务、支持行级锁，支持外键
- TRANSACTIONS：表示ENGINE字段显示的引擎是否支持事务
- XA：表示ENGINE字段显示的引擎是否支持XA事务
- SAVEPOINTS：表示ENGINE字段显示的引擎是否支持事务保存点

**还可以通过`show engines;`语句查询**



## EVENTS表

该表提供查询计划任务事件的相关信息

```bash
## 创建统计事件
DELIMITER $$
CREATE EVENT test_event
    ->    ON SCHEDULE
    ->      EVERY 1 DAY
    ->    COMMENT '每天统计sbtest1表中的最大自增值'
    ->    DO
    ->      BEGIN
    -> insert into test_table select max(id) from sbtest1;
    ->      END $$
Query OK, 0 rows affected (0.00 sec)

 
DELIMITER ;

# 然后在events表中查询事件信息
select * from information_schema.events\G;
*************************** 1. row ***************************
      EVENT_CATALOG: def
        EVENT_SCHEMA: sbtest
          EVENT_NAME: test_event
            DEFINER: root@%
          TIME_ZONE: +08:00
          EVENT_BODY: SQL
    EVENT_DEFINITION: BEGIN
insert into test_table select max(id) from sbtest1;
      END
          EVENT_TYPE: RECURRING
          EXECUTE_AT: NULL
      INTERVAL_VALUE: 1
      INTERVAL_FIELD: DAY
            SQL_MODE: ONLY_FULL_GROUP_BY,STRICT_TRANS_TABLES,NO_ZERO_IN_DATE,NO_ZERO_DATE,ERROR_FOR_DIVISION_BY_ZERO,NO_AUTO_CREATE_USER,NO_ENGINE_SUBSTITUTION
              STARTS: 2018-01-21 17:05:37
                ENDS: NULL
              STATUS: ENABLED
      ON_COMPLETION: NOT PRESERVE
            CREATED: 2018-01-21 17:05:37
        LAST_ALTERED: 2018-01-21 17:05:37
      LAST_EXECUTED: NULL
      EVENT_COMMENT: 每天统计sbtest1表中的最大自增值
          ORIGINATOR: 3306111
CHARACTER_SET_CLIENT: utf8
COLLATION_CONNECTION: utf8_general_ci
  DATABASE_COLLATION: utf8_bin
1 row in set (0.00 sec)
```

### 字段含义

- EVENT_CATALOG：此列的值始终为def
- EVENT_SCHEMA：此事件所属的数据库名称
- EVENT_NAME：事件的名称
- DEFINER：创建事件的账户名称
- TIME_ZONE：事件的时区，是用于调度事件的时区，且在事件执行时生效。默认值为SYSTEM，代表使用system_time_zone系统变量设置的时区
- EVENT_BODY：用于事件的DO子句中的语句的语言类型，在MySQL 5.7中，总是"SQL"。注意：不要将此列值与早期MySQL版本中存在的同名列（该列先更名为EVENT_DEFINITION列）列混淆
- EVENT_DEFINITION：构成事件的DO子句的SQL语句的文本。即被事件执行的SQL语句
- EVENT_TYPE：事件重复类型，一次（transient）或重复（RECURRING）
- EXECUTE_AT：对于一次性事件，该字段表示创建事件的CREATE EVENT语句中、或修改事件的最后一个ALTER EVENT语句的AT子句中指定的DATETIME值（例如，如果事件是使用"ON SCHEDULE AT CURRENT_TIMESTAMP +'1：6'DAY_HOUR"子句创建，且事件在2018-01-21 14:05:30创建的，则此列中显示的值为'2018-01-22 20:05:30'，表示这个一次性事件将在创建时间2018-01-21 14:05:30的基础上再过一天+6小时之后执行）。如果事件的计时由EVERY子句而不是AT子句确定（则表示该事件是一个重复事件），则此列的值为NULL。
- INTERVAL_VALUE：对于重复事件，此列包含事件的EVERY子句中的数字部分。但对于一次性事件，此列为NULL。
- INTERVAL_FIELD：对于重复事件，此列包含EVERY子句的单位部分，用于管理事件的时间。此列有效值可能包含“YEAR”，“QUARTER”，“DAY”等值。但对于一次性事件，此列为NULL。
- SQL_MODE：创建或更改事件时MySQL Server的SQL模式
- STARTS：对于其定义中包含STARTS子句的重复事件，此列包含相应的DATETIME值。与EXECUTE_AT列类似，此值可解析定义语句中所使用的任何表达式并计算出结果值存放在该列中。如果没有STARTS子句，则此列为NULL
- ENDS：对于其定义中包含ENDS子句的重复事件，此列包含相应的DATETIME值。与EXECUTE_AT列类似，此值可解析定义语句中所使用的任何表达式并计算出结果值存放在该列中。如果没有ENDS子句，则此列为NULL
- STATUS：该列包含三个有效值，ENABLED、DISABLED、SLAVESIDE_DISABLED

> \* SLAVESIDE_DISABLED：表示事件是通过主备复制中的binlog重放方式在从库上创建的，事件运行状态在从库上被关闭

- ON_COMPLETION：该列包含两个有效值，PRESVEVE、NOT PRESERVE
- CREATED：创建事件的日期和时间。是一个TIMESTAMP值
- LAST_ALTERED：上次修改事件的日期和时间。是一个TIMESTAMP值。如果该事件自创建以来从未修改，则此列与CREATED列值相同
- LAST_EXECUTED：事件上次执行的日期和时间。是一个 DATETIME值。如果事件从未执行，则此列值为NULL。LAST_EXECUTED表示事件是从什么时候开始的。因此，ENDS列的时间值总是大于LAST_EXECUTED
- EVENT_COMMENT：事件的注释文本信息，如果事件没有注释信息，则该字段为空串
- ORIGINATOR：创建事件的MySQL Server的server id，用于复制。默认值为0
- CHARACTER_SET_CLIENT：创建事件时的character_set_client系统变量的会话值
- COLLATION_CONNECTION：创建事件时的collation_connection系统变量的会话值
- DATABASE_COLLATION：与事件关联的数据库的排序规则
