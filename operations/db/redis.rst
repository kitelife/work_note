Redis
===========

self-contained documents of redis.conf
-----------------------------------------

By default Redis asynchronously dumps the dataset on disk. Tnis mode is good enough in many application, but an issue with the Redis process or a power outage may result into a few minutes of writes lost (depending on th configured save points).

The Append Only File is an alternative persistence mode that provides much better durability. For instance using the default data fsync policy Redis can lose just one second of writes in a dramatic event like a server power outage, or a single write if something wrong with the Redis process itself happens, but the operating system is still running correctly.

AOF and RDB persistence can be enabled at the same time without problems. If the AOF is enabled on startup Redis will load the AOF, that is the file with the better durability guarantees.

The fsync() call tells the Operating System to actually write data on disk instead to wait for more data in the output buffer. Some OS will really flush data on disk, some other OS will just try to do it ASAP.

官网资料学习
-----------------

**Introduction to Redis**

Redis is an advanced **key-value store** , and is often referred to as a data structure server since keys can contain strings, hashes, lists, sets and sorted sets.

You can run atomic operations on these types, like appending to a string; incrementing the value in a hash; pushing to a list; computing set intersection, union and difference; or getting the member with highest ranking in a sorted set.

In order to achieve its outstanding performance, Redis works with an in-memory dataset. Depending on your use case, you can persist it either by dumping the dataset to disk every once in a while, or by appending each command to a log.

Redis also supports trivial-to-setup master-slave replication, with very fast non-blocking first synchronization, auto-reconnection on net split and so forth.

Other features include Transactions, Pub/Sub, Lua scripting, Keys with a limited time-to-live, and configuration settings to make Redis behave like a cache.

------

**Data types**

*Strings*

Strings are the most basic kind of Redis value. Redis Strings are binary safe, this means that a Redis string can contain any kind of data, for instance a JPEG image or a serialized Ruby object.

A String value can be at **max 512 Megabytes** in length.

.. seealso:: `all the available string commands <http://redis.io/commands/#string>`_

*Lists*

Redis Lists are simply lists of strings, sorted by insertion order. It is possible to add elements to a Redis List pushing new elements on the head (on the left) or on the tail (on the right) of the list.

The LPUSH command inserts a new element on the head, while RPUSH inserts a new element on the tail. A new list is created when one of this operations is performed against an empty key. Similarly the key is removed from the key space if a list operation will empty the list.

The max length of a list is 232 - 1 elements (4294967295, more than 4 billion of elements per list).

The main features of Redis Lists from the point of view of time complexity are the support for constant time insertion and deletion of elements near the head and tail, even with many millions of inserted items. Accessing elements is very fast near the extremes of the list but is slow if you try accessing the middle of a very big list, as it is an O(N) operation.

.. seealso:: `all the available commands operating on lists <http://redis.io/commands#list>`_

*Sets*

Redis Sets are an unordered collection of Strings. It is possible to add, remove, and test for existence of members in O(1) (constant time regardless of the number of elements contained inside the Set).

.. seealso:: `The full list of Set commands <http://redis.io/commands#set>`_

*Hashes*

Redis Hashes are maps between string fields and string values, so they are the perfect data type to represent objects.

.. seealso:: `The full list of Hash commands <http://redis.io/commands#hash>`_

*Sorted sets*

Redis Sorted Sets are, similarly to Redis Sets, non repeating collections of Strings. The difference is that every member of a Sorted Set is associated with score, that is used in order to take the sorted set ordered, from the smallest to the greatest score. While members are unique, scores may be repeated.

With sorted sets you can add, remove, or update elements in a very fast way (in a time proportional to the logarithm of the number of elements). Since elements are taken in order and not ordered afterwards, you can also get ranges by score or by rank (position) in a very fast way. Accessing the middle of a sorted set is also very fast, so you can use Sorted Sets as a smart list of non repeating elements where you can quickly access everything you need: elements in order, fast existence test, fast access to elements in the middle!

