# Connecting to SpacialDB from Python

To connect to SpacialDB from your python framework, use [[Psycopg|http://initd.org/psycopg/]], the most popular PostgreSQL adaptor for Python. Check the documentation for information on how to [[install|http://initd.org/psycopg/install/]] this for your system. Psycopg can also be installed from the PyPI by:

```console
$ easy_install psycopg2
```

## Basic usage

The basic usage is common to all database adapters, requiring a connection string:

```python
import psycopg2

conn = psycopg2.connect("dbname=test user=postgres")
```

## Retrieve some data

```python
# Open a cursor to perform database operations
cur = conn.cursor()

# Get the time
cur.execute("SELECT NOW() AS when;")

# Fetch the next row of the query
result = cur.fetchone()

# Do something fancy with it
print result[0]

# Close cursor and connection
cur.close()
conn.close()
```

## Django example

We will use [[django|https://www.djangoproject.com/]] and the [[geodjango|http://geodjango.org/]] extension to build a little demo app. For installation instructions, please refer to the [[django wiki|https://docs.djangoproject.com/en/1.3/intro/install/]].

First of all, let's create a new django project called `spacial_example`:

```console
$ django-admin startproject spacial_example
```

In the next step we will configure django to use the geodjango backend and your database at SpacialDB. Just edit the `settings.py` file in your new project directory:

```python
DATABASES = {
    'default': {
        'ENGINE': 'django.contrib.gis.db.backends.postgis', # geodjango database backend
        'NAME': 'spacialdb0_yourUsername', # name of the database you want to use
        'USER': 'yourUsername', # your spacialdb username
        'PASSWORD': 'aSecret',
        'HOST': 'beta.spacialdb.com',
        'PORT': '9999'
    }
}
...
# add the admin and geodjango app to INSTALLED_APPS
INSTALLED_APPS = (
    'django.contrib.auth',
    'django.contrib.contenttypes',
    'django.contrib.sessions',
    'django.contrib.sites',
    'django.contrib.messages',
    'django.contrib.admin',
    'django.contrib.gis',
)
```

Let's create a new app in the project directory named places.

```console
$ django-admin startapp places
```

Add your app to the installed apps in the `settings.py` file.

```python
INSTALLED_APPS = (
    'django.contrib.auth',
    'django.contrib.contenttypes',
    'django.contrib.sessions',
    'django.contrib.sites',
    'django.contrib.messages',
    'django.contrib.admin',
    'django.contrib.gis',
    'places', # your app
)
```

Define a model which gets mapped into the database. To do so, edit `places/models.py` in your project directory.

```python
# use the extended models from geodjango
import models from django.contrib.gis.db

class Place(models.Model):
    tag = models.CharField(max_length=250) # just a string
    location = models.PointField() # a point geometry
    objects = models.GeoManager() # adds spatial requests

    def __unicode__(self):
        return self.tag
```

We now have to sync the database with our model.
```console
$ python manage.py syncdb
```

Let's add a place to the database. We use the built in shell for this.

```console
$ python manage.py shell

>>> from places.models import Place
>>> # get all places
>>> Place.objects.all()
[]
>>> # add a place to the db
>>>aPlace =  Place(location="POINT(23 42)", tag="a point")
>>>aPlace.save()
>>>Place.objects.all()
```

It's time to add a page to your site. The first step is to add the url pattern for the page. There will be an admin interface and a site serving information about all places. First, edit the `urls.py` file in your project directory.

```python
from django.conf.urls.defaults import *

from django.contrib import admin
admin.autodiscover()

urlpatterns = patterns('',
    (r'^admin/', include(admin.site.urls)),
    (r'^places/', include('places.urls'))
)
```

Next create/edit the `urls.py` file in your `app\` directory.

```python
from django.conf.urls.defaults import *

urlpatterns = patterns('places.views',
    (r'^$', 'index'),
)
```

Create a view which renders all the places. Define your view in `places/views.py`.

```python
from django.http import HttpResponse
from django.template import Context, loader
from places.models import Place

# define the index view
def index(request):
    # get all places
    allPlaces = Place.objects.all()
    # get template for index
    t = loader.get_template('places/index.html')
    # create a context with the data we need in the template
    c = Context({
        'places': allPlaces,
    })
    # render template and return data
    return HttpResponse(t.render(c))
```

Create a directory outside of your project directory for the templates you want to use in your project. Create a directory for the places templates called `places` inside this directory. Finally in your `settings.py` add its absolute path in the `TEMPLATE_DIRS`:

```python
TEMPLATE_DIRS = (
    # absolute path to the your templates directory
    '/tmp/templates/places'
)
```

Let's define the template we want to render. Create a file called `index.html` in the places template directory.
```html
{% if places %}
    <ul>
        {% for place in places %}
        <li>Tag: {{ place.tag }}<br>Longitude: {{place.location.x}}<br>Latitude: {{place.location.y}}</li>
        {% endfor %}
    </ul>
{% else %}
    <p> No places are available. Sorry...</p>
{% endif %}
```

Now you should be able to browse your places under [[http://localhost:8000/places|http://localhost:8000/places]].

Add places to the admin interface. Create an `admin.py` file in the `app\` directory. After this step you should be able to edit your places in the [[admin interface|http://localhost:8000/admin]].

```python
from places.models import Place
from django.contrib import admin

admin.site.register(Place)
```

What if we want all places in Europe? Let's add another view for all places in Europe. Edit the `views.py` file in the `app\` directory.

```python
from django.contrib.gis.geos import *

def europe(request):
    # create polygon for europe
    europe = Polygon.from_bbox((-30, 35, 50, 70))
    # get all places within europe
    places = Place.objects.filter(location__within=europe)
    t = loader.get_template('places/index.html')
    c = Context({ 'places': myPlaces })
    return HttpResponse(t.render(c))
```

Add the `urlpattern` for the view in `urls.py` file in the `app\` directory.

```python
from django.conf.urls.defaults import *

urlpatterns = patterns('places.views',
    (r'^$', 'index'),
    (r'^europe/$','europe'),
)
```

Insert some points into your database. You can do this either using the admin interface or the python shell. Check your results on [[http://localhost:8000/places/europe|http://localhost:8000/places/europe]].
