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


### Sharding

- [开源数据库 Sharding 技术 (Share Nothing)](http://dbanotes.net/database/database_sharding-2.html)
- [Google Code: shard-query](https://code.google.com/p/shard-query/)
- [An Unorthodox Approach To Database Design : The Coming Of The Shard](http://highscalability.com/unorthodox-approach-database-design-coming-shard)
- [Scaling Web Databases: Auto-Sharding with MySQL Cluster](https://blogs.oracle.com/MySQL/entry/scaling_web_databases_auto_sharding)


### 推荐阅读

- 《高性能MySQL》
- [MySQL Internals Manual](http://dev.mysql.com/doc/internals/en/)
- [MySQL索引原理及慢查询优化](http://tech.meituan.com/mysql-index.html)
