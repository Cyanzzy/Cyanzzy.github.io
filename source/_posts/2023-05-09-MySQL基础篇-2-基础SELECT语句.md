---
title: MySQL基础篇-2- 基础SELECT语句
date: 2023-05-09 19:00:07
tags: 
  - MySQL
categories: 
  - Technology
---

# SQL 概述

> SQL 分类

| 分类             | 说明                                                         | 关键字                                             |
| ---------------- | ------------------------------------------------------------ | -------------------------------------------------- |
| DDL 数据定义语言 | 定义不同的数据库、表、视图、索引等数据库对象，还可以用来创建、删除、修改数据库和数据表的结构。 | CREATE 、 DROP 、 ALTER                            |
| DML 数据操作语言 | 用于添加、删除、更新和查询数据库记 录，并检查数据完整性      | INSERT 、 DELETE 、 UPDATE 、 SELECT               |
| DCL 数据控制语言 | 用于定义数据库、表、字段、用户的访问权限和安全级别           | GRANT 、 REVOKE 、 COMMIT 、 ROLLBACK 、 SAVEPOINT |

# SQL 语言规范

> 大小写规范

* MySQL 在 Windows 环境下是大小写不敏感的 
* MySQL 在 Linux 环境下是大小写敏感的
  * 数据库名、表名、表的别名、变量名是严格区分大小写的 
  * 关键字、函数名、列名(或字段名)、列的别名(字段的别名) 是忽略大小写的 

> 数据导入

```sql
-- 在命令行客户端登录mysql，使用source指令导入 
source d:\mysqldb.sql
```

# 基本的SELECT语句

> SELECT

```sql
SELECT col_name;
FROM t_name;
```

> 别名

```sql
SELECT last_name AS name, commission_pct comm
FROM employees;
```

> 去重

```sql
SELECT DISTINCT department_id
FROM employees;
```

> null

 所有运算符或列值遇到 null 值，运算的结果都为 null， 在 MySQL 里面， 空值不等于空字符串。一个空字符串的长度是 0，而一个空值的长度是空。而且，在 MySQL 里面，空值是占用空间的。 

> 着重号

 需要保证表中的字段、表名等没有和保留字、数据库系统或常用方法冲突。如果真的相同，请在 SQL 语句中使用一对 **`**（着重号）引起来。  



