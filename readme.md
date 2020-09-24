How to create a Django backend server
original documentation at https://docs.djangoproject.com/en/3.1/intro/tutorial01/

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

7. Create a view, in polls/views.py, i entered the following code

```
from django.http import HttpResponse


def index(request):
    return HttpResponse("Hello, world. You're at the polls index.")

```

8. Create the file urls.py in the polls directory. In the polls/urls.py include the following 

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

go to http://localhost:8000/polls and you should see your hello world, you're at the polls index

11. Now, open up mysite/settings.py. A normal Python module uses SQLite by default, but you can switch to other databases

ENGINE – Either 'django.db.backends.sqlite3', 'django.db.backends.postgresql', 'django.db.backends.mysql', or 'django.db.backends.oracle'. Other backends are also available.

12. Set TIME_ZONE to desired time zone https://i.stack.imgur.com/JJmzp.png

13. Some of these applications make use of at least one database table, though, so we need to create the tables in the database before we can use them. To do that, run the following command:

``` python manage.py migrate ```

you should see the db.sqlite3 file created in mysite folder

14. Creating models - in this app, we need a Question and Choice(s)
Go to polls/models.py and paste the following

```
from django.db import models


class Question(models.Model):
    question_text = models.CharField(max_length=200)
    pub_date = models.DateTimeField('date published')


class Choice(models.Model):
    question = models.ForeignKey(Question, on_delete=models.CASCADE)
    choice_text = models.CharField(max_length=200)
    votes = models.IntegerField(default=0)
```    

15. Activating models - To include the app in our project, we need to add a reference to its configuration class in the INSTALLED_APPS setting

in mysite/settings - you will add polls.apps.PollsConfig to INSTALLED_APPS, so that it looks like the following 

```
INSTALLED_APPS = [
    'polls.apps.PollsConfig',
    'django.contrib.admin',
    'django.contrib.auth',
    'django.contrib.contenttypes',
    'django.contrib.sessions',
    'django.contrib.messages',
    'django.contrib.staticfiles',
]

```

16. Now let's run the migrations

```python manage.py makemigrations polls```

17. There’s a command that will run the migrations for you and manage your database schema automatically - that’s called migrate, and we’ll come to it in a moment - but first, let’s see what SQL that migration would run. The sqlmigrate command takes migration names and returns their SQL:

``` python manage.py sqlmigrate polls 0001 ```

18.Now, run migrate again to create those model tables in your database:

```python manage.py migrate```

19. Now, let’s hop into the interactive Python shell and play around with the free API Django gives you. To invoke the Python shell, use this command: 

```python manage.py shell```

20 lets try a question

```>>> from polls.models import Choice, Question  # Import the model classes we just wrote.

# No questions are in the system yet.
>>> Question.objects.all()
<QuerySet []>

# Create a new Question.
# Support for time zones is enabled in the default settings file, so
# Django expects a datetime with tzinfo for pub_date. Use timezone.now()
# instead of datetime.datetime.now() and it will do the right thing.
>>> from django.utils import timezone
>>> q = Question(question_text="What's new?", pub_date=timezone.now())

# Save the object into the database. You have to call save() explicitly.
>>> q.save()

# Now it has an ID.
>>> q.id
1

# Access model field values via Python attributes.
>>> q.question_text
"What's new?"
>>> q.pub_date
datetime.datetime(2012, 2, 26, 13, 0, 0, 775217, tzinfo=<UTC>)

# Change values by changing the attributes, then calling save().
>>> q.question_text = "What's up?"
>>> q.save()

# objects.all() displays all the questions in the database.
>>> Question.objects.all()
<QuerySet [<Question: Question object (1)>]>
```


<QuerySet [<Question: Question object (1)>]> is not readable to us, let's change that

21. In polls/models.py, let's add a __str__ method in question and choice

```
from django.db import models

class Question(models.Model):
    # ...
    def __str__(self):
        return self.question_text

class Choice(models.Model):
    # ...
    def __str__(self):
        return self.choice_text

```

22. lets add a custom method to models
in polls/models.py, add the code to existing code

```
import datetime

from django.db import models
from django.utils import timezone


class Question(models.Model):
    # ...
    def was_published_recently(self):
        return self.pub_date >= timezone.now() - datetime.timedelta(days=1)

```


23. close shell and run it again

