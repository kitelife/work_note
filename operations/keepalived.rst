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
