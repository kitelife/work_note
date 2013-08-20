HAProxy配置文档（精简版）
============================

2. 配置HAProxy
-----------------------

2.1 配置文件的格式
^^^^^^^^^^^^^^^^^^^^^^^^^

HAProxy的配置过程包含3个主要的参数来源：

- 来自命令行的参数，始终是优先的
- 配置文件的“global”部分，设置进程全局范围有效的参数
- 配置文件的代理部分，以“defaults”、“listen”、“frontend”和“backend”的形式出现

配置文件的语法是：一行一行的配置信息，每行以本手册涉及的关键词开始；跟随一个或多个可选参数，以空格分隔。如果空格必须出现在字符串中，则空格必须前置一个反斜杠（\\）进行转义。反斜杠自己如果要出现在字符串中，也需要写两遍来转义。

2.2 时间格式
^^^^^^^^^^^^^^^^^

某些参数值表示时间，如超时时间。这些值通常以毫秒为单位表示（除非显式声明为其他单位），也可以在数值后添加单位后缀来声明使用其他单位。谨记这一点非常重要，我们后面不会对关键词的时间参数重复解释。支持的单位有：

- us：微秒。1微秒 = 1/1000000秒
- ms：毫秒。1毫秒 = 1/1000秒，默认单位
- s：秒。1秒 = 1000毫秒
- m：分钟。1分钟 = 60秒 = 60000毫秒
- h：小时。1小时 = 60分钟 = 3600秒 = 3600000毫秒
- d：天。1天 = 24小时 = 1440分钟 = 86400秒 = 86400000毫秒

2.3 示例
^^^^^^^^^^^^^^

::

    # 简单配置一个HTTP代理监听80端口，将请求转发到一个后端“servers”，
    # 该后端仅有一个服务器“server1”监听着127.0.0.1:8000
    global
        daemon
        maxconn 256

    defaults
        mode http
        timeout connect 5000ms
        timeout client 50000ms
        timeout server 50000ms

    frontend http-in
        bind *:80
        default_backend servers

    backend servers
        server server1 127.0.0.1:8000 maxconn 32

    # 以一个listen块来定义与上例相同的配置。更简短但
    # 表达能力弱一些，特别是在HTTP模式下
    global
        daemon
        maxconn 256

    defaults
        mode http
        timeout connect 5000ms
        timeout client 50000ms
        timeout server 50000ms

    listen http-in
        bind *:80
        server server1 127.0.0.1:8000 maxconn 32


假设haproxy命令在$PATH定义的查找路径中，可以这样在shell中测试上述配置：

::

    $ sudo haproxy -f configuration.conf -c


3. 全局参数
---------------

“global”部分的参数是进程全局范围有效的，通常也和操作系统相关。这些参数通常设置一次就一劳永逸了。其中某些参数具有等价的命令行选项。

“global”部分支持以下关键词：

- 进程管理与安全
  - chroot
  - daemon
  - gid
  - group
  - log
  - log-send-hostname
  - nbproc
  - pidfile
  - uid
  - ulimit-n
  - user
  - stats
  - node
  - description

- 性能调优
  - maxconn
  - maxpipes
  - noepoll
  - nokqueue
  - nopoll
  - nosepoll
  - nosplice
  - spread-checks
  - tune.bufsize
  - tune.chksize
  - tune.maxaccept
  - tune.maxpollevents
  - tune.maxrewrite
  - tune.rcvbuf.client
  - tune.rcvbuf.server
  - tune.sndbuf.client
  - tune.sndbuf.server

- 调试
  - debug
  - quiet

3.1 进程管理与安全
^^^^^^^^^^^^^^^^^^^^^^^^^^
