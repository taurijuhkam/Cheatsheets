## Django cheatsheet

### Django project and app quickstart
**Use virtualenv/virtualenvwrapper to isolate the environments**
* `virtualenv --python=python3.5 env_name` & `. ./env_name/bin/activate` - virtualenv
* `mkvirtualenv env_name` & `workon env_name` - virtualenvwrapper

**NB:** when working with virtualenvwrapper, be sure to have a conf file which you can source for
different projects/python versions etc. e.g:
```bash
#!/bin/bash
export WORKON_HOME=$HOME/.virtualenvs
export VIRTUALENVWRAPPER_PYTHON=`/usr/bin/python3.5`
export PROJECT_HOME=$HOME/my_projects
source /usr/local/bin/virtualenvwrapper.sh
```

### Boilerplate view
1. Start a new django project  - `django-admin startproject my_project`
2. Start a new app - `python manage.py startapp my_app`  
3. Write a function-based view  
    ```python
    # my_app/views.py
    from django.http import HttpResponse

    def index(request):
        return HttpResponse('Hello world')
    ```
4. Write an URLConf  
    ```python
    # my_app/urls.py
    from django.conf.urls import url

    from . import views

    urlpatterns = [
        url(r'^$', views.index, name='index'),
    ]    
    ```  
    ...and glue it to project urls
    ```python
    # my_project/urls.py
    from django.conf.urls import include, url
    from django.contrib import admin

    urlpatterns = [
        url(r'^url_path/', include('my_app.urls')),
        url(r'^admin/', admin.site.urls),
    ]
    ```
5. Check results  
    `python manage.py runserver` - http://localhost:8000/url_path/ should now display 'Hello world'

## Models
```python
# my_app/models.py
from django.db import models

class Person(models.Model):
    name = models.CharField(max_length=70)
    age = models.CharField(max_length=70)

    def __str__(self):              # __unicode__ on Python 2
        return self.full_name
```
[Model Field Reference](https://docs.djangoproject.com/en/1.11/ref/models/fields/#model-field-types)

**PS:** Always use `__str__(self):` and return something that identifies the object
**PPS:** For models to work, the app has to be added to the projects settings.py INSTALLED_APPS, by adding the class in
my_app.apps.MyAppConfig.  

[About the database API](https://docs.djangoproject.com/en/1.11/topics/db/queries/) -[Field lookups](https://docs.djangoproject.com/en/1.11/topics/db/queries/#field-lookups) 

## General Best Practices
* An app should do one thing and do it well
* For the sake of clarity, the app's name should be a plural version of the app's main model. e.g. Events (app) --> Event (model)
and if possible, follow the URL structure (and vice versa): http://www.my-project.com/events/
* Keep the number of Models per app as small as possible. If there are a lot of models then you are probably breaking rule #1
* For repeating fields, use Model Inheritance and Abstract Base Models (A class that inherits from models.Model and other models inherit from this new class) - tables will not be created for the ABM's, but for their children
