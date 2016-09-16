## MySQL

### 基本使用

（在shell中）连接MySQL服务器： ``mysql -h host -u user -p``

（在MySQL命令行客户端中）显示所有数据库： ``SHOW DATABASES;`` ，使用某个数据库： ``USE databasename;`` ，显示当前数据库中所有数据表： ``SHOW TABLES;``

（在shell中）数据库备份： ``mysqldump -h host -u user -p dbname > /path/to/target_file.sql`` ，单独导出某个数据表： ``mysqldump -h host -u user -p dbname tablename > /path/to/target_file.sql`` 。

只导出数据表结构：**使用mysqldump时加`-d`选项**

（在shell中）恢复数据库： ``mysql -h host -u user -p dbname < /path/to/source_file.sql`` 。

删除数据库的SQL语句： ``DROP DATABASE dbname`` 。

为数据库添加字段的SQL语句： ``ALTER TABLE `serverinfo` ADD COLUMN processlist varchar(1024) DEFAULT NULL COMMENT '进程列表' AFTER `alias`;``

显示创建的SQL语句：

- ``SHOW CREATE DATABASE db_name``
- ``SHOW CREATE TABLE table_name``

服务器状态信息： ``SHOW STATUS``

显示授权用户： ``SHOW GRANTS``

显示服务器错误或警告信息： ``SHOW ERRORS/SHOW WARNINGS``

修改MySQL数据库默认编码为UTF-8：在MySQL配置文件my.cnf中找到mysqld部分，在这部分中添加一句 ``character_set_server=utf8`` ，重启MySQL服务即可。

MySQL用户创建与授权：

1. 以root用户登录MySQL
2. CREATE USER 'username'@'host' IDENTIFIED BY 'password';
3. GRANT privileges ON databasename.tablename TO 'username'@'host'; // 如：GRANT ALL ON test.* TO 'pig'@'localhost';

- 查看环境变量：`SHOW VARIABLES;`、`SHOW VARIABLES LIKE '%size%';`
- 修改帐号密码：`SET PASSWORD FOR 'user_name'@'where_from' = PASSWORD('password');`。不过貌似不同版本的MySQL修改帐号密码的方式不一样。
- 设置binlog的保持时长：`SET GLOBAL expire_logs_days = 3;`
- 当某个字段的值为null，需要将其更新为空字符串：`update cnvdvul set author="" WHERE author is null;`

#### INSERT INTO + SELECT FROM

```sql
INSERT INTO Evil0day (`app`,`version`) SELECT `app`, `version` FROM Fingerprint WHERE app="joomla" AND version >="3.2.0" AND version <="3.4.4" GROUP BY `version`;
```

```sql
INSERT IGNORE INTO `BaseIndex` (`host`, `realtime`, `ex_realtime`,  `history`, `ex_history`, `environment`, `ex_environment`, `attackrisk`, `ex_attackrisk`, `datestamp`) SELECT `host`, `realtime`, `ex_realtime`,  `history`, `ex_history`, `environment`, `ex_environment`, `attackrisk`, `ex_attackrisk`, 20160412 FROM `BaseIndex` WHERE `datestamp`=20160411;
```

#### 复杂SQL

来自：http://www.sohamkamani.com/blog/2016/07/07/a-beginners-guide-to-sql/

```sql
SELECT members.firstname || ' ' || members.lastname AS "Full Name"

FROM borrowings
INNER JOIN members
ON members.memberid=borrowings.memberid
INNER JOIN books
ON books.bookid=borrowings.bookid

WHERE borrowings.bookid IN (SELECT bookid FROM books WHERE stock>  (SELECT avg(stock) FROM books)  )

GROUP BY members.firstname, members.lastname;
```

```sql
SELECT title, bookid
FROM books
WHERE author IN (SELECT author
  FROM (SELECT author, sum(stock)
  FROM books
  GROUP BY author) AS results
  WHERE sum > 3);
```

```sql
SELECT
members.firstname AS "First Name",
members.lastname AS "Last Name",
count(*) AS "Number of books borrowed"
FROM borrowings
JOIN books ON borrowings.bookid=books.bookid
JOIN members ON members.memberid=borrowings.memberid
WHERE books.author='Dan Brown'
GROUP BY members.firstname, members.lastname;
```

### OUTFILE + INFILE

```sql
SELECT DISTINCT(name), "host" FROM `clean_vul` WHERE `vul_class`=28 AND name NOT LIKE '%/%' INTO OUTFILE '/home/work/app_names.csv' FIELDS TERMINATED BY ',' LINES TERMINATED BY '\n';

LOAD DATA INFILE '/home/work/app_names.csv' INTO TABLE app_name_mapper FIELDS TERMINATED BY ',' ENCLOSED BY '"' LINES TERMINATED BY '\n' (old_app_name, type);
```

### 添加索引

```
ALTER TABLE table_name
    ADD INDEX [index_name] [index_type] (index_col_name,...) [index_option];
```

如：
```
ALTER TABLE Users
    ADD INDEX (user_name);
```

### MySQL必知必会

