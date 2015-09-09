## HAProxy配置文档（精简版）

### 2. 配置HAProxy

#### 2.1 配置文件的格式

HAProxy的配置过程包含3个主要的参数来源：

- 来自命令行的参数，始终是优先的
- 配置文件的“global”部分，设置进程全局范围有效的参数
- 配置文件的代理部分，以“defaults”、“listen”、“frontend”和“backend”的形式出现

配置文件的语法是：一行一行的配置信息，每行以本手册涉及的关键词开始；跟随一个或多个可选参数，以空格分隔。如果空格必须出现在字符串中，则空格必须前置一个反斜杠（\\）进行转义。反斜杠自己如果要出现在字符串中，也需要写两遍来转义。

#### 2.2 时间格式

某些参数值表示时间，如超时时间。这些值通常以毫秒为单位表示（除非显式声明为其他单位），也可以在数值后添加单位后缀来声明使用其他单位。谨记这一点非常重要，我们后面不会对关键词的时间参数重复解释。支持的单位有：

- us：微秒。1微秒 = 1/1000000秒
- ms：毫秒。1毫秒 = 1/1000秒，默认单位
- s：秒。1秒 = 1000毫秒
- m：分钟。1分钟 = 60秒 = 60000毫秒
- h：小时。1小时 = 60分钟 = 3600秒 = 3600000毫秒
- d：天。1天 = 24小时 = 1440分钟 = 86400秒 = 86400000毫秒

#### 2.3 示例

```
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
```


假设haproxy命令在$PATH定义的查找路径中，可以这样在shell中测试上述配置：

```
$ sudo haproxy -f configuration.conf -c
```


### 3. 全局参数

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

#### 3.1 进程管理与安全

**chroot <jail dir>**

将当前目录修改为 ``<jail dir>`` ，并在失去权限之前在该目录下执行一次chroot()。这能够在某些不明漏洞被攻击的情况下提高安全级别，因为这会使得骇客很难攻击系统。但这仅在进程以超级用户权限启动时才有效。一定要确保<jail dir>目录为空且任何人都不可写。

**daemon**

使进程fork为守护进程。生产运营环境（operation）下的推荐模式，等价于命令行的“-D”选项。也可以通过命令行“-db”选项来禁用。

**``log <address> <facility> [max level [min level]]``**

添加一个全局syslog服务器。可以定义多达两个的全局服务器。它们会收到进程启动和退出时的日志，还有配置有“log global”的代理的所有日志。

`<address>`可以以下两者之一：

- 一个IPv4地址，可选跟随一个冒号和一个UDP端口。如果未指定端口，则默认使用514（标准syslog端口）。
- 一个UNIX域套接字（socket）的文件系统路径，留意chroot（确保在chroot中可以访问该路径）和uid/gid（确保该路径相应地可写）。

`<facility>`必须是24个标准syslog设备（facilities）之一：

```
kern    user    mail    daemon  auth    syslog  lpr     news
uucp    cron    auth2   ftp     ntp     audit   alert   cron2
local0  local1  local2  local3  local4  local5  local6  local7
```

可以指定一个可选的级别来过滤输出的信息。默认情况下会发送所有信息。如果指定了一个最大级别，那么仅有严重性至少同于该级别的信息才会被发送。也可以指定一个可选的最小级别，如果设置了，则以比该级别更严重的级别发送的日志会被覆盖到该级别。这样能避免一些默认syslog配置将“emerg”信息发送到所有终端。八个级别如下所示：

```
emerg   alert   crit    err     warning     notice  info    debug
```

**``log-send-hostname [<string>]``**

设置syslog头部的主机名称（hostname）字段。如果设置了可选的“string”参数，那么头部主机名称字段会被设置为string内容，否则使用系统的主机名。通常用于不通过一个中间syslog服务器来转发日志信息之时，或仅仅是为了定制日志中打印的主机名称。

**``log-tag <string>``**

将syslog头部的tag字段设置为string。默认为从命令行启动的程序名称，一般为“haproxy”。有时为了区分同一主机上的多个进程会比较有用。

**``nbproc <number>``**

当以守护进程方式运行时创建<number>个进程。该指令要求“daemon”模式。默认情况下，仅会创建一个进程，这是生产运营环境下的推荐模式。对于每个进程可持有的文件描述符较少的系统，也许需要fork多个守护进程。但使用多个进程调试起来会更困难，所以非常不推荐使用。

**``pidfile <pidfile>``**

将所有守护进程的pid写到文件<pidfile>中。该选项等价于命令行参数“-p”。该文件对于启动进程的用户必须是可访问的。

**``ulimit-n <number>``**

将单个进程可持有文件描述符的最大数目设置为<number>。默认情况下会自动计算出该值，因此建议不使用该选项。

