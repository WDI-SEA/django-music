# Django

## Learning Objectives
* Learn more about the Python programming language.
* Understand the differences between Django, and Express.
* Learn the basics of configuring a Django application.
* Write Views, Models, URLs, Forms, and Templates in Django.
* Build an App modeling Artists and Songs!

## Framing

Earlier in this course, you wrote Express apps in JavaScript. Today we are going to look at writing Django apps in Python. Django is a popular backend framework that is somewhere a bit more heavy duty Express. 

Django follows the mantra "explicit is better than implicit" (which is line 2 of ["The Zen of Python"](https://www.python.org/dev/peps/pep-0020/)), therefore, there isn't a lot of "magic" involved unless there is a really good reason for it. 

Instagram, The Washington Post, the Onion, Spotify, and Pinterest are all written in Django. In addition, YouTube, DropBox, and Reddit are all written in similar Python frameworks.

Instead of being MVC like Express, Django is a MVT framework.

* M - Models
* V - Views
* T - Templates

In practice this ends up being very similar, the terminology is just a bit different. The other big difference is that Django projects (which are what we think of as apps in Express) are subdivided into smaller apps. These are supposed to be reusable so you can plug them into a different project if needed in the future. For most small projects, you will just have one sub app (like in this class), but for really large projects you may have several. When you add plugins to Django projects, a lot of the time they will be apps themselves instead of normal libraries!

## Configuration

Let's start by making a directory for our project:
```bash
$ mkdir django-starter && cd django-starter
```

Let's also build a virtual environment. Virtual environments allow us to have multiple versions of Python on the same system so we can have different versions of both Python and the packages we are using on our computers.

```bash
$ pip3 install virtualenv
$ virtualenv .env -p python3
$ source .env/bin/activate
```

Let's also install some dependencies and save them. Django doesn't utilize a `package.json`. Instead, we just use a text file that lists all of our dependencies. Pip freeze saves the dependencies in our `virtualenv` to that file. 

```bash
$ pip3 install django
$ pip3 install psycopg2
$ pip3 freeze > requirements.txt
```

Django is, of course, the framework we are using. Psycopg2 allows us to use PostgreSQL within Django.

If you are downloading and running a Python project, you can usually install its dependencies with `pip3 install -r requirements.txt`.

Let's go ahead and create our project. Similar to express and react, we can run commands to generate some of our project for us.
```bash
$ django-admin startproject tunr_django
```

&#x1F535; **Activity**
```
* Take a look at the generated files!
* 4 minutes
```

In Django a Project is your whole backend application, which is split into smaller "Apps". Now that we've created our project, let's create our "base app.

```bash
$ django-admin startapp tunr
```

&#x1F535; **Activity**
```
* Take a look at the generated files!
* 4 minutes
```

By default, Django uses MySQL for its database. We're going to use PostgreSQL.

Create a database:
```bash
$ psql
> CREATE DATABASE tunr;
> CREATE USER tunruser WITH PASSWORD 'tunr';
GRANT ALL PRIVILEGES ON DATABASE tunr TO tunruser;
> \q
```

Then, in `tunr_django/settings.py` find the `DATABASE` constant dictionary. Let's edit it to look like this:

```python
DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.postgresql_psycopg2',
        'NAME': 'tunr',
        'USER': 'tunruser',
        'PASSWORD': 'tunr',
        'HOST': 'localhost'
    }
}
```

We must also include the app we generated. On the bottom line of the `INSTALLED_APPS` list, add `tunr`. Whenever you create a new app, you have to include it in the project. In (settings.py)

```python
INSTALLED_APPS = [
    'django.contrib.admin',
    'django.contrib.auth',
    'django.contrib.contenttypes',
    'django.contrib.sessions',
    'django.contrib.messages',
    'django.contrib.staticfiles',
    'tunr'
]
```

Now, in the terminal run `$ python3 manage.py runserver` and then navigate to `localhost:8000`. You should see a page welcoming you to Django!

From the terminal you should also see a warning that looks something like this: 

```bash
You have 13 unapplied migration(s). Your project may not work properly until you apply the migrations for app(s): admin, auth, contenttypes, sessions.
Run 'python manage.py migrate' to apply them.
```

We'll talk about migrations in a moment! 

`manage.py` contains a lot of management commands for Django -- kind of like how we run `npm` and then a command in Node. For now, let's start modeling some data!

## Models

First, lets create a Python class that inherits from the Django built in `models.Model` class. Let's also define the fields within it. We do so like this:

tunr/models.py
```python
class Artist(models.Model):
    name = models.CharField(max_length=100)
    nationality = models.CharField(max_length=100)
    photo_url = models.TextField()
```

Here, we are defining three fields (which will be represented as columns in our database): `name`, `photo_url` and `nationality`. `name` and `nationality` are character fields which means that we must add an upper limit to how many characters are in that database field. The `photo_url` will have unlimited length. The full listing of the available fields are [here](https://docs.djangoproject.com/en/1.11/ref/models/fields/).


Let's also add the magic method `__str__`. This method defines what an instance of the model will show up as by default. It will be really helpful for debugging in the future!

```python
class Artist(models.Model):
    name = models.CharField(max_length=100)
    nationality = models.CharField(max_length=100)
    photo_url = models.TextField()

    def __str__(self):
        return self.name
```

## Migrations 

SQL tables need to be clearly defined in terms of what columns are allowable. We also need to be able to change the tables mid project without risking losing existing data! We also want to be able to _version_ our database! Kind of like git, we want a way to keep track of all changes made to the database, and safely go _back in time_ to a previous version if we make a change that turns out to be not what we want!

Enter **migrations**. 

A migration is a file that specifies some _change_ to our Database Schema. Each migration has an ID that allows us to go back to that point in our Databases life, just like git! 

In order to migrate this model to the database, (in other words, in order to make the database _look like_ our model) we will run two commands. The first is:

```bash
$ python3 manage.py makemigrations
```
This will generate a migration file that gets all of its data from the code in the `models.py` file. You normally don't edit these files manually in Django. 

Let's also run:
```bash
$ python3 manage.py migrate
```
This will commit the migration to the database.

#### Foreign Keys

Let's also start filling out the Song model. We will define the class and then add a foreign key. We do so like this:

```python
class Song(models.Model):
    artist = models.ForeignKey(Artist, on_delete=models.CASCADE, related_name='songs')
```

The `related_name` refers to how the model will be referred to in relation to its parent -- you will see this in use later on. `on_delete` specifies how we want the models to act when their parent is deleted. By using cascade, related children will be deleted.

&#x1F535; **Activity**
```
* Add the rest of the Song model. Add `title`, `album` and `preview_url` fields, then create and run the migrations. 
* 4 minutes
```

### Admin Console

Before we get too far, let's also create a superuser for our app. Django has authentication (and authorization) right out of the box, so you don't have to write it yourself or add a plugin.

In the terminal, run:
```bash
$ python3 manage.py createsuperuser
```
Then fill in the information in the boxes that pop up!

So far in this class,  we have used seed files to add initial data to our databases. We can also do that in Django ([see this article](https://docs.djangoproject.com/en/1.11/howto/initial-data/)), but let's try something a little bit different instead.

Django has an admin dashboard built in, which gives us full CRUD functionality straight out of the box.

Let's set it up! In `tunr/admin.py`, add the following code:

```python
from django.contrib import admin
from .models import Artist

admin.site.register(Artist)
```

If you now navigate to `localhost:8000/admin`, you get a full admin view where you have full CRUD functionality for your model! Create two Artists here.

&#x1F535; **Activity**
```
* Import your songs model into your `admin.py` and register it!
* Create three songs through this dashboard. 
* 4 minutes
```

## Break Time!

### Django's ORM

Django has an ORM, similar to Mongoose in Express. Let's look at a few queries.

```python
# Select all of the artist objects in the database
Artist.objects.all()

# Get the artist with the id of 1 (can also do pk here which stands for primary key)
Artist.objects.get(id=1)

# Get the artist with the name "Aphex Twin", if there are two Aphex Twin's this will error!
Artist.objects.get(name="Aphex Twin")

# Get all the Artists who are from the UK
Artist.objects.filter(nationality="UK")

# Create an Artist with the following attributes, then save, commiting it to the database
richard = Artist(name="Afx Twin", photo_url="test.com", nationality="USA")
richard.save()

# Oops, we misspelled Aphex Twin's name! Let's change it and then commit to the DB
richard.name = "Aphex Twin"
richard.save()

# Let's add a song to the artist
song = Song(title="minipops 67", album="Syro", preview_url="test.com", artist=a)
song.save()

# Delete the song
song.delete()
```

These are some simple ones, but if you like Django, know that you can do some really cool things with it's ORM. For example:

```python
# This will return all Artists who's name starts with an A
Artist.objects.filter(name__startswith="A")

# This will return all the songs with the id's 1 and 2, excluding all those equal to or greater than 3
Song.objects.exclude(artist_id__gte=3)
```

If you want to access a REPL, run `$ python manage.py shell`. If you end up building your own Django application I would **highly** recommend using [django-extensions](https://github.com/django-extensions/django-extensions) so you can use IPython notebooks within Django to debug!

## Views

Now that we have data, let's write a simple view function to display it! Views are really similar to controllers in the other languages we have looked at so far. They pass data to our templates. 

In your view file, you may see that `render` is already imported. This function is super helpful, and it does exactly what it sounds like - it renders views!

`views.py`
```python
from django.shortcuts import render

from .models import Artist, Song

def artist_list(request):
    artists = Artist.objects.all()
    return render(request, 'tunr/artist_list.html', {'artists': artists})
```

Let's break this function down a bit. Let's first look at the declaration of a function. It looks like any other Python function! The only parameter to it is the request, which is what it sounds like. This is the HTTP Request dictionary. Then we are selecting all of the artists from the database into a QuerySet called artists. 

On the third line, we see that we are rendering a template. The first argument is the request argument, the second is the template that we want to render, and the third is a dictionary with the data we want to send to the view. In this case, that's the artist QuerySet with the key 'artists'.

### You Do: Song List Function
Write the view and the url to list all of the songs in the application.

## URLs

Let's go ahead at how we will access these views -- through the URLs! These work much like routes in Express. We use a method `path` which takes an argument of a url endpoing, and a View. 

Details of how the URL Dispatcher in Django works can be found in [the docs](https://docs.djangoproject.com/en/2.0/topics/http/urls/)

Let's look at the existing `urls.py` in the `tunr_django` directory. In there, let's add a couple things. 

**Note:** There was a significant change from Django 1.9 to Django 2.0. You might see instead of `path`, `url`, and a more complicated set of arguments using Regex! `url` will still work in Django 2.0, in fact, the first route set up by our generated app uses it. Please reference the docs above!

```python
from django.urls import include, path
from django.contrib import admin

urlpatterns = [
    path('admin/', admin.site.urls),
    path('', include('tunr.urls'))
]
```

We are adding an import - `include` so that we can include other url files in our main one. We are doing this in order to make our app more modular -- again these "mini apps" in Django are supposed to plug into another parent app if needed.

Let's write our urls for our app in another file called `tunr/urls.py`.

tunr/urls.py
```python
from django.urls import path
from . import views

urlpatterns = [
    path('', views.artist_list, name='artist_list'),
]
```

This is going to be the root URL. This is similar to the "/" URL in other languages -- it's a url that starts, ends, and has nothing in between. This way of doing URLs is great because they are super explicit. We no longer have to reorder or rename URLs in order to make them work!

This URL's second argument is the view function this route is going to match up with. This will go in the view file. Thirdly, we are going to used a named parameter. This is going to be used in our templates in order to link from one page to another.


#### You Do: Add a URL for the 'song_list' (2 mins)

Now that we have two URLs, let's finish up by writing the templates to render our views!


### Templates and Django Templating Language

In previous classes, we've seen the templating languages JSX and EJS. Django also has its own! It looks a lot like EJS  in that it uses a bunch of curly braces. Let's add the following code:

`templates/tunr/artist_list.html`
```html
<h2>Artists <a href="">(+)</a></h2>
<ul>
    {% for artist in artists %}
        <li>
            <a href="">{{ artist.name }}</a>
        </li>
    {% endfor %}
</ul>
```

Right now, we will keep our `href`'s blank until we have more routes. The Django template here loops through our QuerySet of artists, rendering the name of each. The distinction between `{{}}` and `{%%}` usually is the difference between rendering or just running code (i.e. if's or for's). The one exception is with url's - which we will see later on today. 


### You Do: Create List View for Songs
Loop over each song and display them using Django Templates


### We Do: Artist Show

Let's look at another route -- let's do show this time. 

```python
def artist_detail(request, pk):
    artist = Artist.objects.get(id=pk)
    return render(request, 'tunr/artist_detail.html', {'artist': artist})
```
This function is really similar to the list view, this time we just are selecting one artist instead of all of them.  We receive a second parameter to the function -- the primary key of the artist we want to display. Let's look at where that is coming from and hook up the URL to the view in `urls.py`.

```python
    path('artists/<int:id>', views.artist_detail, name='artist_detail'),
```
Whith the show url, we get to see the beauty of RegEx's for Django's url's. This Regex is allowing us to access a variable called `pk` from the URL. The parentheses and P say that everything within those parentheses is a "capture group". The `d` is saying that the `pk` variable must be a number (using d for digit). In Django we use `pk` as an alternate term for `id`. In the database, primary keys are the unique ids for each row, and Django adopts that terminology by convention.

Finally, let's write our template.

`/tunr/artist_detail.html`
```html
<h2>{{ artist.name }} <a href="">(edit)</a></h2>
<h4>{{ artist.nationality }}</h4>

<img src="{{ artist.photo_url }}" alt="" class="artist-photo">

<h3>Songs <a href="">(+)</a></h3>
<ul>
    {% for song in artist.songs.all %}
        <li>
            <a href="">{{ song.title }}-{{ song.album }}</a>
        </li>
    {% endfor %}
</ul>
```

Here, we are showing some attributes of the artist object, then we are looping through the songs attached to that artist.

Let's take a step back to our `artist_list.html`.
Before, we omitted the links to to the next page, but let's look at linking to our show page now. On line 5, let's insert a url.

In the `href` attribute, let's add a URL tag. It looks like this:
```html
<a href="{% url 'artist_detail' pk=artist.pk %}">
    {{ artist.name }}
</a>
```
First, we are going to use the name of the URL that we declared back in the `url` file. Then we need to pass the primary key of that artist into that url. We do so like the above.

### You Do Song Show

Write A Django template to display some song data. 

## We Do: base.html and CSS
Right now we have two views, but they are really ugly. Let's do something about that! Let's first add a `base.html` file in the `tunr` views. It's at first going to look exactly like any other HTML base template we normally have, with one exception. We will have a block where we want our template to go. We will name this block content. We could have multiple blocks if we wanted. In that case they would be named other things -- say "title" or "header" instead of "content". 

```html
<html>
    <head>
        <title>Tunr</title>
    </head>
    <body>
        <h1>Tun.r</h1>
        <nav>
            <a href="/songs">Songs</a>
            <a href="/">Artists</a>
        </nav>
        {% block content %}
        {% endblock %}
    </body>
</html>
```
Now let's hook this up to our templates -- which ends up looking like this:

```html
{% extends 'tunr/base.html' %}

{% block content %}
    <h2>Artists <a href="{% url 'artist_create' %}">(+)</a></h2>
    <ul>
        {% for artist in artists %}
            <li>
                <a href="{% url 'artist_detail' pk=artist.id %}">{{ artist.name }}</a>
            </li>
        {% endfor %}
    </ul>
{% endblock %}
```
Each template is going to be extending the base, and our current content will go with in the `content` block of code. 

Now let's add styling. By default, Django is going to host our static files in the `static` folder. Let's add a `css` subdirectory, and a `tunr.css` file within it.

Let's copy this CSS:
```css
body {
    font-family: 'Helvetica Neue', sans-serif;
    max-width: 50em;
    margin: auto;
    padding: 2em 1em;
}

nav a {
    border: 1px solid black;
    margin: .5em;
    padding: .5em;
    background-color: #eeeeee;
}

nav a:hover {
    background-color: orange;
    color: blue;
}

a,
a:visited {
    text-decoration: none;
    color: blue;
}

a:hover {
    background-color: #ccc;
}

ul {
    list-style-type: none;
}

li {
    margin: .25em;
}

h1 {
    font: inherit;
    color: inherit;
    letter-spacing: -.05em;
    text-decoration: none;
    border-bottom: 1px solid black;
}

h2>a {
    font-size: .75em;
}

input {
    display: block;
    margin: 5px 0 20px 0;
    padding: 9px;
    border: solid 1px black;
    width: 300px;
    background: whitesmoke;
}

input[type=submit],
a.delete {
    width: auto;
    padding: 9px 15px;
    background-color: gray;
    border: 0;
    font-size: 14px;
    color: #FFFFFF;
}

a.delete {
    background-color: red;
}

.artist-photo {
    width: 400px;
}

span.nationality {
    font-size: .5em;
}

.user-info {
    float: right;
}

a.fav {
    text-decoration: none;
    color: red;
}

a.no-fav {
    text-decoration: none;
    color: black;
}

.fav:visited,
.no-fav:visited {
    text-decoration: none;
}
```

Finally, let's add this into our `base.html`. 
```html
{% load staticfiles %}
<html>
    <head>
        <title>Tunr</title>
        <link rel="stylesheet" href="{% static 'css/tunr.css' %}">
    </head>
    <body>
        <h1>Tun.r</h1>
        <nav>
            <a href="/songs">Songs</a>
            <a href="/">Artists</a>
        </nav>
        {% block content %}
        {% endblock %}
    </body>
</html>
```
On line 1, we are telling Django to load the static files onto the current page. Then in the `link` tag, we can refer to our static file like the above. This may seem a bit messy, but it really helps when you deploy your app, especially if you want to host your static files on a separate server. 

### We Do: Artist Create

So far we've just shown our artists. Let's now create a new one! First, let's make a file called `forms.py`. This is going to be where we make our forms using Python code! 

We do so like this:
```python
from django import forms
from .models import Artist, Song

class ArtistForm(forms.ModelForm):

    class Meta:
        model = Artist
        fields = ('name', 'photo_url', 'nationality',)

```

First, we will create a class to house our form. Inside of it we will declare another class. The Meta class within the form contains meta data we have to describe the form. In this case, the model attached to the form and the fields we want to include in our form. Note that the fields are in parenthesis instead of square brackets - they are in a tuple instead of a list! Tuples are like lists but they are immutable -- they can't be changed.

The rationale for the `Meta` class within the class is that it contains information describing the class that isn't specific to a particular instance, rather it just contains configuration details (bonus: [you can do this for models as well](https://docs.djangoproject.com/en/1.11/ref/models/options/)). For clarification (and maybe interview) purposes, this is different than a Python `metaclass`. They deal with meta-programming in Python! 

Now, in our `views.py` file, let's make a view function. 
```python
def artist_create(request):
    if request.method == 'POST':
        form = ArtistForm(request.POST)
        if form.is_valid():
            artist = form.save()
            return redirect('artist_detail', pk=artist.pk)
    else:
        form = ArtistForm()
    return render(request, 'tunr/artist_form.html', {'form': form})
```

We must also change the first line of our file to `from django.shortcuts import render, redirect` so that we have redirects available to us.

Let's break down this function: instead of having different functions that handle different types of requests, we can handle multiple types within the same function in Django. So, in the first line we check to see if our request is a post request. If it is, we will fill in our form with data from the post request, check if the form is valid, if it is, then we will save the new artist and redirect to it's detail view. If it errors, then we will render the artist form with those errors. 

If instead the request method is 'get', we just create an instance of the form without any pre-filled data, and then we will render the form template.

Now let's make the template. Since we already declared the fields we want in our form in the `forms.py file, we can just do this:

```html
{% extends 'tunr/base.html' %}

{% block content %}
    <h1>New Artist</h1>
    <form method="POST" class="artist-form">
        {% csrf_token %}
        {{ form.as_p }}
        <button type="submit" class="save btn btn-default">Save</button>
    </form>
{% endblock %}
```

We declare our form tags in html, we need to have a `csrf_token`, which is a security feature of Django. Then we can just insert our form like `{{ form.as_p }}`. The `.as_p` just formats the form nicely, you could also just do `{{ form }}` but it would be ugly. 

We also need a submit button and then we are good! Errors are handled for us in-line!

Finally, just add a url, in the ```urls.py``` file!
```python
   path('artists/new', views.artist_create, name='artist_create'),
```

### You Do: Song Create

Your turn! Do the same as above but for the song creation form.

### We Do: Artist Edit

Django makes forms really modular, all we have to do is add a new view function and a url to make our form edit instead of create!

```python
def artist_edit(request, pk):
    artist = Artist.objects.get(pk=pk)
    if request.method == "POST":
        form = ArtistForm(request.POST, instance=artist)
        if form.is_valid():
            artist = form.save()
            return redirect('artist_detail', pk=artist.pk)
    else:
        form = ArtistForm(instance=artist)
    return render(request, 'tunr/artist_formƒ.html', {'form': form})
```

Here, the only additionally thing we are doing is adding the instance of the artist as a named parameter to our ArtistForm class. Voilà! We have an edit form now! We are rendering the same template and everything!

Let's also add a new url:

```python
  path('artists/<int:pk>/edit', views.artist_edit, name='artist_edit'),
```

### You Do: Song Edit
Do the same thing for the song edit form!

### We Do: Artist Delete
Delete functions are really simple as well.

```python
def artist_delete(request, pk):
    Artist.objects.get(id=pk).delete()
    return redirect('artist_list')
```
We just need to find an artist, delete it, and then redirect to the index page.

Let's add its url: 
```python
path('artists/<int:pk>/delete', views.artist_delete, name='artist_delete'),
```

### You Do: Song Delete
Do the same thing for Songs!

## Authentication
Authentication in Django is built in out of the box -- no need for Passport or Devise! We already created our first user at the beginning of the lesson with `createsuperuser`. Now, lets make our app require authentication for some views.

In the `tunr_django/urls.py` let's add a few things:

```python
from django.conf.urls import include, path
from django.contrib import admin
from django.contrib.auth import views as auth_views

urlpatterns = [
    path('admin/', admin.site.urls),
    path('', include('tunr.urls')),
    path('accounts/login/', auth_views.login, name='login'),
    path('accounts/logout/', auth_views.logout, name='logout'),
]
```
Only a few things are changing here, we are importing the auth views and then adding urls for them. This will give us all of the login/logout functionality!

Let's add a login form. Let's create a new template folder called `registration` to follow Django convention. Let's add a `login.html` file within it. The form should look like this:

```html
{% extends 'tunr/base.html' %}

{% block content %}
<h2>Login</h2>
<form method="post">
    {% csrf_token %} {{ form.as_p }}
    <button type="submit">Login</button>
</form>
{% endblock %}
```

Let's link to them in our `base.html`.

```html
<nav>
    <a href="/songs">Songs</a>
    <a href="/">Artists</a>
    <div class="user-info">
        {% if user.is_authenticated %}
            Welcome, {{ user.username }}
            <a href="{% url 'logout' %}">Signout</a>
        {% else %}
            <a href="{% url 'login' %}">Login</a>
        {% endif %}
    </div>
</nav>
```

The `user` variable is accessible to us in any view in Django. We just have to check if the user is authenticated to see which button we should view!

Let's edit our configuration in `settings.py` in order add a page for users to redirect to upon login. 

```python
LOGIN_REDIRECT_URL = 'artist_list'
```

This constant says that when a user logs in, we want them to go to the "artist_list" view.

Now let's make it a requirement to be logged in to see some views -- maybe the create, update and delete ones. On line two, add this line: 

```python
from django.contrib.auth.decorators import login_required
```

This will allow us to use what is called a `decorator` on our chosen functions. A decorator is a function called on a function in order to change its behavior. In this case, it adds an if statement to each function: if the user is logged in continue with the view logic, if not then redirect to the login page. Let's look at the syntax for using them:

```python
@login_required
def artist_create(request):
    if request.method == 'POST':
        form = ArtistForm(request.POST)
        if form.is_valid():
            artist = form.save()
            return redirect('artist_detail', pk=artist.pk)
    else:
        form = ArtistForm()
    return render(request, 'tunr/artist_form.html', {'form': form})
```

All we added was the `@login_required`! Add it to the five other views we want secured!

## Many to Many

Finally, let's add a many to many field in our app. Let's go back to `models.py`.

```python
class Favorite(models.Model):
    user = models.ForeignKey('auth.User', related_name='favorites')
    song = models.ForeignKey(Song, related_name='favorites')
```

Here we have two foreign keys between the User model and the Song one. When our related models are in separate files, it is usually easier to refer to them in string form, like we seee in the User example.

Let's add some URL's and Views:

`tunr/urls.py`
```python
path('favorites/<int:song_id>/create', views.add_favorite, name='add_favorite'),
path('favorites/<int:song_id>/remove', views.remove_favorite, name='remove_favorite')
```

```python
@login_required
def add_favorite(request,song_id):
    song = Song.objects.get(id=song_id)
    Favorite.objects.create(song=song, user=request.user)
    return redirect('artist_detail', pk=song.artist.pk)


@login_required
def remove_favorite(request, song_id):
    song = Song.objects.get(id=song_id)
    Favorite.objects.get(song=song, user=request.user).delete()
    return redirect('artist_detail', pk=song.artist.pk)
```

These mirror what we have done so far in this class -- we just find the specific favorite and create or delete it.

The template becomes a bit tricker -- we need to write a custom Django Templating tag to select the desired favorite from within the template. Let's create a new folder called `templatetags`. Within there create two files: `__init__.py` and `favorite_tags.py`. The init file tells Python that our current directory is part of the package (so we can import the files using dot notation), and then the favorite_tags file will contain our custom tag. 

`templatetags/favorite_tags.py`
```python
from django import template

register = template.Library()

@register.filter(name='has_favorite')
def has_favorite(song, user):
    return song.favorites.filter(user=user).exists()
```
We need to register a new template using another decorator. We just want to check to see if the song has a favorite from the current user.

Then, in `artist_detail.html`, below the `extends` tag, add `{% load favorite_tags %}`. This will allow us to access our custom favorite tag.

In the body of the template, add this code. 
```html
{% for song in artist.songs.all %}
    <li>
        <a href="{% url 'song_detail' pk=song.pk %}">{{ song.title }}-{{ song.album }}</a>
        {% if user.is_authenticated %}
            {% if song|has_favorite:user %}
                <a href="{% url 'remove_favorite' song_id=song.id %}" class="fav">&hearts;</a>
            {% else %}
                <a href="{% url 'add_favorite' song_id=song.id %}" class="no-fav">&hearts;</a>
            {% endif %}
        {% endif %}
    </li>
{% endfor %}
```
We are now using our custom template tag! The first argument goes before the pipe, the name of the tag goes after it, and then the second argument comes after a colon. The syntax is kind of funky, but it allows us to check and see if a song is favorited by a user pretty cleanly. Then, we link to our favorite/unfavorite functions accordingly.

## Questions

## More Resources
* [Django Girls](https://www.gitbook.com/book/djangogirls/djangogirls-tutorial)
* [Django Girls Extensions](https://www.gitbook.com/book/djangogirls/django-girls-tutorial-extensions/details)
* [Documentation](https://docs.djangoproject.com/en/1.11/)
