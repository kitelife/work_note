PostgreSQL
===============

读文笔记：On PostgreSQL. Interview with Tom Kincaid
-----------------------------------------------------

*"Application designer need to start by thinking about what level of data integrity they need, rather than what they want, and then design their technology stack around that reality. Everyone would like a database that guarantees perfect availability, perfect consistency, instananeous response times, and infinite throughput, but it's not possible to create a product with all of those properties" *

**Q2. How does PostgreSQL compare with MariaDB and MySQL5.6?**

PostgreSQL has traditionally had a stronger focus on data integrity and compliance with SQL standard. MySQL has traditionally been focused on raw performance for simple queries, and typical benchmark is the number of read queries per second that the database engine can carry out, while PostgreSQL tends to focus more on having a sophisticated query optimizer that can efficiently handle more complex queries, sometimes at the expense of speed on simpler queries. And, for a long time, MySQL had a big lead over PostgreSQL in the area of replication technologies, which discouraged many users from choosing PostgreSQL.

Over time, these differences have diminished. PostgreSQL´s replication options have expanded dramatically in the last three releases, and its performance on simple queries has greatly improved in the most recent release (9.2). On the other hand, MySQL and MariaDB have both done significant recent work on their query optimizers. So each product is learning from the strengths of the other.

.. seealso:: `On PostgreSQL. Interview with Tom Kincaid. <http://www.odbms.org/blog/2013/05/on-postgresql-interview-with-tom-kincaid/>`_