注：linux下有个命令：ulimit，进一步阅读： [通过 ulimit 改善系统性能](http://www.ibm.com/developerworks/cn/linux/l-cn-ulimit/)。

**``node <name>``**

仅允许使用字母、数字、连字符和下划线，就像DNS名称。

在HA配置中当有两个或更多进程或服务器共享相同的IP地址时，该语句比较有用。通过为所有节点设置一个不同的节点名称，非常易于即时找到哪台服务器正在处理流量。

**``description <text>``**

添加一段文本来描述当前实例。

注意：要求对某些字符（例如 ``#`` ）进行转移，并且该文本会被插入到一个html页面中，所以应避免使用“<”和“>”字符。

#### 3.2 性能调优

**``maxconn <number>``**

将单个进程的最大并发连接数目设置为<number>。该选项等价于命令行参数“-n”。当达到该限制时，代理会停止接受连接。“ulimit-n”参数的值会根据该值自动调整。

**``maxpipes <number>``**

将单个进程可持有管道的最大数目设置为<number>。目前，管道仅用于基于内核的TCP粘合（tcp splicing，参考阅读：[TCP Splicing](http://kb.linuxvirtualserver.org/wiki/TCP_Splicing) ）。由于每个管道包含两个文件描述符，“ulimit-n”的值也会相应地增大。默认值为maxconn/4，对于大多数任务繁重的使用情况似乎已远远够用了。粘合的代码会动态分配和释放管道，也可以回退为标准拷贝，因此将该值设置得过低应该仅会影响性能。

**``spread-checks <0..50, in percent>``**

有时会期望不要以精确的时间间隔发送服务器健康检测数据包，例如当多个逻辑服务器位于同一物理服务器上。借助该参数，在检测时间间隔上添加一点随机性是可能的。2到5之间的一个数值看起来效果不错。默认值为0。

**``tune.bufsize <number>``**

将缓冲区大小设置为<number>（以字节为单位）。更低的值允许在相同大小的内存中共存更多会话，而更高的值允许一些带有大量cookie的应用能够正常工作。默认值为16384，在编译构建之时可以修改。强烈建议不要修改默认值，因为非常低的值会破坏某些服务，如统计，而大于默认大小的值会增大内存使用量，可能会导致系统耗尽内存。随着这个值的增大，全局的maxconn参数至少应该减小相同的比率。

**``tune.chksize <number>``**

将检测缓冲区大小设置为该值（单位为字节）。更高的值也许有助于在海量页面中找到字符串或正则模式，虽然这样做也许意味着更多的内存和CPU使用量。默认值为16384，可以在编译构建时修改。不推荐改变该值，但是可能的话还是让检测更优吧（啥检测？目前还没搞懂）

**``tune.maxaccept <number>``**

设置一个进程在一次唤醒中也许会处理的连续接收的请求的最大数目（the maximum number of consecutive accepts that a process may perform on a single wake up）。更高的值给予高请求连接率更高优先级，更低的值则给予已建立的连接更高的（处理）优先级。在单进程模式下，该值默认限制为100。然而，在多进程模式中（nbproc > 1），该值默认为8，这样当一个进程唤醒时，它不会自己接受所有进入的连接，而是将一部分请求留给其他进程。将该值设置为-1会完全禁用这个限制。通常不需要变动该值。

**``tune.maxpollevents <number>``**

设置在对轮询系统（polling system）的一次调用中能同时（at once）处理的事件的最大数目。默认值不同的操作系统有所不同。需要注意的是将该值减小到200以下往往会以网络带宽为代价稍微减小延迟，而增大该值至200以上则是以延迟换取稍稍提升的网络带宽。

**``tune.maxrewrite <number>``**

将保留的缓冲空间设置为该值，以字节为单位。该保留空间是用于头部重写或附加信息的。The first reads on sockets will never fill more than bufsize-maxrewrite（啥意思？）。一直以来该值都默认为bufsize的一半，但这样做并没有多大的意义，因为很少需要添加大量头部。该值设置得过高会影响大量请求或响应的处理。设置得过低又会阻碍为大量请求增加新头部信息或影响POST请求。将该值设置为1024左右通常是比较明智的。如果比那还大，它会自动重新调整为bufsize的一半。这意味着修改bufsize时你并不需要担心该值。

**``tune.rcvbuf.client <number>``**

**``tune.rcvbuf.server <number>``**

将客户端或服务器端的内核套接字接收缓冲大小强制设置为指定大小（单位为字节）。该值应用于所有TCP/HTTP前端和后端。一般不应该设置，默认值（0）让内核根据内存可用量自动优化该值。然而有时为了通过阻止缓冲过多接收到的数据来节省内核内存，将该参数设置为一个非常低的值（如：4096）会有助于此。但是更低的值会显著地增大CPU使用量。

**``tune.sndbuf.client <number>``**

**``tune.sndbuf.server <number>``**

#### 3.3 调试

**debug**

启用debug模式，将所有来往信息输出到标准输出，并进程无法在后台运行（即无法成为守护进程？）。该指令等价于命令行参数“-d”。在生产环境配置中绝对不应该使用，因为它会阻碍完整的系统启动（为什么？什么意思？）。

### 4. 代理

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

#### 4.1 代理关键词参考手册

（仅对部分关键词进行说明）

**``acl <aclname> <criterion> [flags] [operator] <value> ...``**

可用于：frontend、listen、backend

“ACL”：Access Control List

声明或添加一个访问控制列表。其他地方根据<aclname>调用该acl进行条件判断做出不同的行为。

------

**``appsession <cookie> len <length> timeout <holdtime> [request-learn] [prefix] [mode <path-parameters|query-string>]``**

可用于：listen、backend

参考文章： [haproxy 解决集群session 共享问题方法](http://chenwenming.blog.51cto.com/327092/841043)

------

**``balance <algorithm> [ <arguments> ]``**

**``balance url_param <param> [check_post [<max_wait>]]``**

可用于：defaults、listen、backend

定义用于后端的负载均衡算法。

参考文章： [HAProxy 配置手册 4.2 balance 相关](http://hi.baidu.com/maiyudaodao/item/cf01041b5162764a6926bbe8)

------

**``block { if | unless } <condition>``**

可用于：frontend、listen、backend

如果/除非匹配到某条件，则阻止一个7层协议的请求。

如果/除非匹配到<condition>，HTTP请求在7层处理中很早就会被阻止掉。如果请求被阻止，则返回一个403错误。条件参考ACL。这一般用于拒绝对某些敏感资源的访问，如果符合/不符合某些条件。每个实例中“block”语句的数量并没有一个固定的限制。

示例：

```
acl invalid_src src         0.0.0.0/7 224.0.0.0/3
acl invalid_src src_port    0:1023
acl local_dst   hdr(host) -i localhost
block if invalid_src | local_dst
```

------

**``capture cookie <name> len <length>``**

可用于：frontend、listen

捕获并记录请求或相应中的cookie。

*参数* ：

```
<name> 是所要捕获cookie的名称的开始部分。若想精确匹配名称，只需为名称添加一个等于号的后缀即可。完整的名称会出现在日志中。

<length> 是能在日志中记录的字符最大长度，包括cookie名称，等于号和值，以“名称=值”的形式出现。如果字符串长度超过了<length>，则会截掉右边部分。
```

*示例* ：

``` 
capture cookie ASPSESSION len 32
```

------

**``capture request header <name> len <length>``**

可用于：frontend、listen

捕获记录指定请求头首次出现的值。

*示例* :

```
capture request header Host len 15
capture request header X-Forwarded-For len 15
capture request header Referrer len 15
```

------

**``capture response header <name> len <length>``**

可用于：frontend、listen

捕获记录指定响应头首次出现的值。

*示例* ：

```
capture response header Content-length len 9
capture response header Location len 15
```

------

**``cookie <name> [ rewrite | insert | prefix ] [ indirect ] [ nocache ] [ postonly ] [ preserve ] [ httponly ] [ secure ] [ domain <domain> ]\* [ maxidle <idle> ] [ maxlife <life> ]``**

可用于：defaults、listen、backend

在后端（backend）启用基于cookie的持久性。

*HAProxy在WEB服务端发送给客户端的cookie中插入(或添加加前缀)haproxy定义的后端的服务器COOKIE ID。*

------

**default-server [param\*]**

可用于：defaults、listen、backend

修改后端服务器的默认参数。

*参数* :

```
<param*> 提供给服务器的一个参数列表。
```

*示例* :

```
default-server inter 1000 weight 13
```

------

**``default_backend <backend>``**

可用于：defaults、frontend、listen

为没有匹配到“use_backend”规则的情况指定使用的后端。

在使用关键词“use_backend”实现前端后端之间内容转换时，指明未匹配到规则时使用的后端通常比较有用。一般该后端是动态后端，用于接收所有不确定的请求。

*示例* :

```
use_backend     dynamic if  url_dyn
use_backend     static  if  url_css url_img extension_img
default_backend dynamic
```

------

**``errorfile <code> <file>``**

可用于：defaults、frontend、listen、backend

以一个文件的内容替换HAProxy生成的错误信息，返回给请求客户端。

*参数* ：

```
<code> HTTP状态码。目前，HAProxy支持产生200、400、403、408、500、502、503以及504的状态信息。

<file> 指定一个包含完整HTTP响应的文件。建议在文件名后加“.http”后缀，这样别人就不会将该响应内容与HTML错误页面相混淆。也建议使用绝对路径，因为是在执行任何chroot之前读取文件的。
```

需要理解的是该关键词的功能并不是改写服务器返回的错误信息，而是检测错误信息并由HAProxy返回响应。这也就是为什么仅支持一小部分的错误状态码。

Code 200 is emitted in response to requests matching a "monitor-uri" rule.

The files are read at the same time as the configuration and kept in memory. For this reason, the errors continue to be returned even when the process is chrooted, and no file change is considered while the process is running. A simple method for developing those files consists in associating them to the
403 status code and interrogating a blocked URL.

*示例* :

::

    errorfile 400 /etc/haproxy/errorfiles/400badreq.http
    errorfile 403 /etc/haproxy/errorfiles/403forbid.http
    errorfile 503 /etc/haproxy/errorfiles/503sorry.http

------

**``errorloc303 <code> <url>``**

可用于：defaults、frontend、listen、backend

返回一个HTTP重定向到一个URL，而不是返回HAProxy生成的错误信息。

*参数* ：

```
<code> HTTP状态码。目前，HAProxy支持状态码400、403、408、500、502、503和504。

<url> 响应头字段“Location”的确切内容。可能包含一个到相同站点上错误信息页面的相对URI，或一个到另一站点上错误信息页面的绝对URI。特别要注意的是使用相对URI要避免该URI自己也产生相同的错误（如：500）从而造成重定向成环。
```
------

**``hash-type <method>``**

可用于：defaults、listen、backend

指定一种方法用于将哈希值映射到服务器。

------

**``http-request { allow | deny | auth [realm <realm>] } [ { if | unless } <condition> ]``**

可用于：frontend、listen、backend

7层请求的访问控制。

这些选项允许精细地控制对一个frontend/listen/backend的访问。每个选项都可能跟随一个if/unless和acl。如果一个选项的条件被匹配到，则不再继续往后匹配。

每个实例的http-request语句的数量并没有固定的限制。

*示例* :

```
acl nagios src 192.168.129.3
acl local_net src 192.168.0.0/16
acl auth_ok http_auth(L1)

http-request allow if nagios
http-request allow if local_net auth_ok
http-request auth realm Gimme if local_net auth_ok
http-request deny
```

*示例* ：

```
acl auth_ok http_auth_group(L1) G1
    
http-request auth unless auth_ok
```

------

**``ignore-persist { if | unless } <condition>``**

可用于：frontend、listen、backend

声明一个忽略持久性的条件。

默认情况下，当启用cookie持久性，每个包含该cookie的请求都是无条件地持久的（假设目标服务器是启动运行的）。

“ignore-persist”语句允许我们声明各种基于ACL的条件，当条件匹配时，会导致一个请求忽略持久性。对于静态文件请求的负载均衡（通常不要求持久性）有时是有用的。也经常用于针对某个特定User-Agent完全禁用持久性（例如，一些网络抓取机器人）。

与“appsession”结合使用，也有助于减少HAProxy内存使用，因为如果持久性被忽略，appsession表就不会增大。

------

**mode { tcp|http|health }**

可用于：defaults、frontend、listen、backend

设置HAProxy实例的运行模式或协议。

*参数* :

```
tcp 实例将以纯TCP模式工作。在客户端与服务器之间会建立一个全双工的连接，也不会执行7层的检查。默认模式，应用于SSL、SSH、SMTP、...

http 实例将以HTTP模式工作。在连接到任何服务器之前，客户端请求会经过深度的分析。任何非RFC兼容的请求都会被拒绝。7层过滤、处理和转换都可能发生。该模式也是HAProxy的最大价值所在。

health 实例将以“health”模式工作。对进入的连接仅回复“OK”并关闭连接。不会记录任何东西。该模式用于回复外部组件的健康监测。该模式已过时，不应该再使用，因为结合TCP或HTTP模式与“monitor”关键词可以实现同样的功能甚至做得更好。
```

------

**``monitor fail { if | unless } <condition>``**

可用于：frontend、listen

添加一个条件用于向一个监控HTTP请求报告一个失败。

*示例* :

```
frontend www:
    mode http
    acl site_dead nbsrv(dynamic) lt 2
    acl site_dead nbsrv(static) lt 2
    monitor-uri /site_alive
    monitor fail if site_dead
```

------

**``monitor-net <source>``**

可用于：defaults、frontend、listen

声明一个仅可发送监控请求的源网络地址段。

*参数* ：

```
<source> 源IPv4地址或网络地址段，来自该地址或网络的请求仅能得到监控响应。可以是一个IPv4地址、主机名、或一个地址跟随一个斜杠（'/'）以及一个掩码。
```

Monitor requests are processed very early. It is not possible to block nor divert them using ACLs. They cannot be logged either, and it is the intended purpose. They are only used to report HAProxy's health to an upper component, nothing more. Right now, it is not possible to set failure conditions on requests caught by "monitor-net".

please note that only one "monitor-net" statement can be specified in a frontend. If more than one is found, only the last one will be considered.

*示例* ：

```
# address .252 and .253 are just probing us.
frontend www
    monitor-net 192.168.0.252/31
```

------

**``monitor-uri <uri>``**

可用于：defaults、frontend、listen

拦截一个外部组件监控请求的URI。

*参数* ：

```
<uri> 我们想要拦截的确切的URI，向其返回HAProxy的监控状态而不是转发该请求。
```

*示例* ：

```
# Use /haproxy_test to report haproxy's status
frontend www
    mode http
    monitor-uri /haproxy_test
```

------

**option allbackups**

**no option allbackups**

可用于：defaults、listen、backend

一次性使用所有备份服务器或仅使用第一个备份服务器。

默认情况下，当常规服务器都宕掉后，第一台可使用的备份服务器会接收所有流量。有时，可能一次性使用多台备份机器会更好，因为一台机器不够用啊。启用了“option allbackups”之后，当所有常规服务器都不可用时，就会在所有备份服务器上进行负载均衡。使用相同的负载均衡算法并考虑服务器的权重。因此，在备份服务器之间不再有任何优先顺序。

该选项多数时候用于静态服务器群，当一个应用完全下线时返回一个“抱歉”页面。

如果该选项在一个“defaults”部分启用了，则可以在某个具体实例中通过前置一个“no”关键词来禁用它。

------

**option checkcache**

**no option checkcache**

可用于：defaults、listen、backend

分析服务的所有响应并阻塞带有可缓存cookie的请求。

------

**option clitcpka**

**no option clitcpka**

可用于：defaults、frontend、listen

启用或禁用客户端TCP keepalive数据包的发送。

------

**option contstats**

可用于：defaults、frontend、listen

启用持续性的流量统计更新。

默认情况下，用于统计计算的计数器仅当一个会话结束时才增加。当服务小数据对象时，这样的工作效果相当不错，当对于大的数据对象（例如：大的图片或归档文件）或A/V流，haproxy计数器生成的统计图形看起来就像一个刺猬。启用该选项后，在整个会话期间计数器都会持续地增加。

------

**option dontlog-normal**

**no option dontlog-normal**

可用于：defaults、frontend、listen

启用或禁用对正常、成功连接的日志记录。

有些大型网站每秒要处理几千个连接，日志记录这些连接压力是很大的。其中有些网站被迫关闭日志记录，也就无法调试生产环境中的问题。设置该选项则haproxy不再日志记录正常的连接，即那些没有错误、没有超时、没有重试也没有重新分发的连接。这样硬盘空间不会出现异常情况。在HTTP模式中，会检测响应状态码，返回5xx状态码的连接还是会被日志记录的。

大多数时间强烈不推荐使用该选项，复杂问题的突破口往往是在常规的日志里，如果启用该选项，这些关键信息就不会被记录了。如果需要分离日志，可替代之使用“log-separate-errors”选项。

------

**option dontlognull**

**no option dontlognull**

可用于：defaults、frontend、listen

启用或禁用对空（null）连接的日志记录。

------

**``option forwardfor [ except <network> ] [ header <name> ] [ if-none ]``**

可用于：defaults、frontend、listen、backend

启用向发往服务器的请求中插入X-Forwarded-For请求头字段。

*参数* ：

```
<network> 可选参数用于为匹配<network>的来源禁用该选项。

<name> 可选参数指定一个不同的“X-Forwarded-For”头名称。
```

由于HAProxy工作在反向代理模式，服务器将它的IP地址当做请求的客户端地址。当期望服务器日志中有客户端IP地址时有时会令人烦恼。为了解决该问题，HAProxy可以向发往服务器的所有请求中添加众所周知的HTTP头部字段“X-Forwarded-For”。该头部字段的值代表客户端的IP地址。

*示例* ：

```
# Public HTTP address also used by stunnel on the same machine
frontend www
    mode http
    option forwardfor except 127.0.0.1  # stunnel already adds the header

# Those servers want the IP Address in X-Client
backend www
    mode http
    option forwardfor header X-Client
```

------

**``option httpchk``**

**``option httpchk <uri>``**

**``option httpchk <method> <uri>``**

**``option httpchk <method> <uri> <version>``**

可用于：defaults、listen、backend

启用HTTP检测服务器的健康状态。

*参数* :

```
<method> 可选的HTTP方法用于请求。若未设置，则使用“OPTIONS”方法，因为它通常只需要简单的服务器处理，也易于从日志中过滤掉。可以使用任意方法，但不推荐发明使用非标准方法。

<uri> HTTP请求中引用的URI。默认为“/”，因为几乎任何服务器默认都可以访问，但也可以修改为任何其他URI，查询字符串（Query string）也是允许的。

<version> 可选的HTTP版本字符串。默认为“HTTP/1.0”，但某些服务器对于HTTP 1.0的行为可能不正确，所以切换成HTTP/1.1有时可能会有帮助。注意Host头部字段在HTTP/1.1中是必须的。
```

默认情况下，服务器健康检测仅是建立一个TCP连接。当指定了“option httpchk”，一旦建立了TCP连接，就会发送一个完整的HTTP请求，并认为2xx和3xx响应是有效的，而其他的响应状态码都表示服务器挂了，包括没有任何响应。

This option does not necessarily require an HTTP backend, it also works with plain TCP backends. This is particularly useful to check simple scripts bound to some dedicated ports using the inetd daemon.

*示例* ：

```
# Relay HTTPS traffic to Apache instance and check service availability
# Using HTTP request "OPTIONS * HTTP/1.1" on port 80
backend https_relay
    mode tcp
    option httpchk OPTIONS * HTTP/1.1\r\nHost:\ www
    server apache1 192.168.1.1:443 check port 80
```

------

**option httpclose**

**no option httpclose**

可用于：defaults、frontend、listen、backend

启用或禁用被动HTTP连接关闭。

------

**option httplog [ clf ]**

可用于：defaults、frontend、listen、backend

启用日志记录HTTP请求、会话状态以及定时器（timers）。

*参数* ：

```
clf 若添加了“clf”参数，则输出格式为CLF格式而不是HAProxy的默认HTTP格式。当你需要将HAProxy的日志用于一个特定的日志分析器，而该分析器仅支持CLF格式且不可扩展，那就用这种格式。
```

默认情况下，日志输出格式非常简单，仅包含源地址和目的地址，以及实例名称。通过指定“option httplog”，每行日志就会转换为一种更加丰富的格式，包括但不限于：HTTP请求，连接定时器（connection timers），会话状态，连接数，捕获的数据包头部以及cookie，frontend、backend和服务器名称，当然还包括源地址和端口。

该选项可以在frontend或backend部分设置。

如果该选项在“defaults”部分启用，则可以在某个具体的实例中通过前置一个“no”关键词来禁用它。

------

**option ldap-check**

可用于：defaults、listen、backend

使用LDAPv3健康检测来测试服务器。

The server is considered valid only when the LDAP response contains success resultCode (http://tools.ietf.org/html/rfc4511#section-4.1.9).

*示例* ：

```
option ldap-check
```

------

**``option mysql-check [ user <username> ]``**

可用于：defaults、listen、backend

使用MySQL健康检测来测试服务器。

*参数* ：

```
<username> 用户名，用于连接到MySQL服务器。
```

------

**option nolinger**

**no option nolinger**

可用于：defaults、frontend、listen、backend

启用或禁用连接关闭后立即清空会话资源。

------

**option redispatch**

**no option redispatch**

可用于：defaults、listen、backend

启用或禁用连接失败之时重新分发会话。

------

**``option smtpchk``**

**``option smtpchk <hello> <domain>``**

可用于：defaults、listen、backend

使用SMTP健康检测来测试服务器。

*参数* :

```
<hello> 可选参数。即使用“hello”命令。可以是“HELO”（SMTP）或“EHLO”（ESTMP）。任何其他的值都会被转换成默认命令（“HELO”）。

<domain> 发送给服务器的域名。若指定了hello命令才需指定（也必须指定）。默认使用“localhost”。
```

------

**option splice-auto**

**no option splice-auto**

可用于：defaults、frontend、listen、backend

启用或禁用对于套接字双向地自动内核加速。

在frontend或backend中启用该选项后，haproxy会自动评估使用内核TCP拼接在客户端与服务器端间任一方向转发数据的可能性。

注意：基于内核的TCP拼接是一个Linux特有的特性，首次出现在内核2.6.25中。为套接字之间的数据传输提供基于内核的加速，无需将这些数据拷贝到用户空间，因此性能得到显著提升，节省了大量CPU时钟周期。

*示例* :

```
option splice-auto
```

------

**option splice-request**

**no option splice-request**

可用于：defaults、frontend、listen、backend

对请求启用或禁用套接字上的自动内核加速。

------

**option splice-response**

**no option splice-response**

可用于：defaults、frontend、listen、backend

对于响应启用或禁用套接字上的自动内核加速。

------

**option srvtcpka**

**no option srvtcpka**

可用于：defaults、listen、backend

启用或禁用服务器端发送TCP keepalive数据包。

当客户端与服务器之间有防火墙或其他会话相关的组件，当协议包含时间非常长的会话，并有较长的空闲期（如：远程桌面），那么中间组件可能会将长时间空闲的会话做过期处理。

启用套接字级别的TCP保活（keep-alives）使得系统定期发送数据包到连接的另一端，让其保持活跃。keep-alive探针（数据包）之间的延迟/间隔仅由系统控制，依赖于操作系统和调优参数。

关键要理解保活数据包不是在应用层面收发的。仅网络协议栈可以看到它们。因此，即使代理的一方已经使用保活数据包来维持连接的活跃，那些保活数据包不会被转发到代理的另一方。

注意这与HTTP keep-alive无关。

“srvtcpka”选项启用连接的服务器端发送TCP保活探针数据包，当HAProxy与服务器之间对于session过期比较敏感，该选项能提供帮助。

------

**option ssl-hello-chk**

可用于：defaults、listen、backend

使用SSLv3客户端hello健康检测来测试服务器。

------

**option tcpka**

可用于：defaults、frontend、listen、backend

启用或禁用在连接的两端发送TCP保活数据包的发送。

------

**option tcplog**

可用于：defaults、frontend、listen、backend

启用高级的日志记录TCP连接，带有会话状态和定时器。

-----

**option transparent**

**no option transparent**

可用于：defaults、listen、backend

启用客户端透明代理。

为了向3层协议负载均衡器提供7层协议持久性而引入该选项。想法是利用操作系统的能力将一个刚进入的到远程地址的连接重定向到本地进程（这里是HAProxy），并让该进程知道原本请求的是什么地址。

When this option is used, sessions without cookies will be forwarded to the original destination IP address of the incoming request (which should match that of another equipment), while requests with cookies will still be forwarded to the appropriate server.

------

**``reqadd <string> [{if | unless} <cond>]``**

可用于：frontend、listen、backend

在HTTP请求的尾部添加一个头部字段。

*参数* :

```
<string> 需要添加的完整的头部字段行。任何空格或常用定界符都必须使用反斜杠（‘\’）进行转义。

<cond> 可选，ACL构建的匹配条件。
```

Header transformations only apply to traffic which passes through HAProxy, and not to traffic generated by HAProxy, such as health-checks or error responses.

*示例* ：

```
# Add "X-Proto：SSL" to requests coming via port 81
acl is-ssl  dst_port    81
reqadd  X-Proto:\ SSL   if is-ssl
```

------

**``retries <value>``**

可用于：defaults、listen、backend

设置对某个服务器连接失败后重试的次数。

When "option redispatch" is set, the last retry may be performed on another server even if a cookie references a different server.

------

**``stats admin { if | unless } <cond>``**

可用于：listen、backend

启用统计管理级别，如果/除非达到某个条件。

管理级别允许通过web界面启用/禁用后端服务器。默认情况下，统计页面考虑到安全问题是只读的。

Currently, the POST request is limited to the buffer size minus the reserved buffer space, which means that if the list of servers is too long, the request won't be processed. It is recommended to alter few servers at a time.

*示例* ：

```
# statistics admin level only for localhost
backend stats_localhost
    stats enable
    stats admin if LOCALHOST
```

```
# statistics admin level always enabled because of the authentication
backend stats_auth
    stats enable
    stats auth admin:AdMiN123
    stats admin if TRUE
```

```
# statistics admin level depends on the authenticated user
userlist stats-auth
    group admin users admin
    user admin insecure-password AdMiN123
    group readonly users haproxy
    user haproxy insecure-password haproxy

backend stats_auth
    stats enable
    acl AUTH        http_auth(stats-auth)
    acl AUTH_ADMIN  http_auth_group(stats-auth) admin
    stats http-request auth unless AUTH
    stats admin if AUTH_ADMIN
```

------

**``stats auth <user>:<passwd>``**

可用于：defaults、listen、backend

启用带身份认证的统计信息页面，并赋予某账户访问权限。

*示例* ：

```
# public access (limited to this backend only)
backend public_www
    server srv1 192.168.0.1:80
    stats enable
    stats hide-version
    stats scope   .
    stats uri     /admin?stats
    stats realm   Haproxy\ Statistics
    stats auth    admin1:AdMiN123
    stats auth    admin2:AdMiN321

# internal monitoring access (unlimited)
backend private_monitoring
    stats enable
    stats uri     /admin?stats
    stats refresh 5s
```

------

**stats enable**

可用于：defaults、listen、backend

以默认设置启用统计报告。

This statement enables statistics reporting with default settings defined
at build time. Unless stated otherwise, these settings are used :

- stats uri   : /haproxy?stats
- stats realm : "HAProxy Statistics"
- stats auth  : no authentication
- stats scope : no restriction

------

**stats hide-version**

可用于：defaults、listen、backend

启用统计并隐藏HAProxy版本报告。

------

**``stats http-request { allow | deny | auth [realm <realm>] } [ { if | unless } <condition> ]``**

可用于：listen、backend

统计（页面）访问控制。

As "http-request", these set of options allow to fine control access to statistics. Each option may be followed by if/unless and acl. First option with matched condition (or option without condition) is final. For "deny" a 403 error will be returned, for "allow" normal processing is performed, for "auth" a 401/407 error code is returned so the client should be asked to enter a username and password.

There is no fixed limit to the number of http-request statements per instance.

------

**``stats scope { <name> | "." }``**

可用于：defaults、listen、backend

启用统计并限制访问范围。

*参数* ：

```
<name> 被报告的一个listen、frontend或backend部分的名称。特殊名称“.”指派该语句出现的部分。
```

When this statement is specified, only the sections enumerated with this statement will appear in the report. All other ones will be hidden. This statement may appear as many times as needed if multiple sections need to be reported. Please note that the name checking is performed as simple string comparisons, and that it is never checked that a give section name really exists.

------

**``stats show-desc [ <description> ]``**

可用于：defaults、listen、backend

在统计页面上显示一段描述语句。

------

**stats show-legends**

Enable reporting additional informations on the statistics page :

- cap: capabilities (proxy)
- mode: one of tcp, http or health (proxy)
- id: SNMP ID (proxy, socket, server)
- IP (socket, server)
- cookie (backend, server)

------

**``tcp-request content accept [{if | unless} <condition>]``**

可用于：frontend、listen

如果/除非满足一个内容检查条件，则接受一个连接。

Note that the "if/unless" condition is optional. If no condition is set on the action, it is simply performed unconditionally.

If no "tcp-request content" rules are matched, the default action already is "accept". Thus, this statement alone does not bring anything without another "reject" statement.

------

**``tcp-request content reject [{if | unless} <condition>]``**

------

**``timeout http-keep-alive <timeout>``**

可用于：defaults、frontend、listen、backend

设置等待出现一个新的HTTP请求的最大允许时间。

By default, the time to wait for a new request in case of keep-alive is set by "timeout http-request". However this is not always convenient because some people want very short keep-alive timeouts in order to release connections
faster, and others prefer to have larger ones but still have short timeouts once the request has started to present itself.

The "http-keep-alive" timeout covers these needs. It will define how long to wait for a new HTTP request to start coming after a response was sent. Once the first byte of request has been seen, the "http-request" timeout is used
to wait for the complete request to come. Note that empty lines prior to a new request do not refresh the timeout and are not counted as a new request.

There is also another difference between the two timeouts : when a connection expires during timeout http-keep-alive, no error is returned, the connection just closes. If the connection expires in "http-request" while waiting for a
connection to complete, a HTTP 408 error is returned.

In general it is optimal to set this value to a few tens to hundreds of milliseconds, to allow users to fetch all objects of a page at once but without waiting for further clicks. Also, if set to a very small value (eg: 1 millisecond) it will probably only accept pipelined requests but not the non-pipelined ones. It may be a nice trade-off for very large sites running with tens to hundreds of thousands of clients.

------

**``timeout http-request <timeout>``**

可用于：defaults、frontend、listen、backend

设置等待一个完整的HTTP请求的最大允许时间。

------

**``use_backend <backend> if <condition>``**

**``use_backend <backend> unless <condition>``**

可用于：frontend、listen

若满足某个基于ACL的条件，则切换到某个特定的backend。


### 5. server和default-server选项

```
server <name> <address>[:port] [settings ...]
    
default-server [settings ...]
```

**backup**

当“backup”出现在server行中，则该服务器在负载均衡中仅当所有其他非备份服务器都不可用时才会被使用。

------

**``weight <weight>``**

“weight”参数用于调整服务器相对于其他服务器的权重。所有服务器根据它们的权重比上所有权重之和得到一个负载比例，因此权重越高，负载越高。默认权重是1，权重最大值是256。零值意味着该服务器将不参与负载均衡，但仍接受持久连接。如果该参数用于根据服务器的能力分布负载，推荐开始设置为一个可以伸缩的值，例如10到100之间的某个值，为之后向上向下调整留出足够的空间。


### 7. 使用访问控制列表（ACL）和模式抽取

访问控制列表（ACL）的使用为执行内容转换以及基于从请求、响应或任何环境状态中抽取的内容作决策提供了一种灵活的解决方案。

原理很简单：

- 使用一些值来定义测试条件

- 仅当一组测试通过才执行动作

动作通常包括阻塞请求，或选择一个backend。

为了定义一个测试，需使用“acl”关键词。语法为：

``acl <aclname> <criterion> [flags] [operator] <value> ...``

*示例* ：

```
# Match any negative Content-Length header
acl negative-length hdr_val(content-length) lt 0
```

```
# Match SSL versions between 3.0 and 3.1(inclusive)
acl sslv3 req_ssl_ver 3:3.1
```

```
# To select a different backend for requests to static contents 
# on the "www" site and to every request on the "img", "video",
# "download" and "ftp" hosts
acl url_static  path_beg             /static /imags/ /img /css
acl url_static  path_end             .gif .png .jpg .css .js
acl host_www    hdr_beg(host) -i www
acl host_static hdr_beg(host) -i img. video. download. ftp.

# now use backend "static" for all static-only hosts, and for static urls
# of host "www". Use backend "www" for the rest.
use_backend     static  if host_static or host_www url_static
use_backend     www     if host_www
```
