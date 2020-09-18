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

