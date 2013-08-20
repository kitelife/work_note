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

**chroot <jail dir>**

将当前目录修改为 ``<jail dir>`` ，并在失去权限之前在该目录下执行一次chroot()。这能够在某些不明漏洞被攻击的情况下提高安全级别，因为这会使得骇客很难攻击系统。但这仅在进程以超级用户权限启动时才有效。一定要确保<jail dir>目录为空且任何人都不可写。

**daemon**

使进程fork为守护进程。生产运营环境（operation）下的推荐模式，等价于命令行的“-D”选项。也可以通过命令行“-db”选项来禁用。

**log <address> <facility> [max level [min level]]**

添加一个全局syslog服务器。可以定义多达两个的全局服务器。它们会收到进程启动和退出时的日志，还有配置有“log global”的代理的所有日志。

<address>可以以下两者之一：

- 一个IPv4地址，可选跟随一个冒号和一个UDP端口。如果未指定端口，则默认使用514（标准syslog端口）。
- 一个UNIX域套接字（socket）的文件系统路径，留意chroot（确保在chroot中可以访问该路径）和uid/gid（确保该路径相应地可写）。

<facility>必须是24个标准syslog设备（facilities）之一：

::

    kern    user    mail    daemon  auth    syslog  lpr     news
    uucp    cron    auth2   ftp     ntp     audit   alert   cron2
    local0  local1  local2  local3  local4  local5  local6  local7

可以指定一个可选的级别来过滤输出的信息。默认情况下会发送所有信息。如果指定了一个最大级别，那么仅有严重性至少同于该级别的信息才会被发送。也可以指定一个可选的最小级别，如果设置了，则以比该级别更严重的级别发送的日志会被覆盖到该级别。这样能避免一些默认syslog配置将“emerg”信息发送到所有终端。八个级别如下所示：

::

    emerg   alert   crit    err     warning     notice  info    debug

**log-send-hostname [<string>]**

设置syslog头部的主机名称（hostname）字段。如果设置了可选的“string”参数，那么头部主机名称字段会被设置为string内容，否则使用系统的主机名。通常用于不通过一个中间syslog服务器来转发日志信息之时，或仅仅是为了定制日志中打印的主机名称。

**log-tag <string>**

将syslog头部的tag字段设置为string。默认为从命令行启动的程序名称，一般为“haproxy”。有时为了区分同一主机上的多个进程会比较有用。

**nbproc <number>**

当以守护进程方式运行时创建<number>个进程。该指令要求“daemon”模式。默认情况下，仅会创建一个进程，这是生产运营环境下的推荐模式。对于每个进程可持有的文件描述符较少的系统，也许需要fork多个守护进程。但使用多个进程调试起来会更困难，所以非常不推荐使用。

**pidfile <pidfile>**

将所有守护进程的pid写到文件<pidfile>中。该选项等价于命令行参数“-p”。该文件对于启动进程的用户必须是可访问的。

**ulimit-n <number>**

将单个进程可持有文件描述符的最大数目设置为<number>。默认情况下会自动计算出该值，因此建议不使用该选项。

注：linux下有个命令：ulimit，进一步阅读： `通过 ulimit 改善系统性能 <http://www.ibm.com/developerworks/cn/linux/l-cn-ulimit/>`_ 。

**node <name>**

仅允许使用字母、数字、连字符和下划线，就像DNS名称。

在HA配置中当有两个或更多进程或服务器共享相同的IP地址时，该语句比较有用。通过为所有节点设置一个不同的节点名称，非常易于即时找到哪台服务器正在处理流量。

**description <text>**

添加一段文本来描述当前实例。

注意：要求对某些字符（例如 ``#`` ）进行转移，并且该文本会被插入到一个html页面中，所以应避免使用“<”和“>”字符。

3.2 性能调优
^^^^^^^^^^^^^^^^^^

**maxconn <number>**

将单个进程的最大并发连接数目设置为<number>。该选项等价于命令行参数“-n”。当达到该限制时，代理会停止接受连接。“ulimit-n”参数的值会根据该值自动调整。

**maxpipes <number>**

将单个进程可持有管道的最大数目设置为<number>。目前，管道仅用于基于内核的TCP粘合（tcp splicing，参考阅读： `TCP Splicing <http://kb.linuxvirtualserver.org/wiki/TCP_Splicing>`_ ）。由于每个管道包含两个文件描述符，“ulimit-n”的值也会相应地增大。默认值为maxconn/4，对于大多数任务繁重的使用情况似乎已远远够用了。粘合的代码会动态分配和释放管道，也可以回退为标准拷贝，因此将该值设置得过低应该仅会影响性能。

**spread-checks <0..50, in percent>**

有时会期望不要以精确的时间间隔发送服务器健康检测数据包，例如当多个逻辑服务器位于同一物理服务器上。借助该参数，在检测时间间隔上添加一点随机性是可能的。2到5之间的一个数值看起来效果不错。默认值为0。

