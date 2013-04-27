MySQL
==========

基本使用
----------

（在shell中）连接MySQL服务器： ``mysql -h host -u user -p``

（在MySQL命令行客户端中）显示所有数据库： ``SHOW DATABASES;`` ，使用某个数据库： ``USE databasename;`` ，显示当前数据库中所有数据表： ``SHOW TABLES;``

（在shell中）数据库备份： ``mysqldump -h host -u user -p dbname > /path/to/target_file.sql`` ，单独导出某个数据表： ``mysqldump -h host -u user -p dbname tablename > /path/to/target_file.sql`` 。

（在shell中）恢复数据库： ``mysql -h host -u user -p dbname < /path/to/source_file.sql`` 。

删除数据库的SQL语句： ``DROP DATABASE dbname`` 。

为数据库添加字段的SQL语句： ``ALTER TABLE `serverinfo` ADD COLUMN processlist varchar(1024) DEFAULT NULL COMMENT '进程列表' AFTER `alias`;``

资源
--------

- 《高性能MySQL》
