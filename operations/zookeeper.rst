Zookeeper
==============

ZooKeeper is a high-performance coordination service for distributed applications. It exposes common services - such as naming, configuration management, synchronization, and group services - in a simple interface so you don't have to write them from scratch. You can use it off-the-shelf to implement consensus, group management, leader election, and presence protocols. And you can build on it for your own, specific needs.

------

Zookeeper Overview
----------------------

Zookeeper: A Distributed Coordination Service for Distributed Applications
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Zookeeper exposes a simple set of primitives that distributed applications can build upon to implement higher level services for synchronization, configuration maintenance, and groups and naming. It is designed to be easy to program to, and uses a data model styled after the familiar directory tree structure of file systems. It runs in Java and has bindings for both Java and C.

Coordination services are notoriously hard to get right. They are especially prone to errors such as race conditions and deadlock. The motivation behind ZooKeeper is to relieve distributed applications the responsibility of implementing coordination services from scratch.

**Design Goals**

- *Zookeeper is simple.* ZooKeeper allows distributed processes to coordinate with each other through a shared hierarchal namespace which is organized similarly to a standard file system. The name space consists of data registers - called znodes, in ZooKeeper parlance - and these are similar to files and directories. Unlike a typical file system, which is designed for storage, ZooKeeper data is kept in-memory, which means ZooKeeper can achieve high throughput and low latency numbers. The ZooKeeper implementation puts a premium on high performance, highly available, strictly ordered access. The performance aspects of ZooKeeper means it can be used in large, distributed systems. The reliability aspects keep it from being a single point of failure. The strict ordering means that sophisticated synchronization primitives can be implemented at the client.

- *Zookeeper is replicated.*

  .. image:: http://zookeeper.apache.org/doc/trunk/images/zkservice.jpg

  The servers that make up the ZooKeeper service must all know about each other. They maintain an in-memory image of state, along with a transaction logs and snapshots in a persistent store. As long as a majority of the servers are available, the ZooKeeper service will be available.

  Clients connect to a single ZooKeeper server. The client maintains a TCP connection through which it sends requests, gets responses, gets watch events, and sends heart beats. If the TCP connection to the server breaks, the client will connect to a different server.

- *Zookeeper is ordered.* ZooKeeper stamps each update with a number that reflects the order of all ZooKeeper transactions. Subsequent operations can use the order to implement higher-level abstractions, such as synchronization primitives.

- *Zookeeper is fast.* It is especially fast in "read-dominant" workloads. ZooKeeper applications run on thousands of machines, and it performs best where reads are more common than writes, at ratios of around 10:1.

**Data model and the hierarchical namespace**

The name space provided by ZooKeeper is much like that of a standard file system. A name is a sequence of path elements separated by a slash (/). Every node in ZooKeeper's name space is identified by a path.

.. image:: http://zookeeper.apache.org/doc/trunk/images/zknamespace.jpg

**Guarantees**

Zookeeper is very fast and very simple. Since its goal, though, is to be a basis for the construction of more complicated services, such as synchronization, it provides a set of guarantees. These are:

- Sequential Consistency - Updates from a client will be applied in the order that they were sent.
- Atomicity - Updates either succeed or fail. No partial results.
- Single System Image - A client will see the same view of the service regardless of the server that it connects to.
- Reliability - Once an update has been applied, it will persist from that time forward until a client overwrites the update.
- Timeliness - The clients view of the system is guaranteed to be up-to-date within a certain time bound.

**Simple API**

One of the design goals of ZooKeeper is provide a very simple programming interface. As a result, it supports only these operations:

- create: creates a node at a location in the tree
- delete: deletes a node
- exists: tests if a node exists at a location
- get data: reads the data from a node
- set data: writes data to a node
- get children: retrieves a list of children of a node
- sync: waits for data to be propagated

**Implementation**

.. image:: http://zookeeper.apache.org/doc/trunk/images/zkcomponents.jpg

The image above shows the high-level components of the ZooKeeper service. With the exception of(除了...之外) the request processor, each of the servers that make up the ZooKeeper service replicates its own copy of each of components.

The replicated database is an in-memory database containing the entire data tree. Updates are logged to disk for recoverability, and writes are serialized to disk before they are applied to the in-memory database.

Every ZooKeeper server services clients. Clients connect to exactly one server to submit irequests. Read requests are serviced from the local replica of each server database. Requests that change the state of the service, write requests, are processed by an agreement protocol.

As part of the agreement protocol all write requests from clients are forwarded to a single server, called the leader. The rest of the ZooKeeper servers, called followers, receive message proposals from the leader and agree upon message delivery. The messaging layer takes care of replacing leaders on failures and syncing followers with leaders.

ZooKeeper uses a custom atomic messaging protocol. Since the messaging layer is atomic, ZooKeeper can guarantee that the local replicas never diverge. When the leader receives a write request, it calculates what the state of the system is when the write is to be applied and transforms this into a transaction that captures this new state.

------

- `Zookeeper研究和应用 <http://www.searchtb.com/2011/01/zookeeper-research.html>`_
- `Zookeeper工作原理 <http://stblog.baidu-tech.com/?p=1164>`_
- `Building a distributed concurrent queue with Apache ZooKeeper <http://blog.cloudera.com/blog/2009/05/building-a-distributed-concurrent-queue-with-apache-zookeeper/>`_
- `ZooKeeper典型应用场景一览 <http://jm-blog.aliapp.com/?p=1232>`_
- `ZooKeeper管理员指南——部署与管理ZooKeeper <http://jm-blog.aliapp.com/?p=2318>`_
