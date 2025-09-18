# Django

These are summary notes following the tutorial at: https://docs.djangoproject.com/en/5.2/intro/tutorial01/

Repo with commits per section at: https://github.com/nandilugio/djangotutorial

## Projects and apps

A project contains (web) apps and apps can be installed in any project.

### Projects

Include the main `settings.py` and `urls.py`, as well as the WSGI and ASGI entrypoints.

### Apps

Are added to the project in settings `INSTALLED_APPS` so they're included in migrations (and what else?).

Sometimes you also "mount" app's urls into a prefix in your project urls manually using `django.urls.import`.

By default they define some MVC-like entities: models (M) and views (V+C), TODO: though you can add a template engine to separate the V?

## CLI entrypoint

`django-admin`, `python -m django` and `python manage.py` are all the same, except that the last automatically grabs the main settings of your project so can perform more operations.

Common operations are:
- `startproject` and `startapp` to create them
- `check` for problems in the project
- `runserver` to start an embedded dev server that reloads (existing) source files on change
- `migrate`, `makemigrations`, `showmigrations`, `sqlmigrate` to deal witn ORM migrations (see below)
- `createsuperuser` for Django Admin

## ORM

### Models

They _define_ the database. On change, use django command `makemigrations` to generate the migrations that reflect them to the DB.

They also define the DAO interfaces (see `was_published_recently()` below) and are parsed by Django Admin to generate its CRUD interfaces when registered to it by:

```python
# polls/admin.py
from django.contrib import admin
from .models import Question

admin.site.register(Question)
```

```python
class Question(models.Model):
    question_text = models.CharField(max_length=200)
    pub_date = models.DateTimeField("date published")   # An optional first string as doc. TODO Also used by DJango Admin right?

    def was_published_recently(self):
        return self.pub_date >= timezone.now() - datetime.timedelta(days=1)

    def __str__(self):
        return self.question_text


class Choice(models.Model):
    question = models.ForeignKey(Question, on_delete=models.CASCADE)
    choice_text = models.CharField(max_length=200)
    votes = models.IntegerField(default=0)

    def __str__(self):  # Also used by Django Admin
        return self.choice_text
```

#### Querying

Follows a kind of _repository_ pattern. See `*Manager`:

```python
Question.objects  # Returns a QuestionManager
```

##### Creating

```python
t = Question.objects.create(question_text="What's new?", pub_date=timezone.now())

# Also
Question(question_text="What's new?", pub_date=timezone.now())
t.save()
```

##### Retrieving


```python
# Queries return either a single `Question` instance
Question.objects.get(id=3)  # or pk=3
Question.objects.get(question_text="What's new?")
Question.objects.get(question_text__startswith="What")
Question.objects.first()
Question.objects.last()

# Or a `QuerySet` containing many of them
Question.objects.all()
Question.objects.filter(question_text__startswith="What")
Question.objects.filter(pub_date__lte=timezone.now())
Question.objects.order_by('-pub_date')

# `QuerySet` methods are chainable:
Question.objects.filter(question_text__startswith="What").order_by('-pub_date')
```

##### Updating

```python
thing = Question.objects.get(...)
thing.name = 'qwe'
thing.save()
```

##### Deleting

```python
q = Question.objects.get(pk=1)
q.delete()

qs = Question.objects.all()
qs.delete()
```

##### Foreign keys

```python
q = Question.objects.get(pk=1)

# Create related
q.choice_set.create(choice_text="Not much", votes=0)
q.choice_set.create(choice_text="The sky", votes=0)
c = q.choice_set.create(choice_text="Just hacking again", votes=0)
c.question  # <Question: What's up?>

# Access related
q.choice_set.all()    # <QuerySet [<Choice: Not much>, <Choice: The sky>, <Choice: Just hacking again>]>
q.choice_set.count()  # 3

# Filter by related attrs
Choice.objects.filter(question__pub_date__year=current_year)  # <QuerySet [<Choice: Not much>, <Choice: The sky>, <Choice: Just hacking again>]>
```

## NVim, uv and Django

```sh
uv pip install django-stubs # makes pyright understand Django
uv run nvim                 # makes pyright run in the virtualenv so it can find all packages and modules
```