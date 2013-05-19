Django
===========


开发步骤
----------

1.进入存放工程代码的目录，执行::

    django-admin.py startproject mysite

2.在 `mysite/settings.py` 中设置管理员、数据库、时区等信息。

3.为 `mysite/settings.py` 中变量 `INSTALLED_APPS` 指定加载的模块创建数据库::

    python manage.py syncdb

4.启动开发服务器::

    python manage.py runserver

之后访问127.0.0.1:8000/admin就可以看到控制台界面了。

5.一个Django项目（Project）可以包含多个应用（app），可如下创建你的应用::

    python manage.py startapp polls  # polls为应用名称

6.对于数据库驱动的应用，先为应用创建模型。对于polls，编辑polls/models.py::

    from django.db import models
    
    class Poll(models.Model):
        question = models.CharField(max_length=200)
        pub_date = models.DateTimeField('date published')

    class Choice(models.Model):
        poll = models.ForeignKey(Poll)
        choice_text = models.CharField(max_length=200)
        votes = models.IntegerField(default=0)

7.在 `mysite/settings.py`
文件中激活应用，即在INSTALLED_APPS设置中加入"polls"字符。

8.查看Django根据应用的模型产生用于创建数据表的SQL::

    python manage.py sql polls

9.再次运行syncdb命令在你的数据库中创建这些模型对应的表::

    python manage.py syncdb
