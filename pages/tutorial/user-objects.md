# Associating Users with Objects

These new "users" can't create and update `Thing`s yet in our database.
We're going to assume a one-to-one database relationship between users and
`Thing`s — that is users can only create and update one `Thing`. This works best
for schemes like "profiles in a directory" but less so for "items in a store."
For the sake of brevity, we're going to stick with the former but will give some
resources later in the book if you're building something where users will need
to have multiple `Thing`s.

## Changing our model so `User`s can own a `Thing`

First thing we need to do is tie our `Thing` model to our `User`s model, so
every `Thing` is owned by one and only one `User`. 

Update your `Thing` model (or whatever you called it) to the following:

{title="models.py", lang="python"}
```
# don't forget to import this
# leanpub-start-insert
from django.contrib.auth.models import User
# leanpub-end-insert
from django.db import models

class Thing(models.Model):
    name = models.CharField(max_length=255)
    description = models.TextField()
    slug = models.SlugField(unique=True)
    # the new line we're adding
# leanpub-start-insert
    user = models.OneToOneField(User, blank=True, null=True)
# leanpub-end-insert
```

We're in a tiny bit of a pickle. We're saying that each `Thing` needs to have a
unique `User`. Remember the migrations stuff we did earlier? We'll need to
migrate the database to this new schema. Visualize a spreadsheet of `Thing`s
with all their attributes — we need to add a new column for the `Thing`'s
`User`, and fill that in for each row. 

To make this easy, we're going to tell Django that it's okay if the `Thing`
doesn't have a `User` assigned to it — that's the `blank=True, null=True` stuff
we added. That way we don't have to worry about filling in the previous objects
created, just our future ones.

### Migrating your database

Create the migration by running the command below:

{linenos=off}
```
$ python manage.py makemigrations
Migrations for 'collection':
  0002_thing_user.py:
    - Add field user to thing
```

And then apply the migration...

{linenos=off}
```
$ python manage.py migrate
Operations to perform:
  Synchronize unmigrated apps: registration
  Apply all migrations: admin, contenttypes, collection, auth, sessions
Synchronizing apps without migrations:
  Creating tables...
  Installing custom SQL...
  Installing indexes...
Running migrations:
  Applying collection.0002_thing_user... OK
```

Now your `Thing`s can have a `User` owning them!

## Updating your registration flow

You're almost done with this doozy of a chapter. The last thing we need to do is
set up an additional registration page so when a new user signs up, they'll also
set up their `Thing`.

First, add a couple new URLs to our *urls.py* file:

{title="urls.py", lang="python", line-numbers=on, starting-line-number=10}
```
# add this to the top
# leanpub-start-insert
from collection.backends import MyRegistrationView
# leanpub-end-insert
{title="urls.py", lang="python", line-numbers=on, starting-line-number=45}
# leanpub-start-insert
url(r'^accounts/register/$', 
    MyRegistrationView.as_view(),
    name='registration_register'),
url(r'^accounts/create_thing/$', views.create_thing, 
    name='registration_create_thing'),
# leanpub-end-insert
url(r'^accounts/', 
    include('registration.backends.default.urls')),
url(r'^admin/', admin.site.urls),
```

Then head back over to *views.py* to create the new view:

{title="views.py", lang="python", line-numbers=on,starting-line-number=3}
```
# add at the top
# leanpub-start-insert
from django.template.defaultfilters import slugify
# leanpub-end-insert
```

{title="views.py", lang="python", line-numbers=on,starting-line-number=52}
```
# add below your edit_thing view
# leanpub-start-insert
def create_thing(request):
    form_class = ThingForm

    # if we're coming from a submitted form, do this
    if request.method == 'POST':
        # grab the data from the submitted form and
        # apply to the form
        form = form_class(request.POST)
        if form.is_valid():
            # create an instance but don't save yet
            thing = form.save(commit=False)

            # set the additional details
            thing.user = request.user
            thing.slug = slugify(thing.name)

            # save the object
            thing.save()

            # redirect to our newly created thing
            return redirect('thing_detail', slug=thing.slug)

    # otherwise just create the form
    else:
        form = form_class()

    return render(request, ‘things/create_thing.html', {
        'form': form,
    })
# leanpub-end-insert
```

We're going to be reusing the form (`ThingForm`) we made before! Next, we'll
need to create the template. Create the file:

{linenos=off}
```
$ cd collection/templates/things
collection/templates/things $ touch create_thing.html
```

Then fill it out:

{title="create_thing.html", lang="html+django"}
```
{% extends 'base.html' %}
{% block title %}
    Create a Thing - {{ block.super }}
{% endblock title %} 

{% block content %}
    <h1>Create a Thing</h1>
    <form role="form" action="" method="post">
        {% csrf_token %}
        {{ form.as_p }}
        <input type="submit" value="Submit" />
    </form>
{% endblock %}
```

Very last thing we need to do is tell django-registration-redux to go to this
form's page after successful registration instead of the registration complete
page.

We're going to accomplish this by *subclassing* a portion of
django-registration-redux. This is a powerful way to override bits of code in
open-source plugins, so you can change only the bit you want to change, rather
than rewriting the entire plugin.

We're going to put this piece of overwritten code in its own file. In your
`collection` directory, create a file called `backends.py`:

{linenos=off}
```
$ cd collection
collection $ touch backends.py
```

Here are the contents of the new *backends.py*:

{title="backends.py", lang="python"}
```
from registration.backends.simple.views import RegistrationView

# my new registration view, subclassing RegistrationView
# from our plugin
class MyRegistrationView(RegistrationView):
    def get_success_url(self, request, user):
        # the named URL that we want to redirect to after
        # successful registration
        return ('registration_create_thing')
```

How did we know about this? Check out django-registration-redux's documentation
on views ([http://hellowebapp.com/26](http://hellowebapp.com/26)) — this page
lists out all the classes included in django-registration-redux and what is
recommended to subclass. You can also browse the code on the
django-registration-redux GitHub repository
([http://hellowebapp.com/27](http://hellowebapp.com/27)).

Check out your web app — users can now register for your website, which prompts
them to create a `Thing`, and after successful registration, plops them into the
newly created `Thing` page with a link to edit it.

### Update permissions to edit objects

We probably want to make it so the owner of a `Thing` can edit only his or her
respective `Thing`. Update your `Thing` template so the link is hidden to anyone
who isn't logged in:

{title="thing_detail.html", lang="html+django"}
```
{% extends 'base.html' %}
{% block title %}
    {{ thing.name }} - {{ block.super }}
{% endblock title %}

{% block content %}
    <h1>{{ thing.name }}</h1>
    <p>{{ thing.description }}</p>
    # leanpub-start-insert
    {% if user == thing.user %}
        <p>
            <a href="{% url 'edit_thing' slug=thing.slug %}">
                Edit me!
            </a>
        </p>
    {% endif %}
    # leanpub-end-insert
{% endblock content %}
```

We also want to add some additional security precautions to our view to make
sure that we let non-owners edit information that isn't theirs. 

Update your *views.py*:

{title="views.py", lang="python", line-numbers=on,starting-line-number=2}
```
# leanpub-start-insert
from django.contrib.auth.decorators import login_required
from django.http import Http404
# leanpub-end-insert
```
    
{title="views.py", lang="python", line-numbers=on,starting-line-number=28}
```
# leanpub-start-insert
@login_required
# leanpub-end-insert
def edit_thing(request, slug):
    # grab the object...
    thing = Thing.objects.get(slug=slug)

    # leanpub-start-insert
    # make sure the logged in user is the owner of the thing
    if thing.user != request.user:
        raise Http404
    # leanpub-end-insert

    # set the form we're using...
    form_class = ThingForm

    # if we're coming to this view from a submitted form,
    # do this
    if request.method == 'POST':
        # grab the data from the submitted form and
        # apply to the form
        form = form_class(data=request.POST, instance=thing)
        if form.is_valid():
            # save the new data
            form.save()
            return redirect('thing_detail', slug=thing.slug)
    # otherwise just create the form
    else:
        form = form_class(instance=thing)

    # and render the template
    return render(request, 'things/edit_thing.html', {
        'thing': thing,
        'form': form,
    })
```

We're adding two important security measures — adding a *decorator* to our view
(`@login_required`), provided by Django, that basically says, "The only people
who can access this view are ones who are logged in." The second thing we're
doing is checking the `Thing` we're grabbing the edit page for, and creating a
404 "Page not found" error if the logged-in user isn't the owner of the `Thing`
instance. You, as admin, can still update/edit info for your users from your
admin panel, but this prevents users from editing other users' information. 

Our longest, most intense chapter yet! I hope this chapter gave you a better
idea on how you can extend open-source plugins, update them to satisfy your
needs, and how to keep data secure on your website. 

*One last thing* — we set up the templates to do a password update for logged in
users (that's the *password_change* templates we set up before). Can you figure out how to add
that yourself?

Commit your work before moving forward!