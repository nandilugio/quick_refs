# Django

## ORM

### Models

TODO

#### Querying

Follows a kind of _repository_ pattern. See `*Manager`:

```python
Thing.objects  # Returns a ThingManager
```

##### Creating

```python
Thing.objects.create(name='Scarpandol', created_at=timezone.now())
```

##### Retrieving

Queries return either a single `Thing` instance, or a `QuerySet` containing many of them:

```python
Thing.objects.first()
Thing.objects.last()
Thing.objects.get(name='asd')

Thing.objects.all()
Thing.objects.filter(name__contains='asd')
Thing.objects.filter(created_at__lte=timezone.now())

Thing.objects.order_by('-created_at')
```

`QuerySet` methods are chainable:

```python
Thing.objects.filter(name__contains='asd').order_by('-created_at')
```

##### Updating

```python
thing = Thing.objects.get(...)
thing.name = 'qwe'
thing.save()
```

##### Deleting

TODO