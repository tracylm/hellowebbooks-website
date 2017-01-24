# Setting Up Your Templates

Congrats, you've launched your Django web app! How do you actually make this
look like a website? Note that the installation instructions didn't mention
anything about templates, HTML, CSS, or any other files. Let's set that stuff up
now.

Head back to your command line, and `cd` into your `collection` folder (the one
with *models.py*). Using `mkdir`, create a "templates" directory, and then
create *index.html* within it.

{linenos=off}
```
$ cd collection
collection $ mkdir templates
collection $ cd templates
collection/templates $ touch index.html
```

Open up *index.html* in your preferred code editor and add the necessary pieces
to display a typical webpage. We're going to use HTML5 but only the bare bones
for now — we can get fancy later.

{title="index.html", lang=html}
```
<!doctype html>
<html>
    <head>
        <title>My Hello Web App Project</title>
    </head>
    <body>
        <h1>Hello World!</h1>
        <p>I am a basic website.</p>
    </body>
</html>
```

If you open up your browser and go to *http://localhost:8000*, you'll still see
the same default page you saw before — none of the HTML above. We need to tell
Django how to get to that file.

## Adding a URL to `urls.py`

Django doesn't know what to do with the *index.html* file we just created. So
head to *urls.py*, which is in the same folder as *settings.py* — we're going to
associate the root directory (*/*) with the newly created *index.html*.

In *urls.py*'s entries area, find the `urlpatterns` line, remove the
pre-existing comments, and add in the lines indicated:

{title="urls.py", lang="python"}
```
from django.conf.urls import url
from django.contrib import admin

# leanpub-start-insert
from collection import views
# leanpub-end-insert

urlpatterns = [
# leanpub-start-insert
    url(r'^$', views.index, name='home'),
# leanpub-end-insert
    url(r'^admin/', admin.site.urls),
]
```

There are essentially four parts to this new line:
* `url()` encases the entry to indicate it's a URL entry.
* `r'^$'`  is the beginning of the URL pattern. Might look confusing, but just
    remember the pattern for now.
* `views.index` means that we’ll use the index view in views.py in our app
    `collection` (imported at the top). We’ll create this soon.
* Last, `name='home'` is optional, but allows us to assign a name to this URL so
    we can refer to it in the future as "home". I'll explain how this ties in
    later.

I am using the "typical" urls.py layout here, but if it's more understandable to
you, you can layout the URL definition like this:

{title="urls.py", lang="python"}
```
urlpatterns = [
    url(
        regex=r'^$', 
        view=views.index,
        name='home',
    ),
    url(r'^admin/', admin.site.urls),
]
```

This will help you keep track of which bit is what. I'm going to keep using the
previous configuration in this book though, since it's how it's usually laid out
in Django.

Apologies if URL configuration sounds complicated. The most you need to remember
here is that there is a regular expression for the path, then the view
definition, and then we gave the URL a name. We can easily copy and paste from
this template in the future for new URL entries.

## Creating your first view

Now that we have a template and a URL entry, we need to tie them together, so
that the URL will display the template. This is *views.py*'s job, which lives in
the `collection` app.

There are a ton of different ways to display a template simply in the views, and
my favorite is Django's shortcut function called `render`. This is something
you'll need to import, but Django anticipates that you'll use it and already has
it added to the top of *views.py* for us.

{title="views.py", lang="python"}
```
from django.shortcuts import render

# Create your views here.
# leanpub-start-insert
def index(request): 
    # this is your new view
    return render(request, 'index.html')
# leanpub-end-insert
```

Basically, `urls.py` will catch that someone wants the homepage and points to
this piece of code, which will render the `index.html` template. Now open up
your browser and check out the new homepage on *http://localhost:8000*. Woohoo!

![](images/helloworld.png) 

## Adding static files

We're now displaying the HTML file we created, but how do we get CSS styling in
there? Unfortunately it's not as simple as creating a CSS directory and linking
the stylesheets in our HTML file.

Let's go ahead and create the directory for static files now, which Django uses
for files like style sheets. In your app (remember we named it `collection` —
the folder that contains *models.py*), create a "static" directory, and within
it, add in directories for CSS, Javascript, and images. In the CSS directory,
add a blank *style.css* file.

{linenos=off}
```
$ cd collection
collection $ mkdir static
collection $ cd static
collection/static $ mkdir images
collection/static $ mkdir js
collection/static $ mkdir css
collection/static $ cd css
collection/static/css $ touch style.css
```

Head back into your *index.html* page. We'll need to tell Django that there are
static files used on this page, so at the top (like imports in *views.py* and
*urls.py*) add `{% load staticfiles %}`. Why the `{%`? We'll get to that soon.

Now that we can add static files to this template, add your CSS tag to the
`<HEAD>` just like this:

{title="index.html", lang="html+django"}
```
# leanpub-start-insert
{% load staticfiles %}
# leanpub-end-insert
<!doctype html>
<html>
    <head>
        <title>My Hello Web App Project</title>
# leanpub-start-insert
        <link rel="stylesheet" 
      href="{% static 'css/style.css' %}" />
# leanpub-end-insert
    </head>
    <body>
        <h1>Hello World!</h1>
        <p>I am a basic website.</p>
    </body>
</html>
```

We want to avoid using relative paths, such as `href="../css/style.css"`.  For
optimization purposes, someday later you might want to move the static files of
your project to a different server (such as Amazon's *Simple Storage Service*,
or *S3*), and by avoiding relative paths with Django's static files URL utility,
your CSS path will always use what's defined in your settings and will magically
work even  if you change your static file's locations.

At this point, feel free to add some stylesheet declarations to your *style.css*
file and check out *http://localhost:8000* again  (run `python manage.py
runserver` in your command line again if you need to do so).

{title="style.css", lang="css"}
```
body {
    background-color: cornflowerblue;
}
```

![](images/bluehelloworld.png) 

Nice! Feel free to update your CSS more at this point.

To link to any static file from a template, use the `{% static
'FILELOCATION/FILENAME.TYPE' %}` syntax, such as `{% static 'js/script.js' %}`
or `{% static 'images/logo.jpg' %}`. Keep in mind you still need the `IMG` HTML
tag when displaying images, for example: `<img src="{% static 'images/logo.jpg'
%}" alt=""/>`

That said, don't worry about this within your CSS files — feel free to use
relative paths because your static files should always be together anyways.

{lang="css"}
```
h1 {
    /* for example... */
    background-image: url(../images/logo.png);
}
```

Now you can add static files and style your website! Don't forget to commit your
work with git (review git on our git tips page here:
[http://hellowebapp.com/12](http://hellowebapp.com/12)).

Next up, Django has a bunch of awesome template utilities that'll elevate these
static HTML files and make them a lot more interesting (and fun to work with).