.. seealso:: `The full list of Sorted Set commands <http://redis.io/commands#sorted_set>`_

------

**Redis Persistence**

Redis provides a different range of persistence options:

- The RDB persistence performs point-in-time snapshots of your dataset at specified intervals.
- The AOF persistence logs every write operation recevied by the server, that will be played again at server startup, reconstructing the original dataset. Commands are logged using the same format as the Redis protocol itself, in an append-only fashion. Redis is able to rewrite the log on background when it gets too big.
- If you wish, you can disable persistence at all, if you want your data to just exist as long as the server is running.
- It is possible to combine both AOF and RDB in the same instance.

The most important thing to understand is the different trade-offs between the RDB and AOF persistence.

*RDB advantages*

- RDB is a very compact single-file point-in-time representation of your Redis data. RDB files are perfect for backups.
- RDB is very good for disaster recovery, being a single compact file can be transfered to far data centers.
- RDB maximizes Redis performances since the only work the Redis parent process needs to do in order to persist is forking a child that will do all the rest. The parent instance will never perform disk I/O or alike.
- RDB allows faster restarts with big datasets compared to AOF.

*RDB disadvantages*

- RDB is NOT good if you need to minimize the chance of data loss in case Redis stops working (for example after a power outage). You can configure different save points where an RDB is produced (for instance after at least five minutes and 100 writes against the data set, but you can have multiple save points).
- RDB needs to fork() often in order to persist on disk using a child process. Fork() can be time consuming if the dataset is big, and may result in Redis to stop serving clients for some millisecond or even for one second if the dataset is very big and the CPU performance not great. AOF also needs to fork() but you can tune how often you want to rewrite your logs without any trade-off on durability.

*AOF adavantages*

- Using AOF Redis is much more durable: you can have different fsync policies: no fsync at all, fsync every second, fsync at every query.
- The AOF log is an append only log, so there are no seeks, nor corruption problems if there is a power outage. Even if the log ends with an half-written command for some reason (disk full or other reasons) the redis-check-aof tool is able to fix it easily.
- Redis is able to automatically rewrite the AOF in background when it gets too big. The rewrite is completely safe as while Redis continues appending to the old file, a completely new one is produced with the minimal set of operations needed to create the current data set, and once this second file is ready Redis switches the two and starts appending to the new one.
- AOF contains a log of all the operations one after the other in an easy to understand and parse format. You can even easily export an AOF file. For instance even if you flushed everything for an error using a FLUSHALL command, if no rewrite of the log was performed in the meantime you can still save your data set just stopping the server, removing the latest command, and restarting Redis again.

*AOF disadvantages*

- AOF files are usually bigger than the equivalent RDB files for the same dataset.
- AOF can be slower then RDB depending on the exact fsync policy.

------

*Snapshotting*

By default Redis saves snapshots of the dataset on disk, in a binary file called dump.rdb. You can configure Redis to have it save the dataset every N seconds if there are at least M changes in the dataset, or you can manually call the SAVE or BGSAVE commands.

*How it works*

Whenever Redis needs to dump the dataset to disk, this is what happens:

- Redis forks. We now have a child and a parent process.
- The child starts to write the dataset to a temporary RDB file.
- When the child is done writing the new RDB file, it replaces the old one.

------

*Append-only file*

The append-only file is an alternative, fully-durable strategy for Redis.

You can turn on the AOF in your configuration file:::

    appendonly yes

From now on, every time Redis receives a command that changes the dataset (e.g. SET) it will append it to the AOF. When you restart Redis it will re-play the AOF to rebuild the state.

------

*Log rewriting*

As you can guess, the AOF gets bigger and bigger as write operations are performed. For example, if you are incrementing a counter 100 times, you'll end up with a single key in your dataset containing the final value, but 100 entries in your AOF. 99 of those entries are not needed to rebuild the current state.

