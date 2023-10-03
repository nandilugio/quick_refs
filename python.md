Python 3
========

Language properties
-------------------

- Object Oriented, though funtions are First Class Citizens (see `functools`)
- Dynamic: `a = 1; a = '2'` though type annotations can be used (see `mypy`)
- Strongly typed: `1 + '2'  # TypeError`
- Optionally precompiled (TODO)

Syntax
------

### Some conventions

- `CamelCase` for classes
- `snake_case` for functions and variables
- `_single_leading_underscore` for private things like methods, module functions and variables, etc.
    - `__double_leading_underscore` is name mangling: `__name` becomes `_classname__name` so they're private even in inheritance situations
    - `__double_leading_and_trailing_underscore__` for special reserved methods 

### Basic types, built-in functions and operators

```python
#### Assignment ####

a = 1; b = 2
a, b = b, a  # Swap!

(n := len('asd')) == 3  # Walrus operator (:=) assigns and returns the value

head, *other, tail = (1,2,3,4); head == 1; other == [2, 3]; tail == 4  # Unpacking assignment

p = {'x': 5, 'y': 6}
{'a': 1, 'b': 2, **p} == {'a': 1, 'b': 2, 'x': 5, 'y': 6}  # Unpacking a dict


#### None type ####

type(None)  # NoneType


#### Booleans ####

type(True) == bool

(True and False) or not True == False

# Truthiness
all([1, 'no', ' ']) == True
bool(0) == bool('') == False # casting using the Bool constructor

# Object identity
a = 257; b = 257  # [-5,256] integer objects are reused! as are small strings
a == b and id(a) != id(b)


#### Numbers ####

type(1)      == int
type(1.1)    == float
type(3 + 2j) == complex
# Also see the extended type hierarchy in the `numbers` module

1000000 == 1_000_000  # Underscores are ignored in numeric literals
0b1010 == 0o12 == 0xa == 10  # Different bases

abs(-3) == 3
int(3.4) == 3 # casting using the Int constructor

1 + 2.5 == 3.5
2 ** 8 == 256
5 / 2 == 2.5; 5 // 2 == 2; 5 % 2 == 1

3 < 5 >= 2 == 2 != 4

```

### Sequence data types

Ordered, iterable sequences of objects not restricted on type.

```python
#### Strings ####
# Immutable character/byte sequences (see `bytearray` for mutable analogous)

type('asd') == type(u'asd') == str # Unicode in Py3
type(b'asd') == bytes

# Encoding. Default is 'utf-8' in Py3
u'ñ'                  == b'\xc3\xb1'.decode()
u'ñ'.encode()         == b'\xc3\xb1'
u'ñ'.encode('latin1') == b'\xf1'

len('asd') == 3
float('3.4') == 3.4 # casting using the Float constructor

'a,s'.split(',') == ['a', 's']
' a '.strip() == 'a'
'-'.join(['s', 'o', 's']) == 's-o-s'

'a' + 'b' == 'ab'

'A' < 'a' < 'b' < 'bb'

# Interpolation
species = 'roses'; color = 'red'
'roses are red' == \
    f'{species} are {color}' == \
    '{species} are {color}'.format(species=species, color=color) == \
    '%s are %s' % (species, color)


#### Lists ####
# Mutable lists

l = [1, 2, 3, 'a', ['b', 'c']]
type(l) == list

l.append('epa')
l.pop() == 'epa' 


#### Tuples ####
# Immutable, hashable lists

t = 'red', # trailing comma for 1-tuple literals
type(t) == tuple

t = 'red', 'green', 'blue'
t == ('red', 'green', 'blue') # parens are optional


#### Ranges ####
# Implicit, immutable sequences of numbers

r = range(3) # same as range(0, 3)
type(r) == range
list(r) == [0, 1, 2]

list(range(5, -5, -2)) == [5, 3, 1, -1, -3]


#### Common operations on sequence types ####

l = [0, 1, 2, 3, 4]

len(l) == 5
max(l) == 4

# Indexing
l[2] == 2  # 0 indexed
l[2:2] == []; l[2:4] == [2, 3]
l[-4:-2] == [1, 2]
l[:2] == [0, 1]; l[2:] == [2, 3, 4]
l[::2] == [0, 2, 4] # by steps
# Ranges are [incl:non-incl]. Tip: count the commas!

# Comprehensions (very Pythonesque!)
[n + 2 for n in l] == [2, 3, 4, 5, 6]
[n + 2 for n in l if n % 2 == 0] == [2, 4, 6]

# Other
3 in l; 9 not in l
del l[2:4]; l == [0, 1, 4]
```

