读文笔记：Quora’s Technology Examined
========================================

The Search-Box
^^^^^^^^^^^^^^^^^

Previously, they did use an open source search server, called Sphinx. It supports the features they are using above, but they have since moved from this due to real-time constraints. Their new solution is built in-house and allows them better prefix indexing and control over the matching algorithms. They built this in Python.

**Speedy Queries**

Queries are sent over AJAX as a GET request. Responses come back as JSON with the rendered HTML embedded inside the JSON. Rendering of the results on the server-side, as opposed to rendering them in JavaScript, seems to be due to the need to highlight matching words in the text. This is sometimes too complex for JavaScript.

Quora uses persistent connections. A HTTP connection is established with the server when you start tying the search query. This connection is kept open and futher requests are made on this same open connection. The connection will terminate (times-out) if not used for 60 seconds. If a connection times-out then a new connection is established when tying begins.

Webnode2 And LiveNode
^^^^^^^^^^^^^^^^^^^^^^^^^^

Webnode2 and LiveNode are some of Quora's internal systems, which were built for managing the content.

Webnode2 generates HTML, CSS and JavaScript and is tightly coupled with LiveNode, which is responsible for managing the display of the content on the webpage.

One weakness of them is that it is tricky for LiveNode to keep track of what is happening within the browser as it pushes changes from the server. If users A and B are viewing the same question then ones interactions will affect the other. For instance, if user A up-votes an answer then that answer will be promoted and will visibly move up the page. This display change will be pushed over AJAX to user B's browser. Any prior browser-side change that user B made, such as expanding a comments section, might be lost.

LiveNode is written in Python, C++, and JavaScript. jQuery and Cython is also used.


Amazon Web Services
^^^^^^^^^^^^^^^^^^^^^^^

Amazon EC2 and S3 is used for their hosting.

**Ubuntu Linux**

**Static Content**

Using Amazon's distributed content delivery network, Cloudfront.（注：貌似现在Quora是用自己的CDN了，如http://qph.is.quoracdn.net/main-thumb-t-2803-50-pABO3faE4RywN4mCnyw8pMthjzRjmyh3.jpeg）

HAProxy Load-Balancing
^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Quora uses HAProxy at the front-line, which load-balances onto the distributed Nginx server behind them.


Nginx
^^^^^^^^^^^^^^

Behind the load-balancer, Nginx is used as a reverse-proxy server onto the web-servers.

.. seealso:: `Using Nginx As Reverse-Proxy Server On High-Loaded Sites <http://kovyrin.net/2006/05/18/nginx-as-reverse-proxy/>`_

Pylons And Paste
^^^^^^^^^^^^^^^^^^^^

Pylons, a lightweight web framework, is used as their main web-server behind Nginx. The use the default `Pylons + Paste stack <http://spacepants.org/blog/pylons-paste-stack>`_

Python
^^^^^^^^^^^^^

Coming from Facebook, it was a good bet that Charlie and Adam would choose PHP for their development language. As Adam points out, "Facebook is stuck on that for legacy reasons, not because it is the best choice right now". **From this experience they knew that choosing technologies, especially programming languages, for the long-run was very important. Python won over Java because it is more expressive and quicker to write code than Java. Scala was too new. Adam mentions *speed and the lack of type-checking as drawbacks with Python* , but they both already knew the language reasonably well. Where Python lacks speed for performance critical backend components, they opt to write them in C++** . They saw Ruby as a close match to Python, but their experience with Python and lack of experience in Ruby, made Python the winner. Python 2.6, to be precise.

Additional benefits for using Python are the fact that data-structures that map well to JSON, code readability, there is a large collection of libraries and the availability of good debuggers and reloaders. Browser-server communication using JSON is major component of what Quora does, so this was an important factor.

PyPy, a project that aims to produce a flexible and fast Python implementation, was also mentioned as something that might give them a speed-boost.

Thrift
^^^^^^^^^^^^

Thrift is used for communications between backend systems. The Thrift service is written in C++.

Tornado
^^^^^^^^^^^^^^

The Tornado web framework is used for live updating. This is their Comet server, which handles **the large volumes of open connections used for long-polling** and pushed updates to the browsers.

Long Polling (Comet)
^^^^^^^^^^^^^^^^^^^^^^^^^

Quora does not display just static web pages. Each page will updates new content as questions, answers and comments are submitted by other others. As Adam D'Angelo points out, one of the best ways to do this currently is with "long polling". This is different to "polling".

Long polling, also known as `Comet <http://en.wikipedia.org/wiki/Comet_(programming)>`_ , puts the server in control, by making the client wait for responses.

The benefit to long-polling is that there is less back-and-forth between the client and server. The server is in control of the timing, so updates to the browser can be made within milliseconds. This makes it ideal for chat applications or applications that want really snappy updates for their users.

The down-side is that you are going have lots of open connections between the clients and your servers.

The good news is that there are technologies specifically designed for this. It costs very little to hold open connections in memory if you free up all the resources used for that connection. For instance, Nginx (Quora uses this for proxying requests) is a single-threaded event-based application and uses very little memory for each connection. Each Nginx process is actively dealing with only one connection at a time. This means it can scale to tens of thousands of concurrent connections.

.. seealso:: `How do you push messages back to a web-browser client through AJAX?  Is there any way to do this without having the client constantly polling the server for updates? <http://www.quora.com/How-do-you-push-messages-back-to-a-web-browser-client-through-AJAX-Is-there-any-way-to-do-this-without-having-the-client-constantly-polling-the-server-for-updates>`_

.. seealso:: `Browser和Server持续同步的几种方式（jQuery+tornado演示） <http://qinxuye.me/article/ways-to-continual-sync-browser-and-server/>`_ , `WebSocket实战 <http://ued.sina.com.cn/?p=900>`_

MySQL
^^^^^^^^^^^^

The basic advice is to only partition data if necessary, keep data on one machine if possible and use a hash of the primary key to partition larger datasets across multiple databases. Joins must be avoided.

.. seealso:: `How FriendFeed uses MySQL to store schema-less data <http://backchannel.org/blog/friendfeed-schemaless-mysql>`_ , `How does one evaluate if a database is efficient enough to not crash as it's put under increasing load? <http://www.quora.com/How-does-one-evaluate-if-a-database-is-efficient-enough-to-not-crash-as-its-put-under-increasing-load>`_

Memcached
^^^^^^^^^^^^^^^^

Memcached is used as a caching layer in front of MySQL.

Git
^^^^^^^^^^

JavaScript Placement
^^^^^^^^^^^^^^^^^^^^^^^^

To place JavaScript at the end of the page will give the feeling of a quicker loading page, since the browser has content to display before the JavaScript has be seen.

Charlie Cheever Follows "14 Rules for Faster-Loading Web Sites"
-------------------------------------------------------------------

.. seealso:: `14 Rules for Faster-Loading Web Sites <http://stevesouders.com/hpws/rules.php>`_

------

.. seealso:: `原文 <http://www.bigfastblog.com/quoras-technology-examined>`_
