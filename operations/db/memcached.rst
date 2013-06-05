Memcached
==============

读文笔记：Scaling Memcache at Facebook
-------------------------------------------------

**Abstract**

How Facebook leverages memcached as a building block to construct and scale a distributed key-value store that supports the world's largest social network.

**Introduction**

A social network's infrastructure needs to (1) allow near realtime communication, (2) aggregate content on-the-fly from multiple sources, (3) be able to access and update very popular shared content, and (4) scale to process millions of user requests per second.


.. seealso:: `中文翻译版 <http://www.oschina.net/translate/scaling-memcache-facebook>`_ ,  `英文原版 <https://www.usenix.org/conference/nsdi13/scaling-memcache-facebook>`_