HAProxy
=========

官网：http://haproxy.1wt.eu

HAProxy的syslog-ng日志配置
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

http://haproxy.1wt.eu/download/contrib/ 中有两个文件 **syslog-ng.conf** 和 **haproxy.logrotate**

- 将syslog-ng.conf文件内容添加到/etc/syslog-ng/syslog-ng.conf中
- 在/etc/logrotate.d目录下新建文件haproxy，将haproxy.logrotate文件内容写入其中。

HAProxy重启
^^^^^^^^^^^^^^^^

::

    /usr/local/haproxy/sbin/haproxy -f /usr/local/haproxy/conf/haproxy.cfg -st `cat /var/run/haproxy.pid`

HAProxy配置文件示例（来自 `HAProxy的安装和部署 <http://lam.iteye.com/blog/990796>`_ ）
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

::

    #########################################################################
    # HAProxy 配置文件
    #########################################################################
 
    global
        # 使用系统的syslog记录日志（通过udp，默认端口号为514）
        log 127.0.0.1 local0 # info [err warning info debug]
        chroot /home/user/haproxy
 
        #限制单个进程的最大连接数
        maxconn 65535
 
        # 让进程在后台运行，即作为守护进程运行，正式运行的时候开启，此处先禁止，等同于在命令行添加参数 -D
        # daemon
        # 指定作为守护进程运行的时候，要创建多少个进程，默认只创建一个，需要daemon开启模式
        # nbproc 1
 
        # 设置debug模式运行，与daemon模式只能互斥，等同于在命令行添加参数 -d
        # debug
        pidfile /home/user/haproxy/logs/haproxy.pid    # not work
 
    defaults
        # 在连接失败或断开的情况下，允许当前会话被重新分发
        option redispatch
        # 设置在一个服务器上链接失败后的重连次数
        retries 2
        # 设置服务器分配算法
        balance roundrobin
 
        # 不记录空连接
        option dontlognull
 
        # 设置等待连接到服务器成功的最大时间
        timeout connect 5000ms
        # 设置客户端的最大超时时间
        timeout client 1800000ms
        # 设置服务器端的最大超时时间
        timeout server 1800000ms
 
        # Enable the sending of TCP keepalive packets on both sides, clients and servers
        # NOTE: 在服务器CPU强劲的情况下，最好不要开启保活，这样可减少资源消耗
        #option tcpka
 
    ##############################统计页面配置##################################
 
    listen admin_stat
        # 监听端口
        bind *:8011
        # http的7层模式
        mode http
        option httplog
        log global
        # 统计页面自动刷新时间
        stats refresh 30s
        # 统计页面URL
        stats uri /admin?stats
        # 统计页面密码框上提示文本
        stats realm Haproxy\ Statistics
        # 统计页面用户名和密码设置
        stats auth admin:admin
        # 隐藏统计页面上HAProxy的版本信息
        stats hide-version
 
    ###########################TCP连接的监听配置################################
 
    listen  tcp-in
        bind *:2211
        mode tcp
        # 日志记录选项
        option tcplog
        log global
 
        # 后台服务器
        # weight  -- 调节服务器的负重
        # check -- 允许对该服务器进行健康检查
        # inter  -- 设置连续的两次健康检查之间的时间，单位为毫秒(ms)，默认值 2000(ms)
        # rise  -- 指定多少次连续成功的健康检查后，即可认定该服务器处于可操作状态，默认值 2
        # fall  -- 指定多少次不成功的健康检查后，认为服务器为当掉状态，默认值 3
        # maxconn  -- 指定可被发送到该服务器的最大并发连接数
        server localhost 0.0.0.0:2233 weight 3 check inter 2000 rise 2 fall 3
        server 192.168.1.100 192.168.1.100:2233 weight 3 check inter 2000 rise 2 fall 3
        server 192.168.1.101 192.168.1.101:2233 weight 3 check inter 2000 rise 2 fall 3
 
    #########################HTTP连接的监听配置################################
 
    listen  http-in
        bind *:2212
        mode http
        option httplog
        log global
 
        # 设置健康检查模式
        #option httpchk OPTIONS * HTTP/1.1\r\nHost:\ www
        #option smtpchk
 
        # 后台服务器
        server localhost 0.0.0.0:2234 weight 3 check inter 2000 rise 2 fall 3
        server 192.168.1.100 192.168.1.100:2234 weight 3 check inter 2000 rise 2 fall 3
        server 192.168.1.101 192.168.1.101:2234 weight 3 check inter 2000 rise 2 fall 3
    

参考材料
============
- `基于Keepalived+Haproxy搭建四层负载均衡器 <http://blog.liuts.com/post/223/>`_
- `HAProxy的安装和部署 <http://lam.iteye.com/blog/990796>`_