**tune.bufsize <number>**

将缓冲区大小设置为<number>（以字节为单位）。更低的值允许在相同大小的内存中共存更多会话，而更高的值允许一些带有大量cookie的应用能够正常工作。默认值为16384，在编译构建之时可以修改。强烈建议不要修改默认值，因为非常低的值会破坏某些服务，如统计，而大于默认大小的值会增大内存使用量，可能会导致系统耗尽内存。随着这个值的增大，全局的maxconn参数至少应该减小相同的比率。

**tune.chksize <number>**

将检测缓冲区大小设置为该值（单位为字节）。更高的值也许有助于在海量页面中找到字符串或正则模式，虽然这样做也许意味着更多的内存和CPU使用量。默认值为16384，可以在编译构建时修改。不推荐改变该值，但是可能的话还是让检测更优吧（啥检测？目前还没搞懂）

**tune.maxaccept <number>**

设置一个进程在一次唤醒中也许会处理的连续接收的请求的最大数目（the maximum number of consecutive accepts that a process may perform on a single wake up）。更高的值给予高请求连接率更高优先级，更低的值则给予已建立的连接更高的（处理）优先级。在单进程模式下，该值默认限制为100。然而，在多进程模式中（nbproc > 1），该值默认为8，这样当一个进程唤醒时，它不会自己接受所有进入的连接，而是将一部分请求留给其他进程。将该值设置为-1会完全禁用这个限制。通常不需要变动该值。

**tune.maxpollevents <number>**

设置在对轮询系统（polling system）的一次调用中能同时（at once）处理的事件的最大数目。默认值不同的操作系统有所不同。需要注意的是将该值减小到200以下往往会以网络带宽为代价稍微减小延迟，而增大该值至200以上则是以延迟换取稍稍提升的网络带宽。

**tune.maxrewrite <number>**

将保留的缓冲空间设置为该值，以字节为单位。该保留空间是用于头部重写或附加信息的。The first reads on sockets will never fill more than bufsize-maxrewrite（啥意思？）。一直以来该值都默认为bufsize的一半，但这样做并没有多大的意义，因为很少需要添加大量头部。该值设置得过高会影响大量请求或响应的处理。设置得过低又会阻碍为大量请求增加新头部信息或影响POST请求。将该值设置为1024左右通常是比较明智的。如果比那还大，它会自动重新调整为bufsize的一半。这意味着修改bufsize时你并不需要担心该值。

**tune.rcvbuf.client <number>**

**tune.rcvbuf.server <number>**

将客户端或服务器端的内核套接字接收缓冲大小强制设置为指定大小（单位为字节）。该值应用于所有TCP/HTTP前端和后端。一般不应该设置，默认值（0）让内核根据内存可用量自动优化该值。然而有时为了通过阻止缓冲过多接收到的数据来节省内核内存，将该参数设置为一个非常低的值（如：4096）会有助于此。但是更低的值会显著地增大CPU使用量。

**tune.sndbuf.client <number>**

**tune.sndbuf.server <number>**

3.3 调试
^^^^^^^^^^^^^^^^^

**debug**

启用debug模式，将所有来往信息输出到标准输出，并进程无法在后台运行（即无法成为守护进程？）。该指令等价于命令行参数“-d”。在生产环境配置中绝对不应该使用，因为它会阻碍完整的系统启动（为什么？什么意思？）。

4. 代理
--------------

代理配置可以位于以下部分中：

- defaults <name>
- frontend <name>
- backend <name>
- listen <name>

一个“defaults”部分是为该部分之后的所有其他部分设置默认参数。那些默认参数可以在下一个“defaults”部分中被重置。name是可选的，但为了更好的可读性，鼓励使用。

一个“frontend”部分描述一组接受客户端连接的监听套接字。

一个“backend”部分则是描述一组服务器，代理会将进入的连接转发到这些服务器。

一个“listen”部分定义一个完整的代理，将前端后端部分合并入一个部分中。对于仅是TCP的流量通常比较有用（为什么？）。

所有代理的名字必须由大小写字母、数字、‘-’（连字符）、‘_’（下划线）、‘.’（点）和‘:’（冒号）中的字符组成。ACL名称是大小写敏感的，这意味着“www”和“WWW”是两个不同的代理。

当前，支持两种主要的代理模式：“tcp”，又名4层（协议）和“http”，又名7层（协议）。在4层模式中，HAProxy简单地在两方之间转发双向流量。在7层模式中，HAProxy会分析协议，并且可以基于任意标准来允许、阻止、转换、增加、修改或删除请求或响应中的任意内容来与协议交互。

4.1 代理关键词参考手册
^^^^^^^^^^^^^^^^^^^^^^^^^^^^

（仅对部分关键词进行说明）
