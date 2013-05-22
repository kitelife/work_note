Redis
===========

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
