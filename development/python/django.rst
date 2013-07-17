Django
===========

Each view is responsible for doing one of two things: Returing an HttpResponse
object containing the content for the requested page, or raising an exception
such as Http404.

Generally, a view retrieves data according to the parameters, loads a template
and renders the template with the retrieved data.

The syncdb command looks at the INSTALLED_APPS setting and creates any necessary
database tables according to the database settings in your mysite/settings.py
file.


**Projects vs. apps**

What's the difference between a project and an app? An app is a Web application
that does something - e.g., a Weblog system, a database of public records or a
simple poll app. A project is a collection of configuration and apps for a
particular Web site. A project can contain multiple apps. An app can be in
multiple projects.

To create your app, make sure you're in the same directory as manage.py and type
this command:

::
    python manage.py startapp polls

That'll create a directory polls, which is laid out like this:

::
    polls/
        __init__.py
        models.py
        tests.py
        views.py


::

    python manage.py sql polls
    python manage.py validate   # Checks for any errors in the construction of your models
    python manage.py sqlclear polls # Outputs the necessary DROP TABLE statements for this app, according to which tables already exist in your database(if any).

    python manage.py syncdb


::

    from polls.models import Poll, Choice

    Poll.objects.all()

    from django.utils import timezone
    p = Poll(question="What's new?", pub_date=timezone.now())

    # save the object into the database. You have to call save() explicitly.
    p.save()

    # Now it has an ID.
    p.id

    # Access database columns via Python attributes.
    p.question
    p.pub_date

    # Change values by changing the attributes, then calling save()
    p.question = "What's up?"
    p.save()


**It's important to add ``__unicode__()`` methods to your models, not only for your own sanity when dealing with the
interactive prompt, but also because objects' representations are used throughout Django's automatically-generated
admin.**

Generating admin sites for your staff or clients to add , change and delete content is tedious work that doesn't require
much creativity. For that reason, Django entirely automates creation of admin interfaces for models.


::

    python manage.py runserver  # start the development server


如何在admin页面中注册使用App的Model？以App polls为例，在polls目录中创建一个名为admin.py的文件，文件内容为：

::

    from django.contrib import admin
    from polls.models import Poll

    admin.site.register(Poll)

You’ll need to restart the development server to see your changes. Normally, the server auto-reloads code every time
you modify a file, but the action of creating a new file doesn’t trigger the auto-reloading logic.

