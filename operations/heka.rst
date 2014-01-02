heka
============

- `Intro to Heka <https://people.mozilla.org/~rmiller/heka-intro-2013-03>`_
- `Application monitoring with Heka and statsd <http://slides.seld.be/?file=2013-12-13+Application+monitoring+with+Heka+and+statsd.html#1>`_

------


Heka文档阅读笔记
--------------------

Heka simplifies the collection, analysis, and transportation(传输) of data across your cluster/server.

Features
^^^^^^^^^^^

**Multiple Data Sources** : Data can be sent directly in via client libraries or using plug-ins that can read logfile, handle statsd UDP input, or other custom data sources.

**Unified Daemon** : Heka is capable of routing messages like logstash, aggregating counters and submitting them to other sources like statsd, and transporting log messages from various nodes like syslog.

**Plugin Architecture** : In case existing plug-ins are insufficient, additional plug-ins may be written in Go or Lua (to run in the embedded Lua sandbox). Plug-ins can be utilized for input, decoding unstructured data, filtering, or as outputs.

**Performance** : Written in Go, heka utilizes light-weight goroutines for fast, efficient message routing, and parallel plug-in operation that can utilize multi-core systems to move hundreds of thousands of messages per second while using a very modest amount of memory.

**Usability** : A single static binary with a library module is all that’s needed to run heka locally, making it easy to develop with and deploy.

Architecture of Heka
^^^^^^^^^^^^^^^^^^^^^^^^

.. image:: http://heka-docs.readthedocs.org/en/latest/_images/graphviz-d4b4e7ef1c8c6cac271545a345ea79b68897e38d.svg

**Client Libraries**

Client libraries for Heka are capable of collecting:

- Logging output
- Custom counters + timers
- Exceptions

The client libraries send data to *hekad* for routing.

**Hekad Application**

The hekad application daemon is distinguished by the role its configured for. Depending on the configuration, hekad’s role may be:

- Agent
- Aggregator
- Router
- Analyzer

hekad may be configured to act in one or more roles as necessary for the environment its deployed in. Regardless of the configured role, internally hekad works the same, moving messages through a series of filters which then are sent to outputs.

.. toctree::
    :maxdepth: 2

    heka/hekad