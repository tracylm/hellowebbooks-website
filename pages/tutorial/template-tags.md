# Fun With Template Tags

Before we get into dynamic data, let’s explore the fun things that Django’s
template system has built in.

## Building complex templates with inheritance

One of the biggest advantages of using a framework like Django is template
inheritance. Instead of having a bunch of files that repeat information (such as
`<header>...</header>` tags, your nav, etc.), you can have one base layout
template that all other templates import.

In your templates directory, add a new file called *base.html*.

{linenos=off}
```
$ cd collection/templates
collection/templates $ touch base.html
```

We’re going to strip out all website-common code from *index.html* and stick it
into *base.html*:

{title="base.html", lang="html+django"}
```
{% load staticfiles %}
<!doctype html>
<html>
    <head>
        # leanpub-start-insert
        <title>
             <!-- we want to override page titles -->
        </title>
        # leanpub-end-insert
        <link rel="stylesheet" 
             href="{% static 'css/style.css' %}" />
    </head>
    <body>
        # leanpub-start-insert
        <!-- as well as override content bits -->
        # leanpub-end-insert
    </body>
</html>
```

We can define blocks in our layout template where information can be added or
overwritten. For example, we’ll add `{% block content %}{% endblock content %}`
where we’d like the body content from *index.html* to go. We can also add other
overridable blocks in our layout template: I usually add a block for the page
title, optional header information (for including other CSS files, for example),
and right-before-closing-body tag (for adding other Javascript files).

*base.html* now:

{title="base.html", lang="html+django"}
```
{% load staticfiles %}
<!doctype html>
<html>
    <head>
        # leanpub-start-insert
        <title>
            {% block title %}
                My Hello Web App Project
            {% endblock title %}
        </title>
        # leanpub-end-insert
        <link rel="stylesheet"
            href="{% static 'css/style.css' %}" />
        # leanpub-start-insert
        {% block header %}{% endblock header %}
        # leanpub-end-insert
    </head>
    <body>
        # leanpub-start-insert
        {% block content %}{% endblock content %}
        {% block footer %}{% endblock footer %}
        # leanpub-end-insert
    </body>
</html>
```

Note that we have content in the `{% block title %}` tags. That will be our
default content that’ll show if child pages don’t override the title.

Head back over to *index.html*, and delete everything other than our content
that we removed from the new layout template. To indicate that this template has
a layout template that will extend, we'll add `{% extends 'base.html'
%}` to the top of the template, and then we can add in blocks in the template
for the sections we defined in our *base.html* template.

Our new *index.html*:

{title="index.html", lang="html+django"}
```
{% extends 'base.html' %}
{% block title %}
    Homepage - {{ block.super }}
{% endblock title %}

{% block content %}
    <h1>Hello World!</h1>
    <p>I am a basic website.</p>
{% endblock content %}
```

A few important things to note:

* Each block (like `{% block content %}`) has the name we defined in our
    base.html file so the template engine knows where to render which part.
* Check out the title block. See the `{{ block.super }}`? In our *base.html*
    file, we defined "My Hello Web App Project" as the default content. `{{
    block.super }}` will insert the contents of the default block instead of
    overriding it. Now our page titles can have both the local page name as well
    as the title of the website. If you ever change the main title of the
    website, you only have to update it in *base.html* and it will automatically
    update the child pages.
* If you’re loading static assets like images, you’ll still need to add `{% load
    staticfiles %}` to the top of the base layout file. We’re not yet in
    *index.html* so we haven’t added it but keep in mind if you add static files
    to your version.

Now you can add new simple pages to your website without having to repeat the
same code over and over!

## Adding a few other static pages

Having template inheritance doesn't feel real until you have multiple templates
all extending one layout template.

Let’s create a few extra pages: an *about.html* and *contact.html*. First,
create the HTML files in your templates directory. I’m going to make it easy by
copying our *index.html* file because it already has the `{% block %}` tags
added for inheritance.

{linenos=off}
```
templates $ cp index.html about.html
templates $ cp index.html contact.html
```

Then open the pages and edit the HTML so it makes sense for each page.

{title="about.html", lang="html+django"}
```
{% extends 'base.html' %}
{% block title %}
    About - {{ block.super }}
{% endblock title %}

{% block content %}
    <h1>About page</h1>
    <p>About content.</p>
{% endblock content %}
```

{title="contact.html", lang="html+django"}
```
{% extends 'base.html' %}
{% block title %}
    Contact - {{ block.super }}
{% endblock title %}

