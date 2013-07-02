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
