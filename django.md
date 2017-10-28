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
* [Traversing relationships](https://docs.djangoproject.com/en/1.11/topics/db/examples/many_to_one/)
* Database API examples:
    ```python
    # Get all rows from Media model
    Media.objects.all()

    # Get an object by an attribute
    Media.objects.get(pk=1)  # Primary key
    Media.objects.get(name='Google')

    # Getting column values of the record with dot notation
    g.id
    >> 1
    # Traverse foreign keys
    mp = Mediaplan.objects.all()[0]
    mp.media.name
    >> 'Facebook'

    # Filter across joins (notice the dunder!)
    Mediaplan.objects.filter(media__pk=1)

    ```

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
* **Remember:** views are functions, they take a request and return a response. What happens in between is up to us  
* If defining a `success_url` for generic CBV's, use `reverse_lazy()` - since URLConf is loaded later, then using `reverse()` will result in a traceback

##### Class Based Views
* All CBV's at base level should inherit from `django.views.generic.View`
* [Detailed descriptions of GCBVs](http://ccbv.co.uk/)
* Nifty trickwhen overriding methods - write custom logig, then give the resolution back to the generic view with super;
    ```python
    from django.contrib.auth.mixins import LoginRequiredMixin 
    from django.views.generic import CreateView
    from .models import Flavor
    class FlavorCreateView(LoginRequiredMixin, CreateView): 
        model = Flavor
       fields = ['title', 'slug', 'scoops_remaining']

    def form_valid(self, form):
        # Do custom logic here
        return super(FlavorCreateView, self).form_valid(form)
    ```
* For views that require logging in, we can use the LoginRequiredMixin instead of messing with the decorators:
    ```python
    from django.contrib.auth.mixins import LoginRequiredMixin

    class FlavorCreateView(LoginRequiredMixin, CreateView):
        # Do stuff
    ```
* When inheriting from `django.views.generic.View`, we have access to different request methods e.g:  
`def get(); def post()`  
and have the logic there. We could replicate the same functionality in FBV's with  
 `if request.method == "GET" | "POST"` etc...  
but CBV's are in that way cleaner.

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

* **Custom form field validations** - Django gives us basic validations that come from the model (e.g. field length), but there is often need for custom validations. Custom, single-field validators are just callables that raise an error if validation fails:
    ```python
    # core/validators.py
    from django.core.exceptions import ValidationError
    def validate_tasty(value):
    """Raise a ValidationError if the value doesn't start with the
           word 'Tasty'.
       """
    if not value.startswith('Tasty'):
        msg = 'Must start with Tasty' 
        raise ValidationError(msg)
    
    # models.py
    # Attach it to a fields with the validators kwarg:
    from .validators import validate_tasty

    Class foo(models.Model):
        bar = models.Charfield(max_lenth=20, validators=[validate_tasty])

    ```
    This can also be placed in a Abstract base model (e.g. a title field) and make all other models that need that validation inherit from it.

* Custom validators can be attached at form, not model level (e.g. having only some forms require certain validations). Also, they can be added to any field:
    ```python
    class SampleForm(forms.ModelForm):
        def __init__(self, *args, **kwargs):
            super(SampleForm, self).__init__(*args, **kwargs)
            self.fields['title'].validators.append(validate_tasty)
            self.fields['age'].validators.append(validate_tasty)

        class Meta:
            model = Foo
    ```

* If we want to use that form with a GCBV (that usually creates the form from the model attribute), we explicitly override it with our form with the `form_class` attribute
    ```python
    class Foo(models.UpdateView):
        model = bar
        form_class = SampleForm
    ```  
    The Foo view now uses the SampleForm for validations

* `def clean_FIELDNAME()` can be defined for field-specific validations in forms after initial validations when the data has already been cleaned. Clean data can be accessed from the `self.cleaned_data[FIELD_NAME]` attribute.
    ```python
    def clean_title(self):
        title = clean_data['title']
        
        # Get the object with this title and do the validation
        if Foo.objects.get(title=title).age < 5:
            raise ValidationError('blah')

        return title
    ```
* Use the `clean()` method for making interdependant, multi-field validations
    ```python
    class Foo(forms.ModelForm):
        def clean(self):
            cleaned_data = super(Foo, self).clean()
            x = cleaned_data.get('x')
            y = cleaned_data.get('y')

            if x != y:
                raise ValidationError('Blah')
    ```
* Chapter 11.4 - Two Forms, Two views, one model

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
* Template inheritance  - get comfortable with `{% extends %} | {% block NAME %} || {% block.super %}`. Just `{% block NAME %}` will replace a block while calling `{% block.super %}` will extend the block, including the original content.
* A good practice is to start and end blocks with their names, for clarity: `{% block title %} ...stuff.... {% endblock title %}`. This way it is clear what block is being closed.
* In templates, model name can be used instead on object e.g. for Foo model, both will work `{% for f in object_list | for f in foo_list %}`. Although, while object will work anywhere, it is better for code reuse.
* **USE NAMESPACED URLS INSTEAD OF HARDCODED ONES**



[Template language overview](https://docs.djangoproject.com/en/1.11/topics/templates/#the-django-template-language)  

##### Static files
```html
<!-- include staticfiles in template -->
{% static load %}

<script src="{% static 'js/script.js' %}"></script>
<img src="{% static 'img/img.jpg' %}"></img>
```

##### AJAX Requests
1. Create a view function and route that to an url or modify that dispatch() function in a CBV to handle ajax
2. Write the ajax function - parse the values from DOM that you want to pass to the server and pass them as JSON
3. In the view function, do stuff and return a serialized JSON
4. In handle the response client side

```javascript
// Use js-cookie lib to get the csrf token
var csrftoken = Cookies.get('csrftoken');

// Ajax Setup
function csrfSafeMethod(method) {
    // these HTTP methods do not require CSRF protection
    return (/^(GET|HEAD|OPTIONS|TRACE)$/.test(method));
}

//set the token in header
$.ajaxSetup({
    beforeSend: function(xhr, settings) {
        if (!csrfSafeMethod(settings.type) && !this.crossDomain) {
            xhr.setRequestHeader("X-CSRFToken", csrftoken);
        }
    }
});

// Actual ajax request
$('.deleteIcon').on('click', function() {
    var selectedPlan = $(this).attr('data-id');

      $.ajax({
        url: '/delete/' + selectedPlan,
        data: {},
        dataType: 'json',
        success: function (data) {
            console.log('success');
            $('#'+selectedPlan).remove();
        }
      });
});
```

In the view function, overwrite the `dispatch()` method to handle ajax requests and respond with JSON
```python
from django.http import JsonResponse

    def dispatch(self, *args, **kwargs):
        resp = super(MediaPlanDeleteView, self).dispatch(*args, **kwargs)
        if self.request.is_ajax():
            self.delete(*args, **kwargs) # Actually call the delete method to delete stuff
            response_data = serializ{"result": "ok"}
            return JsonResponse(serializers.serialize('json', response_data))
        else:
            return resp

```

* When using a field with choices in template, the human-readable name can be outputted with `{{ object.get_FIELDNAME_display }}`

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

## Mixins
Mixins can be used to define certain functionality and then append that to the views, that inherit from the mixin. Important is to place the base view to the far left and the mixins to the right. Mixins help to keep code DRY. Basically override any function and the classes that inherit from the Mixin also inherit that function.
```python
class BannerTypeMixin:

        """Rewrite dispatch to handle ajax request and respond with a JSON
        that provides a list of banner sizes that are applicable for that media        
        """
        
    def dispatch(self, *args, **kwargs):
        response = super(BannerTypeMixin, self).dispatch(*args, **kwargs)
        if self.request.is_ajax():
            media_id = self.request.GET.get('media_id')
            if media_id:
                qs = BannerType.objects.filter(media__pk=media_id)
            else:
                qs = BannerType.objects.all()
            qs_json = serializers.serialize('json', qs)
            response_data = {"result": qs_json}
            return JsonResponse(response_data)
        else:
            return response   

class MediaPlanCreateView(BannerTypeMixin , CreateView):
    pass

class MediaPlanUpdateView(BannerTypeMixin, UpdateView):
    pass
```

## The User model
[Quick tutorial](https://simpleisbetterthancomplex.com/tutorial/2016/06/27/how-to-use-djangos-built-in-login-system.html)
* Make sure the `LOGIN_URL` and `LOGIN | LOGOUT_REDIRECT_URL` in settings is defined
* In views do `from django.contrib.auth import views as auth_views`, subclass the required views e.g. `LoginView, LogoutView`
and update the required parameters, e.g template_name. [All authentication views](https://docs.djangoproject.com/en/1.11/topics/auth/default/#all-authentication-views)
* After that, make/update a template, wire views to URLs and basically login/logout is good to go


## General Best Practices
* An app should do one thing and do it well
* For the sake of clarity, the app's name should be a plural version of the app's main model. e.g. Events (app) --> Event (model)
and if possible, follow the URL structure (and vice versa): http://www.my-project.com/events/
* Keep the number of Models per app as small as possible. If there are a lot of models then you are probably breaking rule #1
* For repeating fields, use Model Inheritance and Abstract Base Models (A class that inherits from models.Model and other models inherit from this new class) - tables will not be created for the ABM's, but for their children
* Debugging - install instructions for [Django debug toolbar](https://django-debug-toolbar.readthedocs.io/en/stable/installation.html)

# TODO:
* ManyToMany fields - Check Youtube
* User model - permissions on app side!
* Testing django
* auth/contib module. messages
* Vars in urlconfs etc. where do they come from?