{% block content %}
    <h1>Contact page</h1>
    <p>Contact content.</p>
{% endblock content %}
```

Last, we’ll need to create a few new URL definitions in urls.py. If your new
template won’t display information from your database — only simple HTML and CSS
— then we can simplify our URL definition with a shortcut without having to add
anything to *views.py*.

Open your *urls.py* — we're going to import something new and add the new URL
definitions.

{title="urls.py", lang="python"}
```
from django.conf.urls import url
from django.contrib import administration
# leanpub-start-insert
from django.views.generic import TemplateView
# leanpub-end-insert
from collection import views

urlpatterns = [
    url(r'^$', views.index, name='home'),
    # The new URL entries we're adding:
    # leanpub-start-insert
    url(r'^about/$',
        TemplateView.as_view(template_name='about.html'),
        name='about'),
    url(r'^contact/$', 
        TemplateView.as_view(template_name='contact.html'),
        name='contact'),
    # leanpub-end-insert

    url(r'^admin/', admin.site.urls),
]
```

These new patterns are nearly the same thing we had before when we made a URL
entry: The regular expression at the beginning, and the name of the view at the
end. Instead of pointing to a function in *views.py* though, we're going to use
Django's *generic* view called `TemplateView` that basically says, "Hey, just
display this template."

### Add navigation

Every page will need to have navigation, so we'll add that to our layout
template. Open it and add the following basic HTML (I also added a header and
some other bits):

{title="base.html", lang="python"}
```
{% load staticfiles %}
<!doctype html>
<html>
    <head>
        <title>
            {% block title %}
                My Hello Web App Project
            {% endblock title %}
        </title>
        <link rel="stylesheet"
            href="{% static 'css/style.css' %}" />
        {% block header %}{% endblock header %}
    </head>
    <body>
        # leanpub-start-insert
        <header>
            <h1>Hello Web App</h1>
            <nav>
                <ul>
                    <li>
                        <a href="{% url 'home' %}">Home</a>
                    </li>
                    <li>
                        <a href="{% url 'about' %}">About</a>
                    </li>
                    <li>
                        <a href="{% url 'contact' %}">Contact</a>
                    </li>
                </ul>
            </nav>
        </header>
        # leanpub-end-insert
        {% block content %}{% endblock content %}
        {% block footer %}{% endblock footer %}
    </body>
</html>
```

Surprise: We’re not going to link to other HTML files like we normally would.
This is why we name URLs — we can tell Django that we want to link to the URL
named "home," and Django will automatically insert the right path. In the
future, if you ever decide to change the URL (like changing */about/* to
*/about-us/*), you would just need to change it in your *urls.py* file and
Django will update your templates for you.

Check out your website with it's shiny, new navigation:

![](images/nav.png) 

## Passing in variables and template tags

At this point you could launch this basic website, but dynamic content is really
why we’re here.

To show how variables get passed into the templates from the view, head back to
*views.py* to the index function (a function is what each of the code blocks
starting with `def` is called). For fun, let’s define a variable with an integer
and pass it to the view to see the fun things that Django’s template tags can
do.

We're going to create a variable that holds a number and then make this variable
available to the template. Update your view to the following:

{title="views.py", lang="python"}
```
from django.shortcuts import render

def index(request):
    # leanpub-start-insert
    # defining the variable
    number = 6
    # passing the variable to the view
    return render(request, 'index.html', {
        'number': number,
    })
    # leanpub-end-insert
```

Back over in *index.html*, we can access these variables in two ways:

* `{{ variable }}`: Surrounded by two curly braces, this simply displays what
    was passed in.
* `{% tag %}`: `{% %}` is django's version of `< >` in HTML. A tag in curly
    braces/percent signs means we’re doing something either with that variable
    or to the template, such as loading static files.

To see this in action, let’s first display the variable. Add this to your
*index.html* file:

{title="index.html", lang="html+django"}
```
{% extends 'base.html' %}
{% block title %}
    Homepage - {{ block.super }}
{% endblock title %}

