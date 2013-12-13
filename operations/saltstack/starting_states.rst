How Do I Use Salt States?
============================

The core of the Salt State system is the SLS, or **S**\ a\ **L**\ t\ **S** tate file. The SLS is a representation of the state in which a system should be in, and is set up to contain this data in a simple format. This is often called 
configuration management.

The Top File
-------------------

The example SLS files in the below section can be assigned to hosts using a file called **top.sls** .

The top file is used **to map what SLS modules get loaded onto what minions via the state system** . The top file creates a few general abstractions. First it maps what nodes should pull from which environments, 
next it defines which matches systems should draw from.

Environments
^^^^^^^^^^^^^^^^^

The environments in the top file corresponds with the environments defined in the ``file_roots`` variable. In a simple, single environment setup you only have the ``base`` environment, and therefore only one state tree. 
Here is a simple example of ``file_roots`` in the master configuration(默认为/etc/salt/master):

::

    file_roots:
        base:
            - /srv/salt

This means that the top file will only have one environment to pull from, here is a simple, single environment top file:

::

    base:
        '*':
            - core
            - edit

This also means that ``/srv/salt`` has a state tree. But if you want to use multiple environments, or partition the file server to serve more than just the state tree, then the ``file_roots`` option can be expanded:

::

    file_roots:
        base:
            - /srv/salt/base
        dev:
            - /srv/salt/dev
        qa:
            - /srv/salt/qa
        prod:
            - /srv/salt/prod

Then out top files could reference the environments:

::

    dev:
        'webserver*dev*':
            - webserver
        'db*dev*':
            - db
    qa:
        'webserver*qa*':
            - webserver
        'db*qa*':
            - db
    prod:
        'webserver*prod*':
            - webserver
        'db*prod*':
            - db

In this setup we have state trees in three of the four environments, and no state tree in the base environment. Notice that the targets for the minions specify environment data. In Salt the master determines who is in what environment, 
and many environments can be crossed together. For instance, a separate global state tree could be added to the base environment if it suits your deployment:

::

    base:
        '*':
            - global
    dev:
        'webserver*dev*':
            - webserver
        'db*dev*':
            - db
    qa:
        'webserver*qa*':
            - webserver
        'db*qa*':
            - db
    prod:
        'webserver*prod*':
            - webserver
        'db*prod*':
            - db
    
Other Ways of Targeting Minions
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Here is a slightly more complex top file example, showing the different types of matches you can perform:

::

    base:
        '*':
            - ldap-client
            - networking
            - salt.minion

        'salt-master*':
            - salt.master

        '^(memcache|web).(qa|prod).loc$':
            - match: pcre
            - nagios.mon.web
            - apache.server

        'os:Ubuntu':
            - match: grain
            - repos.ubuntu

        'os:(RedHat|CentOS)':
            - match: grain_pcre
            - repos.epel

        'foo,bar,baz':
            - match: list
            - database

        'somekey:abc':
            - match: pillar
            - xyz

        'nag1* or G@role:monitoring':
            - match: compound
            - nagios.server
    
In this example ``top.sls`` , all minions get the ldap-client, networking and salt.minion states. Any minion with an id matching the ``salt-master*`` glob will get the salt.master state. Any minion with ids matching 
the regular expression ``^(memcache|web).(qa|prod).loc$`` will get the nagios.mon.web and apache.server states. All Ubuntu minions will receive the repos.ubuntu state, while all RHEL and CentOS minions will receive 
the repos.epel state. The minions ``foo``, ``bar``, and ``baz`` will receive the database state. Any minion with a pillar named ``somekey``, having a value of ``abc`` will receive the xyz state. Finally, minions 
with ids matching the nag1* glob or with a grain named ``role`` equal to ``monitoring`` will receive the nagios.server state.

How Top Files Are Compiled
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

As mentioned earlier, the top files in the different environments are compiled into a single set of data. The way in which this is done follows a few rules, which are important 
to understand when arranging top files in different environments.

1. The ``base`` environment's top file is processed first. Any environment which is defined in the ``base`` top.sls as well as another environment's top file, will use the instance of the environment configured in ``base`` 
   and ignore all other instances. In other words, the ``base`` top file is authoritative when defining environments. Therefore, in the example below, the ``dev`` section in ``/srv/salt/dev/top.sls`` would be completely ignored.

