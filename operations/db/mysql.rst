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

显示创建的SQL语句：

- ``SHOW CREATE DATABASE db_name``
- ``SHOW CREATE TABLE table_name``

服务器状态信息： ``SHOW STATUS``

显示授权用户： ``SHOW GRANTS``

显示服务器错误或警告信息： ``SHOW ERRORS/SHOW WARNINGS``

修改MySQL数据库默认编码为UTF-8：在MySQL配置文件my.cnf中找到mysqld部分，在这部分中添加一句 ``character_set_server=utf8`` ，重启MySQL服务即可。

Sharding
-----------

.. seealso:: `开源数据库 Sharding 技术 (Share Nothing) <http://dbanotes.net/database/database_sharding-2.html>`_
.. seealso:: `Google Code: shard-query <https://code.google.com/p/shard-query/>`_
.. seealso:: `An Unorthodox Approach To Database Design : The Coming Of The Shard <http://highscalability.com/unorthodox-approach-database-design-coming-shard>`_
.. seealso:: `Scaling Web Databases: Auto-Sharding with MySQL Cluster <https://blogs.oracle.com/MySQL/entry/scaling_web_databases_auto_sharding>`_

存储过程
------------

`MySQL 5.1参考手册 :: 20. 存储程序和函数 <http://dev.mysql.com/doc/refman/5.1/zh/stored-procedures.html>`_

`[PDF]Stored Procedures - MySQL <http://dev.mysql.com/tech-resources/articles/mysql-storedprocedures.pdf>`_

`MySQL Stored Procedure Tutorial <http://www.mysqltutorial.org/mysql-stored-procedure-tutorial.aspx>`_

计划任务
------------

`Using the Event Scheduler <http://dev.mysql.com/doc/refman/5.1/en/events.html>`_

`[PDF]MySQL 5.1 Event Scheduler <http://assets.en.oreilly.com/1/event/21/Using%20the%20Event%20Scheduler_%20The%20Friendly%20Behind-the-Scenes%20Helper%20Presentation.pdf>`_


资源
--------

- 《高性能MySQL》
- `MySQL Internals Manual <http://dev.mysql.com/doc/internals/en/>`_
