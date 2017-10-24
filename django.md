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
