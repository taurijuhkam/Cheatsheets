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

* Use URL-namespaces to more easily reference the url's where needed
```python
urlpatterns += [
    url(r'^tastings/', include('tastings.urls', namespace='tastings')),
]

# In views:
def get_details(self):
    return reverse('tastings:detail', #tastings.url.detail
    kwargs={'pk': self.object.pk})

# In templates
{% url 'tastings:detail' taste.pk %}
```



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

* **Fat models** - Keep data-related code (e.g. getting some filtered data, complex inserts) in models, not in views or templates.
But do not create god objects - if applicable, move to (stateless!!) helper functions etc.

[About the database API](https://docs.djangoproject.com/en/1.11/topics/db/queries/) -[Field lookups](https://docs.djangoproject.com/en/1.11/topics/db/queries/#field-lookups) 

## Views
* Use get_object_or_404() in views! This handles exceptions with grace and usually does what is expected
* Use built-in class based views (CBV) wherever possible - [Class based views reference](https://docs.djangoproject.com/en/1.11/ref/class-based-views/). Or, err on the side of function-based views (FBV), if this things get too complex.    
    ```python
    from django.http import HttpResponse
    from django.views.generic import View

    # The simplest FBV
    def simplest_view(request):
        # Business logic goes here
        return HttpResponse('FBV')

    # The simplest CBV
    class SimplestView(View):
        def get(self, request, *args, **kwargs):
            # Business logic goes here
            return HttpResponse('CBV')
    ```
    (See how Django FBVs are HTTP method neutral, but Django CBVs require specific
    HTTP method declaration.)

* Keep business logic out of views - views are for presentation logic. Refer to fat models. Makes it easier to expand later on.  
* **Remember:** views are functions, they take a request and return a response. What happens in between is up to us;

## Forms
* Forms require two things - where and how. `action=''` and `method=(get|post)`
* `django.forms.ModelForm` for creating forms from models
    ```python
    class MediaPlanForm(forms.ModelForm):
        class Meta:
            model = MediaPlan
            exclude = []
    ```
* For basic CRUD, there are generic views `CreateView, UpdateView, Deleteview` - define template, model, fields and success_url and basically good to go. See `mediaplans.views` and `mediaplan.urls` for clarifications.

## Templates
#### Variables
In Django templates, the context is passed in as a dict-like object. The values can be all sorts of python objects - dicts, objects, lists, etc. Lookups for dicts, objects and lists is done using dot notation, so this works in templates:  
```python

class TestObject(object):
    def __init__(self, foo):
        self.foo = foo

context = {
    'title_text': 'My site is cool',
    'headcount': {'male': 10, 'female': 15},
    'sample_object': TestObject('kala'),
    'sample_list': ['zero', 'one', 'two', 'three'],
    'html': '<h1>Hello</h1>'
}
```
```html
<div>{{ title_text }}</div> 
Outputs: My site is cool
<div>{{ headcount.female }}</div> 
Outputs: 15
<div>{{ sample_object.foo }}</div> 
Outputs: kala
<div>{{ sample_list.1 }}</div> 
Outputs: one
```

#### Tags
Tags provide arbitrary logic in templates - e.g. for loops, if statements etc.
e.g. generating a html list:
```html
<ul>
{% for item in sample_list %}
    <li>{{ item }}</li>
{% endfor %}
</ul>
```

#### Filters
Filters transform the values of variables and tag arguments. E.g. converting to titlecase or allowing HTML to be rendered instead of escaped
```html
<div>{{ title_text|title }}</div> 
Outputs: My Site Is Cool 
<div>{{ html }}</div> 
Outputs: <h1>Hello</h1> 
<div>{{ html|safe }}</div> 
Outputs: Hello (as actual h1 element) 

```

[Template language overview](https://docs.djangoproject.com/en/1.11/topics/templates/#the-django-template-language)

## Helper functions
* User helper functions to perform checks or validations on the requests - less attributes and validations can be reused
```python
def check_rights(request):
    if request.user.is_allowed or request.user.is_staff:
        request.is_valid = True
        return request
    
    # Return a HTTP 403 back to the user
    raise PermissionDenied
```
We can also use this to e.g. add custom errors to the request object etc.

## General Best Practices
* An app should do one thing and do it well
* For the sake of clarity, the app's name should be a plural version of the app's main model. e.g. Events (app) --> Event (model)
and if possible, follow the URL structure (and vice versa): http://www.my-project.com/events/
* Keep the number of Models per app as small as possible. If there are a lot of models then you are probably breaking rule #1
* For repeating fields, use Model Inheritance and Abstract Base Models (A class that inherits from models.Model and other models inherit from this new class) - tables will not be created for the ABM's, but for their children
