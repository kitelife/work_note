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

The ``url()`` function is passed four arguments, two required: ``regex`` and
``view`` , and two optional: ``kwargs`` , and ``name`` .

**regex**

The term "regex" is a commonly used short form meaning "regular expression",
which is a syntax for matching patterns in strings, or in this case, url
patterns. Django starts at the first regular expression and makes its way down
the list, comparing the requested URL against each regular expression until it
finds one that matches.

Note that these regular expressions do not match GET and POST parameters, or the
domain name.

Finally, a performance note: these regular expressions are compiled the first
time the URLconf module is loaded. They're super fast.

**view**

When Django finds a regular expression match, Django calls the specified view
function, with an HttpRequest object as the first argument and any "captured"
values from the regular expression as other arguments. If the regex uses simple
captures, values are passed as positional arguments; if it uses named captures,
values are passed as keyword arguments.

**kwargs**

Arbitrary keyword arguments can be passed in a dictionary to the target view.

**name**

Naming your URL lets you refer to it unambiguously from elsewhere in Django
especially templates. This powerful feature allows you to make global changes to
the url patterns of your project while only touching a single file.


Each view is responsible for doing one of two things: returning an HttpResponse
object containing the content for the requested page, or raising an exception
such as Http404. The rest is up to you.

Your view can read records from a database, or not. It can use a template system
such as Django's - or a third-party Python template system - or not. It can
generate a PDF file, output XML, create a ZIP file on the fly, anything you
want, using whatever Python libraries you want.

All Django wants is that HttpResponse. Or an exception.

------

::

    from django.http import HttpResponse
    from django.template import RequestContext, loader

    from polls.models import Poll

    def index(request):
        latest_poll_list = Poll.objects.order_by('-pub_date')[:5]
        template = loader.get_template('polls/index.html')
        context = RequestContext(request, {
            'latest_poll_list': latest_poll_list,
        })
        return HttpResponse(template.render(context))

A shortcut: **render()**

::

    from django.shortcuts import render

    from polls.models import Poll

    def index(request):
        latest_poll_list = Poll.objects.all().order_by('-pub_date')[:5]
        context = {'latest_poll_list': latest_poll_list}
        return render(request, 'polls/index.html', context)


**Raising a 404 error**

::

    from django.http import Http404

    def detail(request, poll_id):
        try:
            poll = Poll.objects.get(pk=poll_id)
        except Poll.DoesNotExist:
            raise Http404
        return render(request, 'polls/detail.html', {'poll': poll})

A shortcut: **get_object_or_404()**

::

    from django.shortcuts import render, get_object_or_404

    def detail(request, poll_id):
        poll = get_object_or_404(Poll, pk=poll_id)
        return render(request, 'polls/detail.html', {'poll': poll})

The get_object_or_404() function takes a Django model as its first argument and
an aribitrary number of keyword arguments, which it passes to the get() function
of model's manager. It raises Http404 if the object doesn't exist.

There's also a **get_list_or_404** function, which works just as
get_object_or_404() - except using filter() instead of get(). It raises Http404
if the list is empty.


::

    from django.shortcuts import get_object_or_404, render
    from django.http import HttpResponseRedirect, HttpResponse
    from django.core.urlresolvers import reverse
    from polls.models import Choice, Poll

    def vote(request, poll_id):
        p = get_object_or_404(Poll, pk=poll_id)
        try:
            selected_choice = p.choice_set.get(pk=request.POST['choice'])
        except (KeyError, Choice.DoesNotExist):
            return render(request, 'polls/detail.html', {
                'poll': p,
                'error_message': "You didn't select a choice.",
            })
        else:
            selected_choice.votes += 1
            selected_choice.save()

            return HttpResponseRedirect(reverse('polls.results', args=(p.id,)))


**Use generic views: Less code is better**

::

    from django.conf.urls import patterns, url

    from polls import views

    urlpatterns = patterns('',
        url(r'^$', views.IndexView.as_view(), name='index'),
        url(r'^(?P<pk>\d+)/$', views.DetailView.as_view(), name='detail'),
        url(r'^(?P<pk>\d+)/results/$', views.ResultsView.as_view(), name='results'),
        url(r'^(?P<poll_id>\d+)/vote/$', views.vote, name='vote'),
    )


::

    from django.shortcuts import get_object_or_404, render
    from django.http import HttpResponseRedirect
    from django.core.urlresolvers import reverse
    from django.views import generic

    from polls.models import Choice, Poll

    class IndexView(generic.ListView):
        template_name = 'polls/index.html'
        context_object_name = 'latest_poll_list'

        def get_queryset(self):
            """Return the last five published polls"""
            return Poll.objects.order_by('-pub_date')[:5]

    class DetailView(generic.DetailView):
        model = Poll
        template_name = 'polls/detail.html'

    class ResultsView(generic.DetailView):
        model = Poll
        template_name = 'polls/results.html'

    def vote(request, poll_id):
        ...


参考资料
===========
- `Scaling Disqus <http://www.slideshare.net/zeeg/djangocon-2010-scaling-disqus>`_
- `django-xadmin <http://sshwsfc.github.io/django-xadmin/>`_
- `Web Development with Python and Django <https://speakerdeck.com/mpirnat/web-development-with-python-and-django-2014>`_
