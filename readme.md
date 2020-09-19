How to create a Django backend server

1. Check if you have django

```python -m django --version```

2. Create a project 
``` django-admin startproject yourprojectnamehere ```

3. cd into project, you should be able to ls and see manage.py since that will run your commands

4. Check that server is running

```python manage.py runserver```

5. You can change the port and then go to localhost:desiredport

``` python manage.py runserver desiredport#```

6. create an app within the django app, i'm creating a polls app

``` python manage.py startapp polls ```

7. Create a view, in polls/view.py, i entered the following code

```
from django.http import HttpResponse


def index(request):
    return HttpResponse("Hello, world. You're at the polls index.")

```

8. In the polls/urls.py include the following 

```
from django.urls import path

from . import views

urlpatterns = [
    path('', views.index, name='index'),
]

```
9. The next step is to point the root URLconf at the polls.urls module. In mysite/urls.py, add an import for django.urls.include and insert an include() in the urlpatterns list, so you have:

```
from django.contrib import admin
from django.urls import include, path

urlpatterns = [
    path('polls/', include('polls.urls')),
    path('admin/', admin.site.urls),
]

```
You should always use include() when you include other URL patterns. admin.site.urls is the only exception to this.

10.  verify it's working 

```python manage.py runserver```

go to http://localhost:8000/polls and your should see your hello world, you're at the polls index

11.

