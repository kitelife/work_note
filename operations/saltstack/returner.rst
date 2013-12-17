Returners
===============

The returner interface allows the return data to be sent to any system that can receive data. This means that return data can be sent to a Redis server, a MongoDB server, a MySQL server, or any system!

Using Returner
----------------

All commands will return the command data back to the master. Adding more returners will ensure that the data is also sent to the specified returner interfaces.

Specify what returners to use is done when the command is invoked:

::

    salt '*' test.ping --return redis_return

It is also possible to specify multiple returners:

::

    salt '*' test.ping --return mongo_return,redis_return,cassandra_return

Writing a Returner
--------------------------

A returner is a module which contains a returner function, the returner function must accept a single argument. this argument is the return data from the called minion function. So if the minion function ``test.ping`` is called 
the value of the argument will be ``True`` .

You can place your custom returners in a ``_returners`` directory within the ``file_roots`` specified by the master config file. These custom returners are distributed when ``state.highstate`` is run, 
or by executing the ``saltutil.sync_returners`` or ``saltutil.sync_all`` functions.

Any custom returners which have been synced to a minion, that are named the same as one of Salt's default set of returners, will take the place of the default returner with the same name. Note that a returner's default name is 
its filename (i.e. ``foo.py`` becomes returner ``foo``), but that its name can be overridden by using a `__virtual__ function <https://salt.readthedocs.org/en/latest/ref/modules/index.html#virtual-modules>`_.

returner所需要的某些参数（如MySQL的主机名、用户名、密码等）可以在minion的配置文件里设置，详细参考 `mysql returner源码 <https://github.com/saltstack/salt/blob/develop/salt/returners/mysql.py>`_