### Set types

```python
#### Sets ####
# Immutable, unordered list (see `frozenset` for mutable analogous)

s = {1, 2, 3}
s == set((1, 2, 3))
type(s) == set

s.union({4, 5}) == {1, 2, 3, 4, 5}
s.isdisjoint({4, 5})
s.issubset({1,2,3,4}) == s < {1, 2, 3, 4}
```

### Mapping types

```python
#### Dictionaries ####
# Unordered mappings, keys can be any hashable type

d = {1: 2, 'a': 'x', 'b': [1, 2]}
type(d) == dict

d[1] == 2; d['a'] == 'x'

d1 = {1: 2, 3: 4}
d1 | {5: 6} == {1: 2, 3: 4, 5: 6}  # Merge operator (see also & and ^)
d1 |= {5: 6}  # In-place merge, same as `d1.update({5: 6})`

len(d) == 3
 
d.items()  # dict_items([(1, 2), ('a', 'x'), ('b', [1, 2])])
d.values() # dict_values([2, 'x', [1, 2]])
d.keys()   # dict_keys([1, 'a', 'b'])
list(d.keys()) == list(d)

'a' in d
del d['a']
('a' not in d) == (not 'a' in d) # Beware precedences!

# Dict constructor
{'one': 1, 'two': 2} == \
    dict(one=1, two=2) == \
    dict([('two', 2), ('one', 1)])
    # and more...

# Comprehensions (very Pythonesque!)
{f'double {x}': x * 2 for x in (1, 2, 3)} == \
    {'double 1': 2, 'double 2': 4, 'double 3': 6}
```

### Control Structures

```python
while predicate:
    pass

for i in range(10):
    pass

for i, v in enumerate(some_list):  # enumerate() returns a list of tuples, pairs of index and values
    pass
```
The `continue` statement ends the current iteration, and `break` finishes the whole loop.

You can use `else:` clauses in loops. They run when the loops ends without `break` being called.

```python
if predicate:
    pass
elif predicate2:
    pass
else:
    pass

match status:
    case 401 | 403 | 404:
        return "Not allowed"
    case _:
        return "Something's wrong with the internet"

# Arguments are patterns, resembling pattern maching in functional languages
# Matching is similar to the unpacking assignment `(x, y) = point`. `(x, y, *rest)` and dict
# matching like `{"x": x, "y", **rest}`  also work
match point:  # (x, y) tuple
    case (0, 0):
        print("Origin")
    case (0, y):
        print(f"Y={y}")
    case (x, 0):
        print(f"X={x}")
    case (x, y):
        print(f"X={x}, Y={y}")
    case _:
        raise ValueError("Not a point")
```

### Exceptions

BaseException > Exception > Base errors > Concrete errors

```python
try:
    pass
except SomeError:
    pass
except (SomeError, OtherError) as e:
    pass
except SomeError as e:
    pass
else:
    pass
finally:
    pass

raise ExceptionClass("Your argument")
```

### Functions