So Redis supports an interesting feature: it is able to rebuild the AOF in the background without interrupting service to clients. Whenever you issue a BGREWRITEAOF Redis will write the shortest sequence of commands needed to rebuild the current dataset in memory.

------

.. seealso:: `Redis Persistence <http://redis.io/topics/persistence>`_


**Replication**

*How Redis replication works*

If you set up a slave, upon connection it sends a SYNC command. And it doesn't matter if it's the first time it has connected or if it's a reconnection.

The master then starts background saving, and collects all new commands received that will modify the dataset. When the background saving is complete, the master transfers the database file to the slave, which saves it on disk, and then loads it into memory. The master will then send to the slave all accumulated commands, and all new commands received from clients that will modify the dataset. This is done as a stream of commands and is in the same format of the Redis protocol itself.

*Configuration*

To configure replication is trivial: just add the following line to the slave configuration file:::

    slaveof 192.168.1.1 6379

.. seealso:: `Repication <http://redis.io/topics/replication>`_

**Transactions**

*MULTI* , *EXEC* , *DISCARD* and *WATCH* are the foundation of transactions in Redis. They allow the execution of a group of commands in a single step, with two important guarantees:

- All the commands in a transaction are serialized and executed sequentially. It can never happen that a request issued by another client is served in the middle of the execution of a Redis transaction. This guarantees that the commands are executed as a single isolated operation.
- Either all of the commands or none are processed, so a Redis transaction is also atomic. The EXEC command triggers the execution of all the commands in the transaction, so if a client loses the connection to the server in the context of a transaction before calling the MULTI command none of the operations are performed, instead if the EXEC command is called, all the operations are performed.However if the Redis server crashes or is killed by the system administrator in some hard way it is possible that only a partial number of operations are registered. Redis will detect this condition at restart, and will exit with an error. Using the redis-check-aof tool it is possible to fix the append only file that will remove the partial transaction so that the server can start again.

.. seealso:: `Transactions <http://redis.io/topics/transactions>`_

**Pub/Sub**

SUBSCRIBE, UNSUBSCRIBE and PUBLISH implement the Publish/Subscribe messaging paradigm where (citing Wikipedia) senders (publishers) are not programmed to send their messages to specific receivers (subscribers).

**Redis Administration**

*Redis setup hints*

- suggest deploying Redis using the Linux operating system.
- Make sure to set the Linux kernel overcommit memory setting to 1. Add vm.overcommit_memory = 1 to /etc/sysctl.conf and then reboot or run the command sysctl vm.overcommit_memory=1 for this to take effect immediately.
- Make sure to setup some swap in your system (we suggest as much as swap as memory). If Linux does not have swap and your Redis instance accidentally consumes too much memory, either Redis will crash for out of memory or the Linux kernel OOM killer will kill the Redis process.
- Even if you have persistence disabled, Redis will need to perform RDB saves if you use replication.
- Use daemonize no when run under daemontools.

**Redis configuration**

Redis is able to start without a configuration file using a built-in default configuration, however this setup is only recommanded for testing and development purposes.

The proper way to configure Redis is by providing a Redis configuration file, usually called redis.conf.

*Changing Redis configuration while the server is running*

It is possible to reconfigure Redis on the fly without stopping and restarting the service, or querying the current configuration programmatically using the special commands CONFIG SET and CONFIG GET

Not all the configuration directives are supported in this way, but most are supported as expected. Please refer to the CONFIG SET and CONFIG GET pages for more information.

Note that modifying the configuration on the fly has no effects on the redis.conf file so at the next restart of Redis the old configuration will be used instead.

Make sure to also modify the redis.conf file accordingly to the configuration you set using CONFIG SET. There are plans to provide a CONFIG REWRITE command that will be able to run the redis.conf file rewriting the configuration accordingly to the current server configuration, without modifying the comments and the structure of the current file.


《Redis设计与实现》读书笔记
------------------------------