``/srv/salt/base/top.sls:``

::

    base:
        '*':
            - common
    dev:
        'webserver*dev*':
            - webserver
        'db*dev*':
            - db

``/srv/salt/dev/top.sls:``

::

    dev:
        '10.10.100.0/24':
            - match: ipcidr
            - deployments.dev.site1
        '10.10.101.0/24':
            - match: ipcidr
            - deployments.dev.site2
    
.. note::

    The rules below assume that the environments being discussed were not defined in the base ``top`` file.

2. If, for some reason, the ``base`` environment is not configured in the ``base`` environment's top file, then the other environments will be checked in alphabetical order. The first top file found to contain a section for 
   the ``base`` environment wins, and the other top files' ``base`` sections are ignored. So, provided there is no ``base`` section in the ``base`` top file, with the below two top files the ``dev`` environment would win out, 
   and the ``common.centos`` SLS would not be applied to CentOS hosts.

``/srv/salt/dev/top.sls:``

::

    base:
        'os:Ubuntu':
            - common.ubuntu
    dev:
        'webserver*dev*':
            - webserver
        'db*dev*':
            - db
    
``/srv/salt/qa/top.sls:``

::

    base:
        'os:Ubuntu':
            - common.ubuntu
        'os:CentOS':
            - common.centos
    qa:
        'webserver*qa*':
            - webserver
        'db*qa*':
            - db
    
3. For environments other than ``base``, the top file in a given environment will be checked for a section matching the environment's name. If one is found, then it is used. Otherwise, the remaining (non-\ ``base``\ ) environments 
   will be checked in alphabetical order. In the below example, the ``qa`` section in ``/srv/salt/dev/top.sls`` will be ignored, but if ``/srv/salt/qa/top.sls`` were cleared or removed, then the states configured for the ``qa`` 
   environment in ``/srv/salt/dev/top.sls`` will be applied.

``/srv/salt/dev/top.sls:``

::

    dev:
        'webserver*dev*':
            - webserver
        'db*dev*':
            - db
    qa:
        '10.10.200.0/24':
            - match: ipcidr
            - deployments.qa.site1
        '10.10.201.0/24':
            - match: ipcidr
            - deployments.qa.site2
    
``/srv/salt/qa/top.sls:``

::

    qa:
        'webserver*qa*':
            - webserver
        'db*qa*':
            - db

Default Data - YAML
------------------------

::

    apache:
        pkg:
            - installed
        service:
            - running
            - require:
                - pkg: apache

Adding Configs and Users
-----------------------------

When setting up a service like an Apache web server, many more components may need to be added. The Apache configuration file will most likely be managed, and a user and group may need to be set up.

::

    apache:
      pkg:
        - installed
      service:
        - running
        - watch:
          - pkg: apache
          - file: /etc/httpd/conf/httpd.conf
          - user: apache
      user.present:
        - uid: 87
        - gid: 87
        - home: /var/www/html
        - shell: /bin/nologin
        - require:
          - group: apache
      group.present:
        - gid: 87
        - require:
          - pkg: apache
    
    /etc/httpd/conf/httpd.conf:
      file.managed:
        - source: salt://apache/httpd.conf
        - user: root
        - group: root
        - mode: 644

The ``require`` statement under service was changed to watch, and is now watching 3 states instead of just one. The watch statement does the same thing as require, making sure that the other states run 
before running the state with a watch, but it adds an extra component. The ``watch`` statement will run the state's watcher function for any changes to the watched states. So if the package was updated, 
the config file changed, or the user uid modified, then the service state's watcher will be run. The service state's watcher just restarts the service, so in this case, a change in the config file 
will also trigger a restart of the respective service.

Moving Beyond a Single SLS
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

When setting up Salt States in a scalable manner, more than one SLS will need to be used.

Understanding the Render System
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Since SLS data is simply that (data), it does not need to be represented with YAML. Salt defaults to YAML because it is very straightforward and easy to learn and use. But the SLS files can be rendered 
from almost any imaginable medium, so long as a renderer module is provided.

The default rendering system is the ``yaml_jinja`` renderer. The ``yaml_jinja`` renderer will first pass the template through the `Jinja2 <http://jinja.pocoo.org/>`_ templating system, and then through the YAML parser. 
The benefit here is that full programming constructs are available when creating SLS files.