- 显示数据库每一列的信息：`SHOW COLUMNS FROM table_name;` 或 `DESCRIBE table_name;`
- 显示广泛的服务器状态信息：`SHOW STATUS;`
- 显示授予用户的安全权限：`SHOW GRANTS;`
- `DESC`关键字只应用到直接位于其前面的列名。如果想在多个列上进行降序排序，必须对每个列指定DESC关键字。
- 为了检查某个范围的值，可使用 **BETWEEN** 操作符，该操作符需要两个值，即范围的开始值和结束值。√
- 空值检查，用`IS NULL`子句，如：`SELECT prod_name FROM products WHERE prod_price IS NULL;`
- WHERE可包含任意数目的AND和OR操作符，但是在处理OR操作符之前，优先处理AND操作符。可以使用圆括号来明确地分组相应的操作符。
- 通配符`_`，用途与`%`一样，但下划线只匹配单个字符而不是多个字符。
- `Concat`函数用于拼接字符串，如：`SELECT Concat(vend_name, ' (', vend_country, ')') FROM vendors ORDER BY vend_name;`
- 可使用 `RTrim()`、`LTrim()`、`Trim()`函数去除字符串两边的空格。

### 一次简单的数据库迁移过程

原因：我们测试环境的访问速度过慢，主要原因是DB服务器和Web服务器之间的网络太差，所以将DB服务也迁移到Web服务器上。

原MySQL版本：5.1.49，目标MySQL版本：5.6.19

步骤：

- 1.MySQL源码编译，见 http://dev.mysql.com/doc/refman/5.6/en/installing-source-distribution.html ，其中记得加系统账号mysql，设置数据目录权限，使用mysql_install_db进行数据库初始化

- 2.配置MySQL：
  
```
// 比如在mysqld部分添加以下两行
// 数据目录的路径
datadir=/var/lib/mysql
// 套接字文件路径
socket=/tmp/mysql.sock
```

- 3.修改用户密码，比如root的密码： ``mysqladmin -u root password 'new-password'``

- 4.数据导入导出：

    - ``mysqldump -h host -u user -p dbname > /path/to/target_file.sql``
    - ``CREATE DATABASE dbname DEFAULT CHARACTER SET utf8 DEFAULT COLLATE utf8_general_ci`` ，如果dbname属于MySQL的关键字，那么还得加反引号。
    - ``mysql -h host -u user -p dbname < /path/to/source_file.sql``

- 5.测试应用程序：由于新旧两个版本的MySQL版本差别较大，所以需要在一些细节上对应用程序做调整。

    - 如果数据表的某一字段设置为NOT NULL，但没有指定DEFAULT，在5.1.49版本上，如果应用程序在向该数据表插入一条记录且没有指定该字段的值，不会报错，但在5.6.19上会报错，所以更好的习惯是创建数据表/字段的时候，在指定NOT NULL的同时指定DEFAULT值
    - 在版本5.6.19上，若一个字段的类型为timestamp，即使是0值，也必须按照格式给出，如：“0000-00-00 00:00:00”，但在5.1.49上可以直接给“0”

## 错误处理

> Got fatal error 1236 from master when reading data from binary log: ‘binlog truncated in the middle of event; consider out of disk space on master; the first event ‘mysql-bin.000525’ at 175770780, the last event read from ‘/data/mysql/repl/mysql-bin.000525’ at 175770780, the last byte read from ‘/data/mysql/repl/mysql-bin.000525′ at 175771648.’


The solution would be to move the slave thread to the next available binary log and initialize slave thread with the first available position on binary log as below:

> CHANGE MASTER TO MASTER_LOG_FILE='mysql-bin.000526', MASTER_LOG_POS=4;

在主数据库上执行 `SHOW BINARY LOGS`，找到距离错误的bin log文件位置最近的一个日志位置，然后执行上面的这条命令（记得替换参数MASTER_LOG_FILE和MASTER_LOG_POS）。

------

当主从同步出现问题时，可以先按照下面的步骤处理：

1. `SHOW SLAVE STATUS`
2. `STOP SLAVE`
3. `SET GLOBAL sql_slave_skip_counter = N`
4. `START SLAVE`
5. `SHOW SLAVE STATUS`

### 主从同步

- `mysql> show slave hosts` -- 查看所有连接到Master的Slave信息
- `mysql> show master status` -- 查看Master状态信息
- `mysql> show slave status` -- 查看Slave状态信息
- `mysql> show binary logs` -- 查看所有二进制日志

### 慢日志

在my.cnf中添加:

```
# 启用慢日志
slow_query_log = 1
# 记录执行时间大于等于5秒的SQL
long_query_time = 5
# 慢日志文件的路径
slow_query_log_file = /home/work/logs/mysql/mysqld.slow.log
```

### Sharding

- [开源数据库 Sharding 技术 (Share Nothing)](http://dbanotes.net/database/database_sharding-2.html)
- [Google Code: shard-query](https://code.google.com/p/shard-query/)
- [An Unorthodox Approach To Database Design : The Coming Of The Shard](http://highscalability.com/unorthodox-approach-database-design-coming-shard)
- [Scaling Web Databases: Auto-Sharding with MySQL Cluster](https://blogs.oracle.com/MySQL/entry/scaling_web_databases_auto_sharding)

### 推荐阅读

- 《高性能MySQL》
- [MySQL Internals Manual](http://dev.mysql.com/doc/internals/en/)
- [MySQL索引原理及慢查询优化](http://tech.meituan.com/mysql-index.html)
- [Readings in Database Systems, 5th Edition](http://www.redbook.io/)
- [Readings in Databases](https://github.com/rxin/db-readings)
- [Top 10 performance tuning tips for relational databases](http://web.synametrics.com/top10performancetips.htm)
- [Is it safe to delete mysql-bin files?](http://dba.stackexchange.com/questions/41050/is-it-safe-to-delete-mysql-bin-files)