>>>from polls.models import Choice, Question
>>>Question.objects.all() # now it should run as desired
<QuerySet [<Question: What's new?>]>

you can read about all the choices you have in shell to look up questions and choices in the documentation

24. Creating an admin user, let's create it

```python manage.py createsuperuser```

create username, email address, and password when requested

25. Explore the admin site, start the server

```  python manage.py runserver```

26. Navigate to http://127.0.0.1:8000/admin/ and login

27. There's no polls app anywhere though, we must add it to admin. Go to polls/admin.py and make it look like this 

``` 
from django.contrib import admin

from .models import Question

admin.site.register(Question)
```

28. you will now see questions added to admin(you may need to refresh the browser)

29. Add a question. Date and time and generated via javascript

30. Let's add more views
in polls/views.py, add the following

```
def detail(request, question_id):
    return HttpResponse("You're looking at question %s." % question_id)

def results(request, question_id):
    response = "You're looking at the results of question %s."
    return HttpResponse(response % question_id)

def vote(request, question_id):
    return HttpResponse("You're voting on question %s." % question_id)
```
31. Wire these new views to urls.py
In polls/urls.py, add the following
```
from django.urls import path

from . import views

urlpatterns = [
    # ex: /polls/
    path('', views.index, name='index'),
    # ex: /polls/5/
    path('<int:question_id>/', views.detail, name='detail'),
    # ex: /polls/5/results/
    path('<int:question_id>/results/', views.results, name='results'),
    # ex: /polls/5/vote/
    path('<int:question_id>/vote/', views.vote, name='vote'),
]
```


32. Take a look in your browser, at “/polls/34/”. It’ll run the detail() method and display whatever ID you provide in the URL. Try “/polls/34/results/” and “/polls/34/vote/” too – these will display the placeholder results and voting pages.

33. Lets write views that actually do something
in polls/views.py, enter
```
from django.http import HttpResponse

from .models import Question


def index(request):
    latest_question_list = Question.objects.order_by('-pub_date')[:5]
    output = ', '.join([q.question_text for q in latest_question_list])
    return HttpResponse(output)

# Leave the rest of the views (detail, results, vote) unchanged
```

34. http://127.0.0.1:8000/polls/ is still hardcoded with the question. Let's fix this by creating templates

35. create the following directories and index.html in it
polls/templates/polls/index.html
Paste the following in index.html

```
{% if latest_question_list %}
    <ul>
    {% for question in latest_question_list %}
        <li><a href="/polls/{{ question.id }}/">{{ question.question_text }}</a></li>
    {% endfor %}
    </ul>
{% else %}
    <p>No polls are available.</p>
{% endif %}
```

these are incomplete html, you should use complete HTML in your own projects https://developer.mozilla.org/en-US/docs/Learn/HTML/Introduction_to_HTML/Getting_started#Anatomy_of_an_HTML_document

36. lets update index in polls/views
in polls/views.py, update index

```
from django.shortcuts import render

from .models import Question


def index(request):
    latest_question_list = Question.objects.order_by('-pub_date')[:5]
    context = {'latest_question_list': latest_question_list}
    return render(request, 'polls/index.html', context)
```
37. Lets fix the 404 error when a question doesn't exist
in polls/views

```
from django.http import Http404
from django.shortcuts import render

from .models import Question
# ...
def detail(request, question_id):
    try:
        question = Question.objects.get(pk=question_id)
    except Question.DoesNotExist:
        raise Http404("Question does not exist")
    return render(request, 'polls/detail.html', {'question': question})
```
The view raises the Http404 exception if a question with the requested ID doesn’t exist.

38. Create a quick detail view 
polls/templates/polls/detail.html

simply enter this for now
```{{ question }}```

39. A shortcut: get_object_or_404()

t’s a very common idiom to use get() and raise Http404 if the object doesn’t exist. Django provides a shortcut. Here’s the detail() view, rewritten:
in polls/views

```
from django.shortcuts import get_object_or_404, render

from .models import Question
# ...
def detail(request, question_id):
    question = get_object_or_404(Question, pk=question_id)
    return render(request, 'polls/detail.html', {'question': question})
```

40. Here's a little better code from detail.html 

```
<h1>{{ question.question_text }}</h1>
<ul>
{% for choice in question.choice_set.all %}
    <li>{{ choice.choice_text }}</li>
{% endfor %}
</ul>
```

41. Let's make index.html a little less hardcoded, change it to (just the <li> part)

```
<li><a href="{% url 'detail' question.id %}">{{ question.question_text }}</a></li>

```

42. Namespacing URL Names
in polls/urls.py, change it to 

```
from django.urls import path

from . import views

app_name = 'polls'
urlpatterns = [
    path('', views.index, name='index'),
    path('<int:question_id>/', views.detail, name='detail'),
    path('<int:question_id>/results/', views.results, name='results'),
    path('<int:question_id>/vote/', views.vote, name='vote'),
]
```

43. Change index.html to point to the namespace

```
<li><a href="{% url 'polls:detail' question.id %}">{{ question.question_text }}</a></li>
```

44. Write a minimal form
in detail.html

```
<h1>{{ question.question_text }}</h1>

{% if error_message %}<p><strong>{{ error_message }}</strong></p>{% endif %}

<form action="{% url 'polls:vote' question.id %}" method="post">
{% csrf_token %}
{% for choice in question.choice_set.all %}
    <input type="radio" name="choice" id="choice{{ forloop.counter }}" value="{{ choice.id }}">
    <label for="choice{{ forloop.counter }}">{{ choice.choice_text }}</label><br>
{% endfor %}
<input type="submit" value="Vote">
</form>

```

45. Let's handle the submitted data
in polls/urls.py , add

```
path('<int:question_id>/vote/', views.vote, name='vote'),
```

46. Let's create a better view for voting, in polls/views.py, add

```
from django.http import HttpResponse, HttpResponseRedirect
from django.shortcuts import get_object_or_404, render
from django.urls import reverse

from .models import Choice, Question
# ...
def vote(request, question_id):
    question = get_object_or_404(Question, pk=question_id)
    try:
        selected_choice = question.choice_set.get(pk=request.POST['choice'])
    except (KeyError, Choice.DoesNotExist):
        # Redisplay the question voting form.
        return render(request, 'polls/detail.html', {
            'question': question,
            'error_message': "You didn't select a choice.",
        })
    else:
        selected_choice.votes += 1
        selected_choice.save()
        # Always return an HttpResponseRedirect after successfully dealing
        # with POST data. This prevents data from being posted twice if a
        # user hits the Back button.
        return HttpResponseRedirect(reverse('polls:results', args=(question.id,)))
```

47. Let's fix the results page so it's not hard-coded
in polls/views

```
from django.shortcuts import get_object_or_404, render


def results(request, question_id):
    question = get_object_or_404(Question, pk=question_id)
    return render(request, 'polls/results.html', {'question': question})

```

48. Now create the results.html file in templates/polls folder where you other html files are located

```
<h1>{{ question.question_text }}</h1>

<ul>
{% for choice in question.choice_set.all %}
    <li>{{ choice.choice_text }} -- {{ choice.votes }} vote{{ choice.votes|pluralize }}</li>
{% endfor %}
</ul>

<a href="{% url 'polls:detail' question.id %}">Vote again?</a>

```

49. Go to /polls/1/ and vote and you should see the results page

50. Amend URLconf
Go to polls/urls.py and amend it to 

```
from django.urls import path

from . import views

app_name = 'polls'
urlpatterns = [
    path('', views.IndexView.as_view(), name='index'),
    path('<int:pk>/', views.DetailView.as_view(), name='detail'),
    path('<int:pk>/results/', views.ResultsView.as_view(), name='results'),
    path('<int:question_id>/vote/', views.vote, name='vote'),
]

```

51. lets get rid of of our old index, detail and results views and use Django's generic views

```
from django.http import HttpResponseRedirect
from django.shortcuts import get_object_or_404, render
from django.urls import reverse
from django.views import generic

from .models import Choice, Question


class IndexView(generic.ListView):
    template_name = 'polls/index.html'
    context_object_name = 'latest_question_list'

    def get_queryset(self):
        """Return the last five published questions."""
        return Question.objects.order_by('-pub_date')[:5]


class DetailView(generic.DetailView):
    model = Question
    template_name = 'polls/detail.html'


class ResultsView(generic.DetailView):
    model = Question
    template_name = 'polls/results.html'


def vote(request, question_id):
    ... # same as above, no changes needed.
```

52. Testing , lets create a failing test 
in polls/tests.py , add

```
import datetime

from django.test import TestCase
from django.utils import timezone

from .models import Question


class QuestionModelTests(TestCase):

    def test_was_published_recently_with_future_question(self):
        """
        was_published_recently() returns False for questions whose pub_date
        is in the future.
        """
        time = timezone.now() + datetime.timedelta(days=30)
        future_question = Question(pub_date=time)
        self.assertIs(future_question.was_published_recently(), False)

```

in your terminal 

```
python manage.py test polls
```

Your test will fail because you are creating a question in the future and asking if it was published recently, it's returning true,  which should be false

53. Fixing the bug, in polls/models.py , change the code to 

```
def was_published_recently(self):
    now = timezone.now()
    return now - datetime.timedelta(days=1) <= self.pub_date <= now

```
if you run the test again, it will pass

54. Add a couple of more tests
```
def test_was_published_recently_with_old_question(self):
    """
    was_published_recently() returns False for questions whose pub_date
    is older than 1 day.
    """
    time = timezone.now() - datetime.timedelta(days=1, seconds=1)
    old_question = Question(pub_date=time)
    self.assertIs(old_question.was_published_recently(), False)

def test_was_published_recently_with_recent_question(self):
    """
    was_published_recently() returns True for questions whose pub_date
    is within the last day.
    """
    time = timezone.now() - datetime.timedelta(hours=23, minutes=59, seconds=59)
    recent_question = Question(pub_date=time)
    self.assertIs(recent_question.was_published_recently(), True)
```

55. Let's fix the bug so that future polls don't show
in polls/views.py

```
from django.utils import timezone

def get_queryset(self):
    """
    Return the last five published questions (not including those set to be
    published in the future).
    """
    return Question.objects.filter(
        pub_date__lte=timezone.now()
    ).order_by('-pub_date')[:5]
```

More tests to consider
https://docs.djangoproject.com/en/3.1/intro/tutorial05/

56. Customizing the look with css
create a folder in your polls folder called static
in static create a folder called polls
and then in here style.css

polls/static/polls/style.css

57. In style.css, add some css

```
li a {
    color: green;
}
```

58. Connect style.css to index.html

```
{% load static %}

<link rel="stylesheet" type="text/css" href="{% static 'polls/style.css' %}">

```

Restart your server and now you links are green

59. Add a background , save a background.gif at
polls/static/polls/images/background.gif.

You can create other css files or reuse the same one for other html, just point it at the right file in the html doc

60. Customizing the admin section to create questions and answers there

in polls/admin

```
from django.contrib import admin

from .models import Question


class QuestionAdmin(admin.ModelAdmin):
    fields = ['pub_date', 'question_text']

admin.site.register(Question, QuestionAdmin)
```
You’ll follow this pattern – create a model admin class, then pass it as the second argument to admin.site.register() – any time you need to change the admin options for a model.

61. You may want to split up the form into fieldsets

```
from django.contrib import admin

from .models import Question


class QuestionAdmin(admin.ModelAdmin):
    fieldsets = [
        (None,               {'fields': ['question_text']}),
        ('Date information', {'fields': ['pub_date']}),
    ]

admin.site.register(Question, QuestionAdmin)
```

62. We need to add choices since polls need those
in polls/admin.py

```
from django.contrib import admin

from .models import Choice, Question
# ...
admin.site.register(Choice)
```

63. Choices and Questions are not connected though, fix in admin.py

```
from django.contrib import admin

from .models import Choice, Question


class ChoiceInline(admin.StackedInline):
    model = Choice
    extra = 3


class QuestionAdmin(admin.ModelAdmin):
    fieldsets = [
        (None,               {'fields': ['question_text']}),
        ('Date information', {'fields': ['pub_date'], 'classes': ['collapse']}),
    ]
    inlines = [ChoiceInline]

admin.site.register(Question, QuestionAdmin)
```

64. Change the view to Tabularinline to make it take up less space

```
class ChoiceInline(admin.TabularInline):
    #...
```
65. add list_display after inlines(no comma needed)
```
list_display = ('question_text', 'pub_date')
```

66. Lets also add was_published recently 
```
class QuestionAdmin(admin.ModelAdmin):
    # ...
    list_display = ('question_text', 'pub_date', 'was_published_recently')
```

67. improve the look in models.py
```
    def was_published_recently(self):
        now = timezone.now()
        return now - datetime.timedelta(days=1) <= self.pub_date <= now
    was_published_recently.admin_order_field = 'pub_date'
    was_published_recently.boolean = True
    was_published_recently.short_description = 'Published recently?'
```

68. add a filter in admin.py under inlines = [ChoiceInLine]
```
list_filter = ['pub_date']

```
69. add a search bar for your polls
```
search_fields = ['question_text']
```

That's basically it, this is just a basic app to get you started