{% block content %}
    <h1>Hello World!</h1>
    <!-- here's the variable we are passing in -->
    # leanpub-start-insert
    <p>{{ number }}</p>
    # leanpub-end-insert
{% endblock content %}
```

![](images/6.png) 

Ta-da! Django translates `{{ number }}` to the value of the variable called
`number` from the view which was passed into the template.

Django’s templates also allow you to use basic logic similar to Python's within
the templates — most importantly, *if-else statements* and *for loops*. (Can’t
remember what these are? Refer to Learn Python The Hard Way
([http://hellowebapp.com/13](http://hellowebapp.com/13)) for a refresher.)

Check out an *if-else statement* in action:

{lang=django, line-numbers=on, starting-line-number=6}
```
<p>{% if number %}Number exists!{% endif %}</p>
```

![](images/numberexists.png) 

And another example:

{lang=django, line-numbers=on, starting-line-number=6}
    <p>There are six dogs! <b>
    {% if number == 6 %}
        True
    {% else %}
        False
    {% endif %}
    </b></p>

![](images/sixdogs.png) 

Django also has a lot of fun template tags
([http://hellowebapp.com/14](http://hellowebapp.com/14)). For example, if you
were passing in the number of dogs in your website, you could make a word plural
if the number is more than one:

{lang=django, line-numbers=on, starting-line-number=6}
```
<p>There are {{ number }} dog{{ number|pluralize }}!</p>
```

![](images/pluralize.png) 

This template tag also accepts different suffixes:

{lang=django, line-numbers=on, starting-line-number=6}
```
<p>There are {{ number }} walrus{{ number|pluralize:"es" }}!</p>
```

![](images/sixwalruses.png) 

Django has an optional library you can use to format and "humanize" numbers. Add
`'django.contrib.humanize'`, to your `INSTALLED_APPS` list in *settings.py*:

{title="settings.py", lang="python", line-numbers=on, starting-line-number=40}
```
INSTALLED_APPS = (
    'collection',
    'django.contrib.admin',
    'django.contrib.auth',
    'django.contrib.contenttypes',
    'django.contrib.sessions',
    'django.contrib.messages',
    'django.contrib.staticfiles',
    # leanpub-start-insert
    'django.contrib.humanize',
    # leanpub-end-insert
)
```

Then add `{% load humanize %}` to the top of our template. We're going to start
with `apnumber`, which spells out the numbers 1-9:

{title="index.html", lang="python"}
```
{% extends 'base.html' %}
# leanpub-start-insert
{% load humanize %}
# leanpub-end-insert
{% block title %}
    Homepage - {{ block.super }}
{% endblock title %}

{% block content %}
    <h1>Hello World!</h1>
    # leanpub-start-insert
    <p>There are {{ number|apnumber }} dogs!</p>
    # leanpub-end-insert
{% endblock content %}
```

![](images/apnumber.png) 

Note that what was displayed as "6" before is now "six". Read more about
humanize and the other fun options it has here:
[http://hellowebapp.com/15](http://hellowebapp.com/15)

There are also a bunch of template tags that work on strings. Let’s change our
view:

{title="views.py", lang="python", line-numbers=on, starting-line-number=4}
```
def index(request):
    number = 6
    # don't forget the quotes because it's a string,
    # not an integer
    # leanpub-start-insert
    thing = "Thing name" 
    # leanpub-end-insert
    return render(request, 'index.html', {
        'number': number,
        # don't forget to pass it in, and the last comma
        # leanpub-start-insert
        'thing': thing, 
        # leanpub-end-insert
    })
```

As well as our template:

{lang="html+django", line-numbers=on, starting-line-number=6}
```
<p>This is my name: {{ thing }}</p>
```

![](images/stringname.png) 

So now we can play with more fun template tags, like this one which would turn
it into a slug, a shortened, lowercased version of a string used for URLs:

{lang="html+django", line-numbers=on, starting-line-number=6}
```
<p>This is my name: {{ thing|slugify }}</p>
```

![](images/slugify.png) 

Feel like using title case?

{lang="html+django", line-numbers=on, starting-line-number=6}
```
<p>This is my name: {{ thing|title }}</p>
```

![](images/title.png) 

Perhaps it's too long of a name for you? Truncate it.

{lang="html+django", line-numbers=on, starting-line-number=6}
```
<p>This is my name: {{ thing|truncatewords:1 }}</p>
```

![](images/truncate.png) 

I hope at this point you can see how Django’s template system has a bunch of
powerful features that knock a static website out of the water. Check out the
full list of included Django template tags here:
[http://hellowebapp.com/14](http://hellowebapp.com/14). It gets even more fun
when you have real dynamic data to work with, which is coming up in the next
section.

Don't forget to commit your work at this point!