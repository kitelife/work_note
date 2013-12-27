NTP
===

`鸟哥的私房菜 - NTP服务器的安装与设定 <http://vbird.dic.ksu.edu.tw/linux_server/0440ntp_2.php>`_

安装NTP服务，其实就是将NTP安装好后，规定一部上层NTP服务器来同步化你的时间即可！如果你只是想要进行你自己单部主机的时间同步化，别架设NTP，直接使用NTP客户端软件即可。

与时间及 NTP 服务器设定相关的配置文件与重要数据文件有如下几个：

-   **/etc/ntp.conf** : NTP服务器的主要配置文件，也是唯一的一个；
-   **/usr/share/zoneinfo/** : 由tzdata提供，为各时区的时间格式对应文件；
-   **/etc/sysconfig/clock** : 设定时区与是否使用 UTC 时间钟的配置文件。 每次开机后Linux会自动的读取这个文件来设定自己系统所默认要显示的时间；
-   **/etc/localtime** : 这个文件就是“本地端的时间配置文件”啦！刚刚那个clock文件里面规定了使用的时间配置文件(ZONE)为/usr/share/zoneinfo/Asia/Taipei，所以说这就是本地端的时间了，此时Linux系统就会将Taipei那个文件复制一份成为/etc/localtime ，所以未来我们的时间显示就会以Taipei那个时间配置文件为准。

常用于时间服务器与修改时间的命令，主要有如下几个：

-   **/bin/date** : 用于 Linux 时间 (软件时钟) 的修改与显示的指令；
-   **/sbin/hwclock** : 用于 BIOS 时钟 (硬件时钟) 的修改与显示的指令。 这是一个root才能执行的指令，因为Linux系统上面BIOS时间与Linux系统时间是分开的，所以使用date这个指令调整了时间之后，还需要使用hwclock才能将修改过后的时间写入 BIOS 当中！
-   **/usr/sbin/ntpd** : 主要提供 NTP 服务的程序，配置文件为 /etc/ntp.conf
-   **/usr/sbin/ntpdate** : 用于客户端的时间校正，如果你没有要启用 NTP 而仅想要使用NTP Client功能的话，那么只会用到这个指令而已啦！

------

配置文件ntp.conf
^^^^^^^^^^^^^^^^^^^^^^

**利用restrict来管理权限控制**

::

    restrict [你的IP] mask [netmask_IP] [parameter]


其中parameter的值有如下几种：

- ignore：拒绝所有类型的NTP联机
- nomodify：客户端不能使用ntpc与ntpq这两个程序来修改服务器的时间参数，但客户端仍可透过这部主机来进行网络校时
- noquery：客户端不能使用ntpq、ntpc等命令来查询时间服务器，等于不提供NTP的网络校时
- notrap：不提供trap这个远程事件记录（remote event logging）的功能
- notrust：拒绝没有认证的客户端

如果你没有在parameter的地方加上任何参数的话，这表示 **该 IP 或网段不受任何限制** 的意思。一般来说，我们可以先关闭NTP的权限，然后再一个一个的启用允许登入的网段。

**利用server设定上层NTP服务器**

::

    server [IP or hostname] [prefer]


在server后可以接IP或主机名，鸟哥个人比较喜欢使用IP来设定。至于那个perfer表示“优先使用”的服务器。

**以driftfile记录时间差异**

::

    driftfile [可以被ntpd写入的目录与文件]


因为默认的NTP Server本身的时间计算是依据BIOS的芯片震荡周期频率来计算的，但是这个数值与上层Time Server不见得会一致，所以NTP这个daemon(ntpd)会自动的去计算我们自己主机的频率与上层Time server的频率，并且将两个频率的误差记录下来，记录下来的内容就是在driftfile后面接的绝对路径文件当中。关于该文件的文件名，需注意：

- driftfile后面接的文件需要使用完整路径的文件名；
- 该文件不能是链接文件
- 该文件需要设置成ntpd这个daemon可以写入的权限
- 该文件锁记录的数值单位为百万分直译秒（ppm）

driftfile后面接的文件会被ntpd自动更新，所以它的权限一定要能够让ntpd写入才行。

**keys[key_file]**

除了以restrict来限制客户端的联机之外，也可以通过密钥来给客户端认证。

客户端的时间更新方式
-------------------------

Linux系统时间与硬件时间
^^^^^^^^^^^^^^^^^^^^^^^^^

在Linux操作系统中有两个时间：

- 一个是BIOS记录的实际时间，这也是硬件所记录的
- 一个是Linux自己的系统时间，由1970/01/01开始记录的时间参数

当Linux开机后，它会主动的读出BIOS所记录的时间，然后开始用自己的方式计算时间。当我们使用date之类的指令来查询或设置时间时，该时间指的是Linux的时间而已，并没有改变BIOS内所记录的时间。除非你使用hwclock来写入或读出BIOS的时间。

由于BIOS会记录时间而且会持续计时，因为我们关机后再开机时，会发现BIOS时间一直在累加。为了维持BIOS所记录的信息，主板上面的电池就很重要，它可以让BIOS在关机的时候还记录的记录硬件信息以及计时。所以如果你发现开机后整个BIOS时间恢复成为系统出厂值，很可能就是主板上的电池没电了。如果你将BIOS断电处理，时间也会恢复成为系统出厂值。

由于每个BIOS内部的时间计算器会有误差，因为与Linux时间就会产生差异，这个差异在时间长了之后，会越来越大。所以需要进行网络校时。

Linux系统时区与手动校时工作
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Linux的时区文件是/etc/localtime，这是一个时间格式的文件，而不是ASCII类型的文件。至于所有的Time Zone则放置在/usr/share/Zoneinfo目录下。注意：

- 当/etc/localtime存在时，系统的时区以该文件代表的时区来显示
- 当/etc/localtime不存在时，系统的时区主要以GMT（或UTC）为准

所以，如果你想要变更Linux系统的时区，那么只要在/usr/share/Zoneinfo里找到你需要的时区文件，将它另存为/etc/localtime，就可以顺利地更新时区设置了。另外，同时建议修正一下/etc/sysconfig/clock这个文件里的ZONE设置值。以中国台湾地区的Time Zone为例，在/etc/sysconfig/clock这个文件中应该是“ZONE="Asia/Taipei"”，这表示我们的时区文件为/usr/share/Zoneinfo/Asia/Taipei，请对应修改成你想要的时区。

可以通过date指令来手动修正目前主机的时间，不过，date指令仅修正Linux时间而已，还需要通过hwclock指令（ ``hwclock -w`` ）来更新BIOS时间。

Linux的网络校时
^^^^^^^^^^^^^^^^^

在Linux的环境中可利用NTP的客户端程序，即ntpdate程序进行时间同步。因为NTP服务器本来就会与上层时间服务器进行时间同步，所以在默认情况下，NTP服务器不可以使用ntpdate，也就是说ntpdate与ntpd不能同时启用。

.. image:: https://raw.github.com/youngsterxyf/work_note/master/operations/pics/network_time_asyc.png
