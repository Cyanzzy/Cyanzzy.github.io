---
title: MySQL高级篇-1-Linux下的安装和使用
date: 2023-08-07 21:06:28
tags: 
  - MySQL
categories: 
  - Technology
---

# 安装前检查

>  查看是否安装过 MySQL 

 如果你是用 rpm 安装, 检查一下 RPM PACKAGE： 

```bash
rpm -qa | grep -i mysql # -i 忽略大小写
```

 检查 mysql service： 

```bash
systemctl status mysqld.service
```

 如果存在 mysql-libs 的旧版本包，显示如下： 

![](https://cyan-images.oss-cn-shanghai.aliyuncs.com/images/03-mysql-20230507-24.jpg)

 如果不存在 mysql-lib 的版本，显示如下：

![](https://cyan-images.oss-cn-shanghai.aliyuncs.com/images/03-mysql-20230507-25.jpg) 

> MySQL 的卸载 

 关闭 mysql 服务 

```bash
systemctl stop mysqld.service
```

查看当前 mysql 安装状况 

```bash
rpm -qa | grep -i mysql
# 或
yum list installed | grep mysql
```

 卸载上述命令查询出的已安装程序 

```bash
yum remove mysql-xxx mysql-xxx mysql-xxx mysqk-xxxx
```

 务必卸载干净，反复执行 `rpm -qa | grep -i mysql`确认是否有卸载残留 

 删除 mysql 相关文件 

```bash
# 查找相关文件
find / -name mysql
# 删除上述命令查找出的相关文件
rm -rf xxx
```

删除 my.cnf 

```bash
rm -rf /etc/my.cnf
```

# MySQL 的 Linux 版安装

## 下载 MySQL

1.  下载地址 https://www.mysql.com  
2.   点击 MySQL Community Server 

3.   Linux 系统下安装软件的常用三种方式： 

   *  使用 rpm 命令安装扩展名为 ".rpm" 的软件包 
   *  从互联网获取 的 yum 源，直接使用 yum 命令安装。  
   *  针对 tar.gz 这样的压缩格式，要用 tar 命令来解压；如果是其它压缩格式，就使用其它命令。 

4. 本文选择 tar 方式，下载 `mysql-8.0.25-1.el7.x86_64.rpm-bundle`

5. 将下面文件上传服务器

   ![](https://cyan-images.oss-cn-shanghai.aliyuncs.com/images/03-mysql-20230507-26.jpg)

## 检查 MySQL 依赖 

 检查 /tmp 临时目录权限（必不可少） 

 由于 mysql 安装过程中，会通过 mysql 用户在 /tmp 目录下新建 tmp_db 文件，所以请给 /tmp 较大的权限。执行 

```bash
chmod -R 777 /tmp
```

 安装前，检查依赖 

```bash
rpm -qa|grep libaio
rpm -qa|grep net-tools
```

## CentOS 7 下 MySQL 安装过程 

 将安装程序拷贝到 /opt 目录下 

 在 mysql 的安装文件目录下执行：（**必须按照顺序执行**）  

```bash
rpm -ivh mysql-community-common-8.0.25-1.el7.x86_64.rpm
rpm -ivh mysql-community-client-plugins-8.0.25-1.el7.x86_64.rpm
rpm -ivh mysql-community-libs-8.0.25-1.el7.x86_64.rpm
rpm -ivh mysql-community-client-8.0.25-1.el7.x86_64.rpm
rpm -ivh mysql-community-server-8.0.25-1.el7.x86_64.rpm
```

*  注意: 如在检查工作时，没有检查 mysql 依赖环境在安装 mysql-community-server 会报错 
*  rpm 是 Redhat Package Manage 缩写，通过 RPM 的管理，用户可以把源代码包装成以 rpm 为扩展名的 文件形式，易于安装 

* -i , --install 安装软件包
*  -v , --verbose 提供更多的详细信息输出 
* -h , --hash 软件包安装的时候列出哈希标记 (和 -v 一起使用效果更好)，展示进度条 

![](https://cyan-images.oss-cn-shanghai.aliyuncs.com/images/03-mysql-20230507-27.jpg)

> 查看MySQL版本

 执行如下命令，如果成功表示安装 mysql 成功。类似 java -version 如果打出版本等信息 

```bash
mysql --version
#或
mysqladmin --version
```

 执行如下命令，查看是否安装成功。需要增加 -i 不用去区分大小写，否则搜索不到。 

```bash
rpm -qa|grep -i mysql
```

> 服务的初始化

 为了保证数据库目录与文件的所有者为 mysql 登录用户，如果你是以 root 身份运行 mysql 服务，需要执 行下面的命令初始化： 

```bash
mysqld --initialize --user=mysql
```

 说明： --initialize 选项默认以“安全”模式来初始化，则会为 root 用户生成一个密码并将该密码标记为过期 ，登录后你需要设置一个新的密码。生成的临时密码会往日志中记录一份。  

 查看密码，root@localhost: 后面就是初始化的密码 ： 

```bash
cat /var/log/mysqld.log
## root@localhost: t;I!kjq>t2/6
```

> 启动 MySQL，查看状态 

```bash
# 加不加.service后缀都可以
启动：systemctl start mysqld.service
关闭：systemctl stop mysqld.service
重启：systemctl restart mysqld.service
查看状态：systemctl status mysqld.service
```

 mysqld 这个可执行文件就代表着 MySQL 服务器程序，运行这个可执行文件就可以直接启动一个服务器进程。 

![](https://cyan-images.oss-cn-shanghai.aliyuncs.com/images/03-mysql-20230507-28.jpg)

 查看进程： 

```bash
ps -ef | grep -i mysql
```

> 查看 MySQL 服务是否自启动 

```bash
systemctl list-unit-files|grep mysqld.service
```

 如不是 enabled 可以运行如下命令设置自启动 

```bash
systemctl enable mysqld.service
```

 如果希望不进行自启动，运行如下命令设置 

```bash
systemctl disable mysqld.service
```

## MySQL 登录 

> 首次登录 

通过 `mysql -hlocalhost -P3306 -uroot -p` 进行登录，在 Enter password：录入初始化密码 

> 修改密码  

因为初始化密码默认是过期的，所以查看数据库会报错 

 修改密码： 

```sql
ALTER USER 'root'@'localhost' IDENTIFIED BY 'new_password';
```

 5.7 版本之后（不含 5.7），mysql 加入了全新的密码安全机制。设置新密码太简单会报错， 改为更复杂的密码规则之后，设置成功，可以正常使用数据库 

> 设置远程登录 

1.当前问题：用 SQLyog 或 Navicat 中配置远程连接 Mysql 数据库时遇到如下报错信息，这是由于 Mysql 配置了不支持远程连接引起的 

![](https://cyan-images.oss-cn-shanghai.aliyuncs.com/images/03-mysql-20230507-29.jpg)

2.确认网络 

* 在远程机器上使用 `ping ip地址` 保证网络畅通 

* 在远程机器上使用 telnet 命令保证端口号开放访问 （开启 telnet 命令：控制面板--程序和功能--启动或关闭 Windows 功能--Telnet 客户端）

  ```bash
  telnet ip地址 端口号
  ```

3.关闭防火墙或开放端口 

关闭防火墙

```bash
systemctl start firewalld.service
systemctl status firewalld.service
systemctl stop firewalld.service
# 设置开机启用防火墙
systemctl enable firewalld.service
# 设置开机禁用防火墙
systemctl disable firewalld.service
```

开放端口

```bash
# 查看开放的端口号
firewall-cmd --list-all

# 设置开放的端口号
firewall-cmd --add-service=http --permanent
firewall-cmd --add-port=3306/tcp --permanent

# 重启防火墙
firewall-cmd --reload
```

4.Linux 下修改配置 

 在 Linux 系统 MySQL 下测试： 

```sql
use mysql;
select Host,User from user;
```

![](https://cyan-images.oss-cn-shanghai.aliyuncs.com/images/03-mysql-20230507-30.jpg)

 可以看到 root 用户的当前主机配置信息为 localhost。 

5.修改 Host 为通配符 % 

Host 列指定了允许用户登录所使用的 IP，user=root Host=localhost，表示只能通过本机客户端去访问。

而 % 是个 通配符 ，如果 Host=192.168.1.%，那么就表示只要是 IP 地址前缀为 “192.168.1.” 的客户端都可以连接。如果 Host=% ，表示所有IP都有连接权限。 

**注意：在生产环境下不能为了省事将 host 设置为 %，这样做会存在安全问题，具体的设置可以根据生产环境的 IP 进行设置** 

```sql
update user set host = '%' where user ='root';
```

 Host 设置了 `%` 后便可以允许远程访问。  

 **Host 修改完成后记得执行 flush privileges 使配置立即生效：** 

```sql
flush privileges
```

6.测试 

* 如果是 MySQL 5.7 版本，接下来就可以使用 SQLyog 或者 Navicat 成功连接至 MySQL
* 如果是 MySQL 8 版本，连接时还会出现如下问题：  

![](https://cyan-images.oss-cn-shanghai.aliyuncs.com/images/03-mysql-20230507-31.jpg)

 配置新连接报错：错误号码 2058，分析是 mysql 密码加密方法变了。 

解决方法：Linux下 mysql -u root -p 登录你的 mysql 数据库，然后 执行这条SQL： 

```sql
ALTER USER 'root'@'%' IDENTIFIED WITH mysql_native_password BY '密码';
```

 然后在重新配置 SQLyog 的连接，则可连接成功 

# 字符集的相关操作

##  修改 MySQL 5.7 字符集 

在 MySQL 8.0 版本之前，默认字符集为 latin1 ，utf8 字符集指向的是 utf8mb3 。从 MySQL 8.0开始，数据库的默认编码将改为 utf8mb4 ，从而避免上述乱码的问题。  

### 修改步骤

>  查看默认使用的字符集 

```bash
show variables like 'character%';
# 或者
show variables like '%char%';
```

 MySQL 8.0 中执行：  

![](https://cyan-images.oss-cn-shanghai.aliyuncs.com/images/03-mysql-20230507-32.jpg)

 MySQL 5.7 中执行： 

![](https://cyan-images.oss-cn-shanghai.aliyuncs.com/images/03-mysql-20230507-33.jpg)

 MySQL 5.7 默认的客户端和服务器都用了 latin1 ，不支持中文，保存中文会报错 

>  修改字符集 

```bash
vim /etc/my.cnf
```

 在MySQL5.7或之前的版本中，在文件最后加上中文字符集配置： 

```text
character_set_server=utf8
```

> 重新启动MySQL服务 

```bash
systemctl restart mysqld
```

 **但是原库、原表的设定不会发生变化，参数修改只对新建的数据库生效** 

### 已有库&表字符集的变更 

 MySQL5.7版本中，以前创建的库，创建的表字符集还是latin1。 

 **修改已创建数据库的字符集**  

```sql
alter database dbtest1 character set 'utf8';
```

 **修改已创建数据表的字符集** 

```sql
alter table t_emp convert to character set 'utf8';
```

**注意：但是原有的数据如果是用非'utf8'编码的话，数据本身编码不会发生改变。已有数据需要导 出或删除，然后重新插入。** 

##  各级别的字符集 

 MySQL有4个级别的字符集和比较规则，分别是： 

* 服务器级别 
* 数据库级别 
* 表级别 
* 列级别 

```sql
show variables like 'character%';
```

![](https://cyan-images.oss-cn-shanghai.aliyuncs.com/images/03-mysql-20230507-34.jpg)

* character_set_server：服务器级别的字符集 
* character_set_database：当前数据库的字符集 
* character_set_client：服务器解码请求时使用的字符集 
* character_set_connection：服务器处理请求时会把请求字符串从character_set_client转为 character_set_connection 
* character_set_results：服务器向客户端返回数据时使用的字符集 

### 服务器级别

`character_set_server`：服务器级别的字符集 

 可以在启动服务器程序时通过启动选项或者在服务器程序运行过程中使用 SET 语句修改这两个变量的值 

 可以在配置文件中这样写， 当服务器启动的时候读取这个配置文件后这两个系统变量的值便修改 

```properties
[server]
character_set_server=gbk # 默认字符集
collation_server=gbk_chinese_ci #对应的默认的比较规则
```

###  数据库级别

`character_set_database` ：当前数据库的字符集 

在创建和修改数据库的时候可以指定该数据库的字符集和比较规则 

```sql
CREATE DATABASE 数据库名
	[[DEFAULT] CHARACTER SET 字符集名称]
	[[DEFAULT] COLLATE 比较规则名称];

ALTER DATABASE 数据库名
	[[DEFAULT] CHARACTER SET 字符集名称]
	[[DEFAULT] COLLATE 比较规则名称];
```

###  表级别 

 可以在创建和修改表的时候指定表的字符集和比较规则 

```sql
CREATE TABLE 表名 (列的信息)
	[[DEFAULT] CHARACTER SET 字符集名称]
	[COLLATE 比较规则名称]]

ALTER TABLE 表名
	[[DEFAULT] CHARACTER SET 字符集名称]
	[COLLATE 比较规则名称]
```

**如果创建和修改表的语句中没有指明字符集和比较规则，将使用该表所在数据库的字符集和比较规则作为该表的字符集和比较规则。**  

### 列级别 

对于存储字符串的列，同一个表中的不同的列也可以有不同的字符集和比较规则。我们在创建和修改列 定义的时候可以指定该列的字符集和比较规则 

```sql
CREATE TABLE 表名(
    列名 字符串类型 [CHARACTER SET 字符集名称] [COLLATE 比较规则名称],
    其他列...
);

ALTER TABLE 表名 MODIFY 列名 字符串类型 [CHARACTER SET 字符集名称] [COLLATE 比较规则名称];
```

 **对于某个列来说，如果在创建和修改的语句中没有指明字符集和比较规则，将使用该列所在表的字符集 和比较规则作为该列的字符集和比较规则。** 

>  在转换列的字符集时需要注意，如果转换前列中存储的数据不能用转换后的字符集进行表示会发生 错误。比方说原先列使用的字符集是utf8，列中存储了一些汉字，现在把列的字符集转换为ascii的 话就会出错，因为ascii字符集并不能表示汉字字符。 

### 小结

* 如果 创建或修改列 时没有显式的指定字符集和比较规则，则该列 默认用表的 字符集和比较规则 
* 如果 创建表时 没有显式的指定字符集和比较规则，则该表 默认用数据库的 字符集和比较规则 
* 如果 创建数据库时 没有显式的指定字符集和比较规则，则该数据库 默认用服务器的 字符集和比较规 则  

##  字符集与比较规则（了解）

> utf8 与 utf8mb4 

 utf8 字符集表示一个字符需要使用1～4个字节，但是我们常用的一些字符使用1～3个字节就可以表示 了。而字符集表示一个字符所用的最大字节长度，在某些方面会影响系统的存储和性能，所以设计 MySQL的设计者偷偷的定义了两个概念：

*  utf8mb3 ：阉割过的 utf8 字符集，只使用1～3个字节表示字符。
*  utf8mb4 ：正宗的 utf8 字符集，使用1～4个字节表示字符。 

> 比较规则 

MySQL版本一共支持41种字符集，其中的 Default collation 列表示这种字符集中一种默认 的比较规则，里面包含着该比较规则主要作用于哪种语言，比如 `utf8_polish_ci` 表示以波兰语的规则 比较， `utf8_spanish_ci` 是以西班牙语的规则比较， `utf8_general_ci `是一种通用的比较规则。  

 后缀表示该比较规则是否区分语言中的重音、大小写。具体如下： 

| 后缀   | 英文释义             | 描述              |
| ------ | -------------------- | ----------------- |
| `_ai`  | `accent insensitive` | 不区分重音        |
| `_as`  | `accent sensitive`   | 区分重音          |
| `_ci`  | `case insensitive`   | 不区分大小写      |
| `_cs`  | `case sensitive`     | 区分大小写        |
| `_bin` | `binary`             | 以二进制方式比较S |

```sql
-- 查看GBK字符集的比较规则
SHOW COLLATION LIKE 'gbk%';
-- 查看UTF-8字符集的比较规则
SHOW COLLATION LIKE 'utf8%';


-- 查看服务器的字符集和比较规则
SHOW VARIABLES LIKE '%_server';
-- 查看数据库的字符集和比较规则
SHOW VARIABLES LIKE '%_database';
-- 查看具体数据库的字符集
SHOW CREATE DATABASE dbtest1;
-- 修改具体数据库的字符集
ALTER DATABASE dbtest1 DEFAULT CHARACTER SET 'utf8' COLLATE 'utf8_general_ci';


-- 查看表的字符集
show create table employees;
-- 查看表的比较规则
show table status from atguigudb like 'employees';
-- 修改表的字符集和比较规则
ALTER TABLE emp1 DEFAULT CHARACTER SET 'utf8' COLLATE 'utf8_general_ci';
```

##  请求到响应过程中字符集的变化

| 系统变量                   | 描述                                                         |
| -------------------------- | ------------------------------------------------------------ |
| `character_set_client`     | 服务器解码请求时使用的字符集                                 |
| `character_set_connection` | 服务器处理请求时会把请求字符串从 `character_set_client` 转为 `character_set_connection` |
| `character_set_results`    | 服务器向客户端返回数据时使用的字符集                         |

 这几个系统变量在我的计算机上的默认值如下（不同操作系统的默认值可能不同）： 

![](https://cyan-images.oss-cn-shanghai.aliyuncs.com/images/03-mysql-20230507-36.jpg)

 为了体现出字符集在请求处理过程中的变化，我们这里特意修改一个系统变量的值： 

```sql
mysql> set character_set_connection = gbk;
Query OK, 0 rows affected (0.00 sec)
```

 现在假设我们客户端发送的请求是下边这个字符串： 

```sql
SELECT * FROM t WHERE s = '我';
```

 分析字符 '我' 在这个过程中字符集的转换 

1.  客户端发送请求所使用的字符集 

    一般情况下客户端所使用的字符集和当前操作系统一致，不同操作系统使用的字符集可能不一 样 

   *  类 Unix 系统使用的是 utf8 
   *  Windows 使用的是 gbk 

    当客户端使用的是 utf8 字符集，字符 `我`在发送给服务器的请求中的字节形式就是： `0xE68891` 

   >  如果你使用的是可视化工具，比如navicat之类的，这些工具可能会使用自定义的字符集来编 码发送到服务器的字符串，而不采用操作系统默认的字符集（所以在学习的时候还是尽量用 命令行窗口）。 

2.   服务器接收到客户端发送来的请求其实是一串二进制的字节，它会认为这串字节采用的字符集是 `character_set_client` ，然后把这串字节转换为 `character_set_connection` 字符集编码的字符。  

    由于我的计算机上 `character_set_client` 的值是 utf8 ，首先会按照 utf8 字符集对字节串 `0xE68891 `进行解码，得到的字符串就是 `我` ，然后按照 `character_set_connection` 代表的字符集，也就是 gbk 进行编码，得到的结果就是字节串 `0xCED2` 

3.   因为表 t 的列 col 采用的是 gbk 字符集，与 `character_set_connection` 一致，所以直接到列 中找字节值为 `0xCED2` 的记录，最后找到了一条记录

   >  如果某个列使用的字符集和character_set_connection代表的字符集不一致的话，还需要进行 一次字符集转换。 

4.  上一步骤找到的记录中的 col 列其实是一个字节串 `0xCED2` ， col 列是采用 gbk 进行编码的，所 以首先会将这个字节串使用 gbk 进行解码，得到字符串 '我' ，然后再把这个字符串使用 `character_set_results` 代表的字符集，也就是 utf8 进行编码，得到了新的字节串： `0xE68891` ，然后发送给客户端。

5.  由于客户端是用的字符集是 utf8 ，所以可以顺利的将 `0xE68891` 解释成字符 `我` ，从而显示到显示器上

![](https://cyan-images.oss-cn-shanghai.aliyuncs.com/images/03-mysql-20230507-35.jpg)`

#  SQL大小写规范 

##  Windows和Linux平台区别 

 在 SQL 中，关键字和函数名是不用区分字母大小写的，比如 SELECT、WHERE、ORDER、GROUP BY 等关 键字，以及 ABS、MOD、ROUND、MAX 等函数名。 

不过在 SQL 中，你还是要确定大小写的规范，因为在 Linux 和 Windows 环境下，你可能会遇到不同的大 小写问题。`windows系统默认大小写不敏感` ，但是 `linux系统是大小写敏感` 。 

```bash
SHOW VARIABLES LIKE '%lower_case_table_names%'
```

lower_case_table_names参数值的设置：

* 默认为0，大小写敏感 。 (Linux下值为0)
* 设置1，大小写不敏感。创建的表，数据库都是以小写形式存放在磁盘上，对于sql语句都是转 换为小写对表和数据库进行查找（Windows下值为1） 
* 设置2，创建的表和数据库依据语句上格式存放，凡是查找都是转换为小写进行。 

>  MySQL在Linux下数据库名、表名、列名、别名大小写规则是这样的：
>
> 1. 数据库名、表名、表的别名、变量名是严格区分大小写的
> 2. 关键字、函数名称在 SQL 中不区分大小写
> 3. 列名（或字段名）与列的别名（或字段别名）在所有的情况下均是忽略大小写的
>
> **MySQL在Windows的环境下全部不区分大小写**  

##  Linux下大小写规则设置 

 当想设置为大小写不敏感时，要在 my.cnf 这个配置文件 [mysqld] 中加入 lower_case_table_names=1 ，然后重启服务器

* 但是要在重启数据库实例之前就需要将原来的数据库和表转换为小写，否则将找不到数据库名 

* 此参数适用于MySQL5.7。在MySQL 8下禁止在重新启动 MySQL 服务时将`lower_case_table_names`设置成不同于初始化 MySQL 服务时设置的 值。如果非要将MySQL8设置为大小写不敏感 

> 1. 停止MySQL服务
> 2. 删除数据目录，即删除 /var/lib/mysql 目录
> 3. 在MySQL配置文件（ /etc/my.cnf ）中添加 lower_case_table_names=1
> 4. 启动MySQL服务

##  SQL编写建议 

*  关键字和函数名称全部大写；  
*  数据库名、表名、表别名、字段名、字段别名等全部小写； 

*  SQL 语句必须以分号结尾。 

#  sql_mode的合理设置

## 宽松模式 vs 严格模式

> 宽松模式

如果设置的是宽松模式，那么我们在插入数据的时候，即便是给了一个错误的数据，也可能会被接受， 并且不报错。 

通过设置sql mode为宽松模式，来保证大多数sql符合标准的sql语法，这样应用在不同数据 库之间进行 迁移 时，则不需要对业务sql 进行较大的修改 

> 严格模式

在 生产等环境 中，必须采用的是严格模式，进而开发、测试环境 的数据库也必须要设置，这样在 开发测试阶段就可以发现问题 

##  模式查看和设置

>  查看当前的sql_mode 

```sql
select @@session.sql_mode
select @@global.sql_mode
-- 或者
show variables like 'sql_mode';
```

> 临时设置方式：设置当前窗口中设置sql_mode 

````sql
SET GLOBAL sql_mode = 'modes...'; -- 全局
SET SESSION sql_mode = 'modes...'; -- 当前会话
````

> 永久设置方式：在/etc/my.cnf中配置sql_mode  

 在my.cnf文件(windows系统是my.ini文件)，新增： 

```properties
[mysqld]
sql_mode=ONLY_FULL_GROUP_BY,STRICT_TRANS_TABLES,NO_ZERO_IN_DATE,NO_ZERO_DATE,ERROR_FOR
_DIVISION_BY_ZERO,NO_ENGINE_SUBSTITUTION
```

然后 重启MySQL 。 当然生产环境上是禁止重启MySQL服务的，所以采用临时设置方式 + 永久设置方式 来解决线上的问题
