Pillar Walkthrough
=======================

The pillar interface inside of Salt is one of the most important components of a Salt deployment.Pillar is the interface **used to generate arbitrary data for specific minions**. The data generated in pillar is 
made available to almost every component of Salt and is used for a number of purposes:

* **Highly Sensitive Data** : Information transferred via pillar is guaranteed to only be presented to the minions that are targeted, this makes pillar the engine to use in Salt for managing security information, 
  such as cryptographic keys and passwords.

* **Minion Configuration** : Minion modules such as the execution modules, states, and returners can often be configured via data stored in pillar.

* **Variable** : Variables which need to be assigned to specific minions or groups of minions can be defined in pillar and then accessed inside sls formulas and template files.

* **Arbitrary Data** : Pillar can contain any basic data structure, so a list of values, or a key/value store can be defined making it easy to iterate over a group of values in sls formulas.

Setting Up Pillar
---------------------

The pillar is already running in Salt by default. The data in the minion's pillars can be seen via the following command:

::

    salt '*' pillar.items

By default the contents of the master configuration file are loaded into pillar for all minions, this is to enable the master configuration file to be used for global configuration of minions.

The pillar is built in a similar fashion as the state tree, it is comprised of sls files and has a top file, just like the state tree. The pillar is stored in a different location on 
the Salt master than the state tree. The default location for the pillar is in /srv/pillar.

.. note::

    The pillar location can be configured via the *pillar_roots* option inside the master configuration file.
    
To start setting up the pillar, the /srv/pillar directory needs to be present:

::

    mkdir /srv/pillar

Now a simple top file, following the same format as the top file used for states needs to be created:

``/srv/pillar/top.sls``:

::

    base:
        '*':
            - data

This top file associates the data.sls file to all minions. Now the ``/srv/pillar/data.sls`` file needs to be populated:

``/srv/pillar/data.sls``:

::

    info: some data

Now that the file has been saved the minions' pillars will be updated:

::

    salt '*' pillar.items

The key ``info`` should now appear in the returned pillar data.

More Complex Data
^^^^^^^^^^^^^^^^^^^^^^

Pillar files are sls files, just like states, but unlike states they do not need to define **formulas**, the data can be arbitrary, this example for instance sets up user data with a UID:

``/srv/pillar/users/init.sls``:

::

    users:
        thatch: 1000
        shouse: 1001
        utahdave: 1002
        redbeard: 1003

.. note::

    The same directory lookups that exist in states exist in pillar, so the file ``users/init.sls`` can be referenced with ``users`` in the top file.

The top file will need to be updated to include this sls file:

``/srv/pillar/top.sls``:

::

    base:
        '*':
            - data
            - users

Now the data will be available to the minions. To use the pillar data in a state just access the pillar via Jinja:

``/srv/salt/users/init.sls``:

::

    {% for user, uid in pillar.get('users', {}).items() %}
    {{user}}:
        user.present:
            - uid: {{uid}}
    {% endfor %}

This approach allows for users to be safely defined in a pillar and then the user data is applied in an sls file.

Paramaterizing States With Pillar
----------------------------------------

One of the most powerful abstractions in pillar is the ability to parameterize states. Instead of defining macros or functions within the state context the entire state tree can be freely parameterized relative to the minion's pillar.

This approach allows for Salt to be very flexible while staying very straightforward. It also means that simple sls formulas used in the state tree can be directly parameterized without needing to refactor the state tree.

A simple example is to set up a mapping of package names in pillar for separate Linux distributions:

``/srv/pillar/pkg/init.sls``:

::

    pkgs:
        {% if grains['os_family'] == 'Redhat' %}
        apache: httpd
        vim: vim-enhanced
        {% elif grains['os_family'] == 'Debian' %}
        apache: apache2
        vim: vim
        {% elif grains['os'] == 'Arch' %}
        apache: apache
        vim: vim
        {% endif %}

The new ``pkg`` sls needs to be added to the top file:

``/srv/pillar/top.sls``:

::

    base:
        '*':
            - data
            - users
            - pkg

Now the minions will auto map values based on respective operating systems inside of the pillar, so sls files can be safely parameterized:

``/srv/salt/apache/init.sls``:

::

    apache:
        pkg.installed:
            - name: {{ pillar['pkgs']['apache'] }}

Or, if no pillar is available a default can be set as well:

``/srv/salt/apache/init.sls``:

::

    apache:
        pkg.installed:
            - name: {{ salt['pillar.get']('pkgs:apache', 'httpd')}}

In the above example, if the pillar value ``pillar['pkgs']['apache']`` is not set in the minion's pillar, then the default of ``httpd`` will be used.
