# Adding Dynamic Data

Now that we know what we can do with templates and know how variables can be
passed in from the view, we'll head back to *models.py* to actually define the
dynamic information that we want to add to and store in our database —
basically, the *things* in our *collection*.

## Your local database

Back when we were setting up Django (the instructions that live in our GitHub
online repository: [http://hellowebapp.com/16](http://hellowebapp.com/16)),
Django already created our database for us using *SQLite3*.

We're using SQLite3 on our computer because it's quick and easy, but it's not an
appropriate database if you were to launch your app — it was built to support
only a single user at a time. Fine for development, but research other database
solutions before launching your web app. More on that later, don't worry.

### Creating a superuser — you!

We're going to be using Django's super handy admin, which means we need to
create an account for yourself so you can pop in any time to check out the state
of your web app.

Type this into your command line:

{linenos=off}
```
$ python manage.py createsuperuser
```

This'll prompt you to create an admin user — choose any username, email address,
and password that you like. It can always be changed later.

## Setting up your model

Remember that this tutorial is teaching you to build a "collection of things,"
which is going to seem complicated now that we're in the model where we actually
define what our thing is.

Thankfully any "collection of things" generally would need the same couple of
basic properties (keeping with our MVP philosophy):

* *Name* of the thing.
* *Description* of the thing.
* *Slug* of the thing, used in its future URL. 

We can add a lot more info to our `Thing` eventually, but we're going to start
with this for now, keeping everything minimal to start.

The first two attributes we're defining are fairly obvious to why we need them.
We're also going to add a slug because eventually we're going to build
individual pages for these objects, and it's a lot easier to have the slug for
the object already defined in the database for creating URLs in the future.

URLs can't have any spaces and are best limited to letters, numbers,
underscores, or hyphens — so the URL for something named "This Beautiful
Something" would have a slug of `this-beautiful-something`. We're also going to
make this field automatically fill itself out — more on that soon.

In *models.py*, we're going to build a model for the information we want to
store. Here's where you're going to want to customize the titles — I'll give the
generic example first, and then a customized example after to make it easy.

{title="models.py", lang="python"}
```
from django.db import models

# leanpub-start-insert
class Thing(models.Model):
    name = models.CharField(max_length=255)
    description = models.TextField()
    slug = models.SlugField(unique=True)
# leanpub-end-insert
```

If you were building a directory of invitation designers, you might want to
change the class from `Thing` to `Profile` — so, `class Profile(models.Model)`.

For a list of awesome surfboards, you could change it to `Surfboard`— so, `class
Surfboard(models.Model)`.

You get the drift.

As for the rest of the model, this is basically what we're defining:

* A *name* field (with a character limit of 255 characters — you're required to
    define a limit and 255 is a good basic limit.)
* A *description* field with a text field (which doesn't require a limit — think
    of these like HTML forms, such `<input>`s versus `<textareas>`.)
* A *slug* field using Django's SlugField
    ([http://hellowebapp.com/17](http://hellowebapp.com/17)) that'll add some
    automatic checking to make sure it's always in the right format, and ensures
    uniqueness against other objects because we use it in each unique object's
    URL.

## Finishing setting up your database with migrations

Think of the database like a giant spreadsheet filled with information about our
users — names, descriptions, etc. One day we decide to add another column of
information for all of our users — a location. How do we fill in that column for
those pre-existing users? We *could* do it by hand. What if the spreadsheet had
thousands of rows? What then?

Thankfully Django comes included with a database migrations utility that both
tracks changes to the database when new fields are added, edited, or removed;
and gracefully updates the existing database to make sure everything plays
nicely with one another.

We'll going to create our *initial migration*. This is basically the starting
point to help Django track the changes you make to your database, so when you
make new changes, Django will know exactly what changed and will migrate the
information correctly to the new database plan (known as a schema).

We need to tell Django we're creating the initial migration, so in your command
line (in the same folder as *manage.py*), type this in:

{linenos=off}
```
$ python manage.py makemigrations
```

And this is what you should see as a result:

{linenos=off}
```
Migrations for 'collection':
  0001_initial.py:
    - Create model Thing
```

We've created the migration file, but we haven't applied it.  Run `python
manage.py migrate` next:

{linenos=off}
```
$ python manage.py migrate
Operations to perform:
  Apply all migrations: admin, contenttypes, collection, auth, sessions
Running migrations:
  Applying collection.0001_initial... OK
```

Like above, we're running the `migrate` command, telling Django to search for
new migrations and apply them to the existing database tables.

Our database is now set up and we can start adding some information!

## Using the Django admin

The other huge advantage Django has as a Python framework is the included visual
administration system, commonly called simply the "admin." This'll allow you to
see and update all information stored in your database in your browser without
having to log into your database using the command line.

We need to tell the admin to display the information in our `Thing` model (or
whatever you called it). The admin doesn't grab info from our model
automatically.

Within your `collection` app folder, open up *admin.py* and add the following
lines:

{title="admin.py", lang="python"}
```
from django.contrib import admin

# import your model
# leanpub-start-insert
from collection.models import Thing
# leanpub-end-insert

# and register it
# leanpub-start-insert
admin.site.register(Thing)
# leanpub-end-insert
```

This is the bare minimum that you'll need to get your new model to show up in
the admin. Let's add one more extra bit just to make it more awesome.

{title="admin.py", lang="python"}
```
from django.contrib import admin
# import your model
from collection.models import Thing

# set up automated slug creation
# leanpub-start-insert
class ThingAdmin(admin.ModelAdmin):
    model = Thing
    list_display = ('name', 'description',)
    prepopulated_fields = {'slug': ('name',)}
# leanpub-end-insert

# and register it
# leanpub-start-insert
admin.site.register(Thing, ThingAdmin)
# leanpub-end-inserted
```

Don't forget to add `ThingAdmin` to your register statement at the bottom of
your file!

First, `list_display` tells Django that we want the name and description of our
fields to show up in the admin.

We set up our model to require a slug, but we don't want to fill it out manually
— much better to have it auto-created by Django magic. The `ThingAdmin` block
basically says that we're going to use the model `Thing` and we're going to
prepopulate the slug field based off the name field when someone types the name
of a `Thing` in the admin. You'll see this in a second.

Now if you go back to your browser and go to *http://localhost:8000/admin*, you
can log in using the admin username and password that you made earlier when
creating a superuser. 

![](images/admin.png) 

Voilà: a visual representation of your database information — your admin panel.

![](images/admin_loggedin.png) 

As you can see, we have links for a `User` database as well as an area for our
new app (`Collection`) and a link for our models defined in the app (so far,
only `Thing`). 

Click on Users and you can see the entry for your admin user account (you!), and
if you click on that entry, you can update any of the information on that `User`
account.

![](images/admin_admin.png) 

Here's where you can see that Django's included `User` accounts really saves us
some time: The password on the account is hashed for us, which is kind-of like
encryption but one-way and can't be reversed. This is important because if the
database ever leaks, hackers won't be able to reverse our users' original
passwords. This would be especially-bad because many folks use the same password
on many sites. Staying up-to-date with the latest security practices is a
full-time job, but Django takes care of these decisions for us. Thanks, Django!

If you back up a few screens to check out the Things area, it's empty because we
haven't added any new Things to our site yet — or people, or surfboards, or
whatever you decided to call your model. Let's add our first one.

Back out of the User area and click on "Things" (or whatever you named your
model). At the top right of the page, there's a button to add objects to your
database.

![](images/admin_addthing.png) 

![](images/admin_addthing_after.png) 

As you can see, this is a handy place for us to see, add, change, and remove any
account added to our website without logging into the database through the
command line. Our slug was also auto-created, saving us valuable time. Win!

Add a couple of more test objects to your database.

![](images/admin_things.png) 

Customers can't see the admin though (and really shouldn't ever — this panel
should only be used for administrative use). 

Next up, let's get our database information showing up in the templates. Don't
forget to commit your work!