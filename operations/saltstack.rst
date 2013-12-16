SaltStack
==============

Salt is a new approach to infrastructure management. Easy enough to get running in minutes, scalable enough to manage tens of thousands of servers, and fast enough to communicate with them in *seconds* .

Salt delivers a dynamic communication bus for infrastructures that can be used for orchestration(业务流程？), remote execution, configuration management and much more.


SaltStack Walkthrough
-------------------------

Salt is a different approach to infrastructure management, it is founded on the idea that high speed communication with large numbers of systems can open up new capabilities. 
This approach makes Salt a powerful multitasking system that can solve many specific problems in an infrastructure. The backbone of Salt is the remote execution engine, 
which creates a high speed, secure and bi-directional communication net for groups of systems. On top of this communication system Salt provides an extremely fast, flexible 
and easy to use configuration management system called ``Salt States`` .

------

**The Salt Master needs to bind to 2 TCP network ports on the system, these ports are 4505 and 4506.**

------

Now that the minion is started it will generate cryptographic keys and attempt to connect to the master. The next step is to venture back to the master server and accept the new minion's public key.

When the minion is started, it will generate an ``id`` value, unless it has been generated on a previous run and cached in the configuration directory ( ``/etc/salt`` by default). This is the name 
by which the minion will attempt to authenticate to the master. The following steps are attempted, in order to try to find a value that is not ``localhost`` :

1. The Python function ``socket.getfqdn()`` is run
2. ``/etc/hostname`` is checked (non-Windows only)
3. ``/etc/hosts`` ( ``%WINDIR%\system32\drivers\etc\hosts`` on Windows hosts) is checked for hostnames that map to anything within 127.0.0.0/8.

If none of the above are able to produce an id which is not ``localhost`` , then a sorted list of IP addresses on the minion (excluding any within 127.0.0.0/8) is inspected. The first publicly-routable 
IP address is used, if there is one. Otherwise, the first privately-routable IP address is used.

If all else fails, then ``localhost`` is used as a fallback.

.. note:: The minion id can be manually specified using the ``id`` parameter in the minion config file. If this configuration value is specified, it will override all other sources for the ``id`` .

------

Salt authenticates minions using public key encryption and authentication. For a minion to start accepting commands from the master the minion keys need to be accepted. The ``salt-key`` command is used to manage all of the keys on the master.

------

The ``salt`` command is comprised of command options, target specification, the function to execute, and arguments to the function. 

Salt comes with a vast library of functions available for execution, and Salt functions are self documenting. To see what functions are available on the minions execute the ``sys.doc`` function:

::

    salt '*' sys.doc

This will display a very large list of available functions and documentation on them, this documentation is also available `here <https://salt.readthedocs.org/en/latest/ref/modules/all/index.html>`_ .

These functions cover everything from shelling out to package management to manipulating database servers. They comprise a powerful system management API which is the backbone to Salt configuration management and many other aspects of Salt.

------

The `cmd <https://salt.readthedocs.org/en/latest/ref/modules/all/salt.modules.cmdmod.html>`_ module contains functions to shell out on minions, such as ``cmd.run`` and ``cmd.run_all`` :

::

    salt '*' cmd.run 'ls -l /etc'

------

The ``pkg`` functions automatically map local system package managers to the same salt functions. This means that ``pkg.install`` will install packages via yum on Red Hat based systems, apt on Debian systems, etc.:

::

    salt '*' pkg.install vim

------

The ``network.interfaces`` function will list all interfaces on a minion, along with their IP addresses, netmasks, MAC addresses, etc:

::

    salt '*' network.interfaces

------

``salt-call``

The examples so far have described running commands from the Master using the ``salt`` command, but when troubleshooting it can be more beneficial to login to the minion directly and use ``salt-call`` .
Doing so allows you to see the minion log messages specific to the command you are running (which are not part of the return data you see when running the command from the Master using ``salt`` ), 
More information on ``salt-call`` and how to use it can be found `here <https://salt.readthedocs.org/en/latest/topics/troubleshooting/index.html#using-salt-call>`_ .

------

**Grains**

Salt uses a system called `Grains <https://salt.readthedocs.org/en/latest/topics/targeting/grains.html>`_ to build up static data about minions. This data includes information about the operating system that is running, CPU architecture and much more. The grains system 
is used throughout Salt to deliver platform data to many components and to users.

Grains can also be statically set, this makes it easy to assign values to minions for grouping and managing. A common practice is to assign grains to minions to specify what the role or roles a minion 
might be. These static grains can be set in the minion configuration file or via the ``grains.setval`` function.

------

**Targeting**

Salt allows for minions to be targeted based on a wide range of criteria. The default targeting system uses globular expressions(应该是指 ``*`` 符号) to match minions.