```python
def f(a, b):
    return a + b
type(f) == function

# Positional args are mandatory. They come before any args with default values.
# All args can be called positionally or by name (keyword args), except when:
#  - using the (greedy) `*` packing arg, that captures every positional arg from there on
#  - using the `/` separator, that marks the end of strictly positional args (can't be called by name)
def details(stricltypos1, strictlypos2, /, posarg, *remainingpos, kwarg1='default', kwarg2=None, **remainingkw):
    '''Some documentation...'''
    
    a = 'foo'  # local var assignment hide any higher scope definition, unless stating:
    global b, c  # usage in this scope will refer to the globally defined var
    nonlocal d, e  # usage in this scope will refer to the neares enclosing scope, excluding globals

    return 'is explicit'  # `None` is returned in other case
```

#### Lambdas

Implicit return, one expresion syntax to create functions.

```python
l = lambda x, y: x * y
type(l) == function  # Same as regular `def` functions
l(2, 3) == 6
```

#### Higher order functions:

```python
# Filter intends to receive a function (predicate)
def filter(predicate, iterable):
    for item in iterable:
        if predicate(item):
            yield item  # See Generators below

def is_even(n):
    return n % 2 == 0

[0, 2, 4, 6, 8] == \
    list(filter(is_even, range(10))) == \
    list(filter(lambda n: n % 2))
```

### Iterators and Generators

Iterators are objects suporting the `__next__()` method to iterate (and a couple extra details, see the docs if you want to build a custom one using a class). You can get an iterator from any iterable using `iter()`, and then call `next()` on it to get the next value.

Sequence types and other containers adhere to this protocol.

The easiest way to define a custom one is using `yield`:

```python
def evens_to(limit):
    for i in range(0, limit, 2):
        yield i

list(evens_to(10)) == [0, 2, 4, 6, 8]
```

The whole idea is not to have all the list in memory while iterating, but to _generate_ it on the fly.

### Classes

```python
class SomeClass:
    '''Some documentation...'''

    # Everything defined at this level is shared among instances,
    # including vars (class attrs) or functions (methods).

    class_var = 'shared between objects'

    def __init__(self, arg):
        # `self` is passed implicitly as 1st arg (when talking with an instance, see below)
        # Everything _defined_ on `self` at this level, becomes instance data
        self.instance_var = arg
        # But note that unless there's an instance definition overriding the name,
        # shared attributes are accessed using `self` too:
        assert self.class_var == 'shared between objects'

    def some_method(self, arg):
        return self.instance_var + arg

    @classmethod  # Ensures all calls pass the `cls` implicit argument
    def some_class_method(cls, arg):
        # Receives implicitly the _class object_, not an instance
        return cls.class_var + arg

    @staticmethod  # Ensures no call receives any `self` or `cls`
    def some_static_method(arg):
        # Doesn't have any access to the class or instances
        return arg * 2

inst = SomeClass('foo')
inst.class_var == 'shared between objects'
inst.some_method('bar') == 'foobar'

inst.some_method('asd') == SomeClass.some_method(inst, 'asd')  # Note the explicit `inst`
SomeClass.some_class_method('!') == inst.some_class_method('!') == 'shared between objects!'
SomeClass.some_static_method(3) == inst.some_static_method(3) == 6
```

#### Inheritance

```python
class Base1: pass
class Base2: pass
class Derived(Base1, Base2): pass  # TODO: constructor from super

issubclass(Derived, Base1); issubclass(Derived, Base2)
issubclass(Derived, Derived)  # Quirky, good to know

i = Derived()
i.__class__ == Derived
isinstance(i, Derived)
isinstance(i, Base1); isinstance(i, Base2)
```
TODO `super()` and its usage in `__init__()`
TODO lowercase str vs String and the result of type()

### Modules (files) and packages (dirs)

TODO

```python
```

### Context Managers

See `contextlib` module for tools.

```python
with open('file.txt', 'w') as f:
    f.write('foo')
```

### IO

TODO

```python
print()
open()
# CLI args
```

### Decorators

### Type annotations

### Metaprogramming

### Concurrency

### async and await

```python
```

### Debugging

```python
print(f'{var=}')  # prints `var=value`
breakpoint()  # drops to Pdb

```
