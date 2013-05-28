High Scalability 读文笔记
===============================

Tumblr架构设计
-----------------

Tumblr最开始是非常典型的LAMP应用。目前正在向分布式服务模型演进，该模型基于Scala、HBase、Redis、Kafka、Finagle。

开始采用新的NoSQL存储方案如HBase和Redis。但大量数据仍然存储在大量分区的MySQL架构中，并没有用HBase代替MySQL。

Gearman用于会长期运行无需人工干预的工作。

22台Redis服务器，每台的都有8-32个实例，因此线上同时使用了100多个Redis实例：

- Redis主要用于Dashboard通知的后端存储。
- 所谓通知就是指某个用户like了某篇文章这样的事件。通知会在用户的Dashboard中显示，告诉他其他用户对其内容做了哪些操作。
- 高写入率使MySQL无法应对。
- 通知转瞬即逝，所以即使遗漏也不会有严重问题，因此Redis是这一场景的合适选择。
- 这也给了开发团队了解Redis的机会。
- 使用中完全没有发现Redis有任何问题，社区也非常棒。
- 短地址生成程序使用Redis作为一级Cache，HBase作为永久存储。
- Dashboard的二级索引是以Redis为基础开发的。
- Redis还用作Gearman的持久存储层，使用Finagle开发的memcache代理。
- 正在缓慢地从memcache转向Redis。希望最终只用一个cache服务。性能上Redis与memcache相当。

**软件部署中的问题**

- 开发了一套rsync脚本，可以随处部署PHP应用程序。一旦机器的数量超过200台，系统便开始出现问题，部署花费了很长时间才完成，机器处于部署进程中的各种状态。
- 接下来，使用Capistrano（一个开源工具，可以在多台服务器上运行脚本）在服务堆栈中构建部署进程（开发、分期、生产）。在几十台机器上部署可以正常工作，但当通过SSH部署到数百台服务器时，再次失败。

**经验及教训**

- 自动化无处不在
- Redis 总能带给人惊喜
- 基于 Scala 语言的应用执行效率是出色的
- 不顾用在他们发展经历中没经历过技术挑战的人，聘用有技术实力的人是因为他们能适合你的团队以及工作
- 选择正确的软件集合将会帮助你找到你需要的人
- 多与同行交流，可以接触一些领域中经验丰富的人，例如与在 Facebook、Twitter、LinkedIn 的工程师多交流，从他们身上可以学到很多
- 对技术要循序渐进，在正式投入使用之前他们煞费苦心的学习 HBase 和 Redis。同时在试点项目中使用或将其控制在有限损害范围之内

.. seealso:: `中文翻译 <http://www.oschina.net/translate/the-tumblr-architecture-yahoo-bought-for-a-cool-billion-doll>`_ , `原文 <http://highscalability.com/blog/2013/5/20/the-tumblr-architecture-yahoo-bought-for-a-cool-billion-doll.html>`_