Keepalived
================

官网：http://www.keepalived.org

主要功能是实现真实机的故障隔离及负载均衡器间的失败切换（failover）。负载均衡器之间的失败切换，是通过VRRPv2（Virtual Router Redundancy Protocol）stack实现的，VRRP当初被设计出来的目的就是为了解决静态路由器的单点故障问题。

iptables的启用是不会影响Keepalived的运行的，但为了更好的性能，通常会将整套系统内所有主机的iptables都停用。

Keepalived产生的VIP就是整个系统对外的IP，如果最外端的防火墙采用的是路由模式，那就映射此内网IP为公网IP。

tlinux上源码编译安装Keepalived
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Keepalived依赖于OpenSSL、popt，所以先编译安装这两者：

- OpenSSL
  
  - ``wget http://www.openssl.org/source/openssl-1.0.1e.tar.gz`` 
  - ``tar -xvf openssl-1.0.1e.tar.gz``
  - ``cd openssl-1.0.1e && ./config --prefix=/usr/local/openssl && make && make install``

- popt

  - ``wget http://rpm5.org/files/popt/popt-1.14.tar.gz``
  - ``tar -xvf popt-1.14.tar.gz``
  - ``cd popt-1.14 && ./configure && make && make install``

然后安装Keepalived：

- ``wget http://www.keepalived.org/software/keepalived-1.2.7.tar.gz``
- ``tar -xvf keepalived-1.2.7.tar.gz``
- ``cd keepalived-1.2.7 && ./configure --prefix=/usr/local/keepalived && make && make install``

一般情况下，这样就能成功安装了，但tlinux上这样并不行，configure的过程应为：

- ``LDFLAGS=-ldl CFLAGS="-I/usr/local/openssl/include  -L/usr/local/openssl/lib  -L/usr/local/lib" ./configure --prefix=/usr/local/keepalived``

- ``export LIBRARY_PATH=$LIBRARY_PATH:/usr/local/openssl/lib``

然后 ``make && make install`` 就可以了。

VRRP协议
^^^^^^^^^^^

VRRP协议是为了消除在静态缺省路由环境下的缺省路由器单点故障引起的网络失效而设计的主备模式的协议，使得在发生故障而进行设备功能切换时可以不影响内外数据通信，不需要再修改内部网络的网络参数。

VRRP协议将两台或多台路由器设备虚拟成一个设备，对外提供虚拟路由器IP（一个或多个），而在路由器内部，如果实际拥有这个对外IP的路由器工作正常的话就是MASTER，或者通常算法选举产生，MASTER实现针对虚拟路由器IP的各种网络功能，如ARP请求、ICMP、以及数据的转发等；其他设备不拥有该IP，状态为BACKUP，除了接收MASTER的VRRP状态通告信息外，不执行对外的网络功能。当MASTER主机失效时，BACKUP将接管原先MASTER的网络功能。

Keepalived原理
^^^^^^^^^^^^^^^^^^

Keepalived是模块化设计的，不同模块负责不同的功能，下面是Keepalived的组件：

**core check vrrp libipfwc libipvs-2.4 libipvs-2.6**

- core：是keepalived的核心，负责主进程的启动和维护，全局配置文件的加载解析等
- check：负责healthchecker（健康检查），包括各种健康检查方式，以及对应的配置的解析包括LVS的配置解析
- vrrp：VRRPD子进程，VRRPD子进程就是来实现VRRP协议的
- libipfwc：iptables（ipchains）库，配置LVS会用到（注：现在已经没有该模块了）
- libipvs*：配置LVS会用到

*注：keepalived和LVS完全是两码事，只不过它们各负其责相互配合而已*



参考资料
^^^^^^^^^^

- `VRRP协议介绍 <http://bbs.ywlm.net/thread-790-1-1.html>`_
- `Keepalived原理与实战精讲 <http://bbs.ywlm.net/thread-845-1-1.html>`_
