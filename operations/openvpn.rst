OpenVPN
================

官网：http://openvpn.net/


如何让客户端流量全部走VPN？
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

见HOWTO文档的“Routing all client traffic (including web-traffic) through the VPN”部分。

主要是在VPN服务器端配置文件（如：/etc/openvpn/server.conf）添加如下一条指令：

::

    push "redirect-gateway def1"

如果VPN服务建立无线网络之上（所有客户端与服务器在同一个无线子网中），则还需添加一个 ``local`` 标记：

::

    push "redirect-gateway local def1"

该指令的作用其实是在VPN客户端连接到VPN服务器时，服务器将push指令的内容推送到客户端中，其内容会修改客户端本地的路由表。

另外，既然push指令是将其内容推送到客户端生效，并不是对服务器本身生效，那么可以将push指令的内容直接写在客户端配置文件中，如在OpenVPN客户端安装目录的config子目录下有个“.ovpn”后缀的文件，
在该文件中添加：

::

    redirect-gateway def1

换个角度，如果不想让客户端流量全部走VPN，则需确保OpenVPN服务器端配置文件中没有 ``push "redirect-gateway def1"`` ，并且客户端配置文件中也没有 ``redirect-gateway def1`` 。


参考资料
--------------

- `OpenVPN HOWTO <http://openvpn.net/index.php/open-source/documentation/howto.html>`_
