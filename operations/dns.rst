DNS/BIND
============

编译安装BIND9.9.3
^^^^^^^^^^^^^^^^^^^^^

1. 从 `http://www.isc.org/downloads/ <http://www.isc.org/downloads/>`_ 下载BIND9.9.3-P2版本

2. tar -xvf bind-9.9.3-P2.tar.gz && cd bind-9.9.3-P2 && ./configure --preifx=/usr/local/bind9.9.3 --with-openssl=/usr/local/ssl && make && make install

配置BIND（缓存DNS）
^^^^^^^^^^^^^^^^^^^^^

编译安装好BIND后，默认并没有BIND的配置文件，需自己添加/usr/local/bind9.9.3/etc/named.conf（参考 `http://www.centos.bz/2012/02/bind-cacheing-only-dns-deploy/ <http://www.centos.bz/2012/02/bind-cacheing-only-dns-deploy/>`_ ）

::

    options {
	    directory "/usr/local/bind9.9.3/etc/";
	    forward only;
	    forwarders {
		    8.8.8.8;
		    8.8.4.4;
	    };
	    allow-query {any;};
    };

可使用 `/usr/local/bind9.9.3/sbin/named-checkconf` 来检测配置文件语法。

运行BIND： /usr/local/bind9.9.3/sbin/named &

查看端口占用： netstat -tlnp ，DNS服务器占用UDP(TCP)协议的53号端口。

BIND还提供了工具rndc来管理DNS服务器，命令工具为 `/usr/local/bind9.9.3/sbin/rndc` ，但rndc也需要配置，默认也没有配置文件，可使用sbin下的命令 `rndc-confgen` 来生成：

::

    /usr/local/bind9.9.3/sbin/rndc-confgen -s 127.0.0.1 -r /dev/urandom > /usr/local/bind9.9.3/etc/rndc.conf

生成的配置内容为：

::

    # Start of rndc.conf
    key "rndc-key" {
	    algorithm hmac-md5;
	    secret "tahthw+V9UfRd0q8E63vPw==";
    };

    options {
	    default-key "rndc-key";
	    default-server 127.0.0.1;
	    default-port 953;
    };
    # End of rndc.conf

    # Use with the following in named.conf, adjusting the allow list as needed:
    # key "rndc-key" {
    # 	algorithm hmac-md5;
    # 	secret "tahthw+V9UfRd0q8E63vPw==";
    # };
    # 
    # controls {
    # 	inet 127.0.0.1 port 953
    # 		allow { 127.0.0.1; } keys { "rndc-key"; };
    # };
    # End of named.conf
    
根据注释部分的内容，应在 `named.conf` 的最后添加：

::

    key "rndc-key" {
	    algorithm hmac-md5;
	    secret "tahthw+V9UfRd0q8E63vPw==";
    };

    controls {
	    inet 127.0.0.1 port 953
		    allow { 127.0.0.1;} keys { "rndc-key"; };
    };
    
重启named，然后就可以使用 `rndc` 命令来管理DNS服务器了：

::

    /usr/local/bind9.9.3/sbin/rndc status
    /usr/local/bind9.9.3/sbin/rndc stats    # 这条命令会将结果放入/usr/local/bind9.9.3/etc/named.stats中
    /usr/local/bind9.9.3/sbin/rndc dumpdb   # 这条命令会将结果放入/usr/local/bind9.9.3/etc/named_dump.db中

更详细使用查看帮助 `/usr/local/bind9.9.3/sbin/rndc --help`

推荐资料
------------

- `鸟哥的私房菜---第十九章、主机名控制者：DNS服务器 <http://vbird.dic.ksu.edu.tw/linux_server/0350dns_1.php>`_
- `第二十二章 DNS服务器 <http://man.ddvip.com/os/freebsd_book_chs/ch22.htm>`_
- TCP/IP详解（卷一）第14章 DNS:域名系统
- `Wikipedia-Domain Name System <http://en.wikipedia.org/wiki/Domain_Name_System>`_
- `bind使用rndc <http://xjsunjie.blog.51cto.com/999372/379245>`_