Redis中的每个数据库，都由一个redis.h/redisDb结构表示::

    typedef struct redisDb {
        // 保存着数据库以整数表示的号码
        int id;

        // 保存着数据库中的所有键值对数据
        // 这个属性页被称为键空间（key space）
        dict *dict;

        // 保存着键的过期时间
        dict *expires;

        // 实现列表阻塞原语，如 BLPOP
        // 在列表类型一章有详细的讨论
        dict *blocking_keys;
        dict *ready_keys;

        // 用于实现 WATCH 命令
        // 在事务章节有详细的讨论
        dict *watched_keys;
    } redisDb;

**数据库的切换**

redisDb结构的id域保存着数据库的号码，但它并不是SELECT命令所使用的数据库编号，
而是给Redis内部程序使用的。

当 Redis 服务器初始化时， 它会创建出 redis.h/REDIS_DEFAULT_DBNUM 个数据库，
并将所有数据库保存到 redis.h/redisServer.db 数组中， 每个数据库的 id 为从0
到REDIS_DEFAULT_DBNUM - 1的值。

当执行 SELECT number 命令时，程序直接使用 redisServer.db[number] 来切换数据库。

**数据库键空间**

因为Redis是一个键值对数据库（key-value pairs database），所以它的数据库本身也是一个字典（俗称 key space）：

- 字典的键是一个字符串对象。
- 字典的值则可以是包括字符串、列表、哈希表、集合或有序集在内的任意一种Redis类型对象。

在 redisDb 结构的 dict 属性中，保存着数据库的所有键值对数据。

**键空间的操作**

*取值*

在数据库中取值实际上就是在字典空间中取值， 再加上一些额外的类型检查：

- 键不存在，返回空回复；
- 键存在，且类型正确，按照通讯协议返回值对象；
- 键存在，但类型不正确，返回类型错误。

**键的过期时间**

通过 EXPIRE 、 PEXPIRE 、 EXPIREAT 和 PEXPIREAT 四个命令， 客户端可以给某个存在的键设置过期时间，
当键的过期时间到达时，键就不再可用。

命令 TTL 和 PTTL 则用于返回给定键距离过期还有多长时间。

**过期时间的保存**

在数据库中， 所有键的过期时间都被保存在 redisDb 结构的 expires 字典里。

expires 字典的键是一个指向 dict 字典（键空间）里某个键的指针， 而字典的值则是键所指向的数据库键的到期时间，
这个值以 long long 类型表示。

**设置生存时间**

Redis 有四个命令可以设置键的生存时间（可以存活多久）和过期时间（什么时候到期）：

- EXPIRE 以秒为单位设置键的生存时间；
- PEXPIRE 以毫秒为单位设置键的生存时间；
- EXPIREAT 以秒为单位，设置键的过期 UNIX 时间戳；
- PEXPIREAT 以毫秒为单位，设置键的过期 UNIX 时间戳。

虽然有那么多种不同单位和不同形式的设置方式， 但是 expires 字典的值只保存“以毫秒为单位的过期 UNIX 时间戳”，
这就是说，通过进行转换， 所有命令的效果最后都和 PEXPIREAT 命令的效果一样。


资源
-------

- `Redis设计与实现 <http://www.redisbook.com/en/latest/index.html>`_
- `Redis命令参考 <https://redis.readthedocs.org/en/latest/index.html>`_
- `Redis 常见的性能问题和解决方法 <http://zhupan.iteye.com/blog/1576108>`_
- `Redis几个认识误区 <http://timyang.net/data/redis-misunderstanding/>`_
- `Redis经验谈 <http://www.programmer.com.cn/14577/>`_
- `Redis Masterclass - Part 1, Configuration <http://snmaynard.com/2013/01/14/redis-masterclass-part-one-configuring-redis/>`_
- `Redis Masterclass - Part 2, Monitoring <http://snmaynard.com/2013/01/22/redis-masterclass-part-two-monitoring-redis/>`_