Many other targeting systems can be used other than globs, these systems include:

* **Regular Expressions** : Target using PCRE compliant regular expressions
* **Grains** : Target based on grains data: `Targeting with Grains <https://salt.readthedocs.org/en/latest/topics/targeting/grains.html>`_
* **Pillar** : Target based on pillar data: `Targeting with Pillar <https://salt.readthedocs.org/en/latest/ref/pillar/index.html>`_
* **IP** : Target based on IP addr/subnet/range
* **Compound** : Create logic to target based on multiple target: `Targeting with Compound <https://salt.readthedocs.org/en/latest/topics/targeting/compound.html>`_
* **Nodegroup** : Target with nodegroups: `Targeting with Nodegroup <https://salt.readthedocs.org/en/latest/topics/targeting/nodegroups.html>`_

The concepts of targets are used on the command line with salt, but also function in many other areas as well, including the state system and the systems used for ACLs and user permissions.

------

Salt States
^^^^^^^^^^^^^^

Salt ``States`` , or the ``State System`` is the component of Salt made for configuration management.

.. note:: Salt states are based on data modeling, and build on a low level data structure that is used to execute each state function. Then more logical layers are built on top of each other. The high 
    layers of the state system which this tutorial will cover consists of everything that needs to be known to use states, the two high layers covered here are the *sls* layer and the highest layer *highstate* .

**The First SLS Formula**

The state system is built on sls formulas, these formulas are built out in files on Salt's file server. To make a very basic sls formula open up a file under /srv/salt named vim.sls and get vim installed:

::

    /srv/salt/vim.sls

::

    vim:
        pkg.installed

Now install vim on the minions by calling the sls directly:

::

    salt '*' state.sls vim

This command will invoke the state system and run the named sls which was just created, ``vim`` .

Now, to beef up the vim sls formula, a vimrc can be added:

::

    /srv/salt/vim.sls:

::

    vim:
        pkg.installed

    /etc/vimrc:
        file.managed:
            - source: salt://vimrc
            - mode: 644
            - user: root
            - group: root


**Adding Some Depth**

Obviously maintaining sls formulas right in the root of the file server will not scale out to reasonably sized deployments. This is why more depth is required. Start by making an nginx formula a better 
way, make an nginx subdirectory and add an init.sls file:

::

    /srv/salt/nginx/init.sls:

::

    nginx:
        pkg:
            - installed
        service:
            - running
            - require:
                - pkg: nginx

A few things are introduced in this sls formula, first is the service statement which ensures that the nginx service is running, but the nginx service can't be started unless the package is installed, 
hence the ``require`` . The ``require`` statement makes sure that the required component is executed before and that it results in success.

.. note:: The *require* option belongs to a family of options called *requisites* .Requisites are a powerful component of Salt States, for more information on how requisites work and what is available 
    see: `Requisites <https://salt.readthedocs.org/en/latest/ref/states/requisites.html>`_ Also evaluation ordering is available in Salt as well: `Ordering States <https://salt.readthedocs.org/en/latest/ref/states/ordering.html>`_

Now this new sls formula has a special name, ``init.sls`` , when an sls formula is named ``init.sls`` it inherits the name of the directory path that contains it, so this formula can be referenced via 
the following command:

::

    salt '*' state.sls nginx

Now that subdirectories can be used the vim.sls formula can be cleaned up, but to make things more flexible, move the vim.sls and vimrc into a new subdirectory called ``edit`` and change the vim.sls file to reflect the change:

::

    /srv/salt/edit/vim.sls:

::

    vim:
        pkg.installed
    /etc/vimrc:
        file.managed:
            - source: salt://edit/vimrc
            - mode: 644
            - user: root
            - group: root

Now the formula is referenced as ``edit.vim`` because it resides in the edit subdirectory. Now the edit subdirectory can contain formulas for emacs, nano, joe or any other editor that may need to be deployed.

.. toctree::
    :maxdepth: 2

    saltstack/starting_states
    saltstack/pillar_walkthrough
    saltstack/getting_deeper_into_states


相关链接
---------------

https://github.com/saltstack/salt

http://docs.saltstack.com/

http://wiki.saltstack.cn/

`系统自动化配置和管理工具SaltStack <http://www.vpsee.com/2013/08/a-system-configuration-management-and-orchestration-tool-saltstack/>`_

`灿哥的Blog - SaltStack <http://www.shencan.net/index.php/category/%E8%87%AA%E5%8A%A8%E5%8C%96%E8%BF%90%E7%BB%B4/saltstack/>`_

`salt.readthedocs.org <https://salt.readthedocs.org/en/latest/>`_
