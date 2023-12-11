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
1.2e3 == 1200  # Scientific notation

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

TODO: greenlets and gevent
TODO: Locks and Thread-safety

#### Preemptive multitasking: Multiprocessing

Processes are OS-level constructs. They're scheduled by the OS and can run in parallel. They don't share memory, but they can communicate by writing to shared (mp capable: threadsafe?) structures passed as params.

```python
import multiprocessing
import queue # for the queue.Empty exception

# Memory is not shared between processes, and we can't get returned values, but
# we can use queues to communicate. We'll pass them as parameters to the worker function
pending = multiprocessing.Queue()
done = multiprocessing.Queue()

def work(pending, done):
    while True: # Keep working until there's no more work to do
        try:
            data = pending.get_nowait()
        except queue.Empty:
            print(f'{multiprocessing.current_process().name}: No more work to do!')
            break
        print(f'{multiprocessing.current_process().name}: Working on {data}...')
        done.put(data * 2)
        print(f'{multiprocessing.current_process().name}: Done!')

def main():
    print('Putting some work in the queue...')
    for i in range(20):
        pending.put(i)

    print('Creating a process for each CPU...')
    processes = [multiprocessing.Process(target=work, args=(pending, done)) \
                    for _ in range(multiprocessing.cpu_count())]
    
    print('Starting the processes...')
    for p in processes:
        p.start()
    
    print('Waiting for the processes to finish...')
    for p in processes:
        p.join()

    print('Getting the results...')
    results = []
    while not done.empty():
        results.append(done.get())
    print(f'{results=}')

# It's important to use the main guard, otherwise we could attempt to start
# a child process before the parent process has finished bootstraping.
if __name__ == '__main__':
    main()
```

#### Preemptive multitasking: Threads 

Python threads are green threads, meaning they're scheduled by the interpreter, not the OS. They're not run in parallel, but interleaved.

They share memory, so they can communicate by writing to shared variables.

```python
import threading
import time

results = [] # Memory is shared between threads. Lists are thread-safe

def work(data):
    global results
    time.sleep(2)
    results.append(data * 2) # Cannot return values but can use global vars
    print('Work done!')

thread1 = threading.Thread(target=work, args=(2,)) 
thread2 = threading.Thread(target=work, args=(2,))
thread1.start()
thread2.start()

print('Is thread1 alive?', thread1.is_alive()) # True
print('Results before join:', results) # []

# Waits for the threads to finish (each join blocks until the thread is done)
thread1.join()
thread2.join()

print('Results after join:', results) # [4, 4]
```

Threads can be pooled too:

```python
from concurrent.futures import ThreadPoolExecutor
import time

def work(data):
    time.sleep(2)
    return data * 2

print("Starting ThreadPoolExecutor")
with ThreadPoolExecutor(max_workers=5) as executor:
    future = executor.submit(work, 2)  # Non-blocking
    futures = executor.map(work, [1, 2, 3])  # Non-blocking
    print("Done submitting work to ThreadPoolExecutor")
    print('Is future running?', future.running()) # True
    print('Is future cancelled?', future.cancelled()) # False
    print('Is future done?', future.done()) # False
    try:
        print('Is future result ready?')
        print(future.result(timeout=1)) # Blocks until done or times out
    except TimeoutError:
        print('No')
# The executor will wait for all threads to finish before exiting the `with` block, like `pool.shutdown(wait=True)`
print("Out of ThreadPoolExecutor's context manager")

# `result()` without timeout, blocks until done (though it is done now)
result = future.result()
print(result)

# `map()` returns an iterator. When casted, blocks until all are done
results = list(futures)
print(results)
```

#### Cooperative multitasking: Coroutines (asyncio)

We first need an __event loop__. It's a scheduler that runs coroutines, giving control to each one while the others are blocked waiting (`await`).

The easiest way to start the event loop it is by calling `asyncio.run(coro)` from the main thread, where `coro` is the initial coroutine to run.

##### Awaitables

Awaitables can be awaited using the `await` keyword. It's only valid inside coroutines. Await calls are blocking, and yield control to the event loop.

```
Awaitable <--- Coroutine
    ^--- Future <--- Task
```

- __Coroutines__: defined using the `async` keyword. They're functions that can be paused and resumed while waiting (`await`). They're run by awaiting them (blocking) or sheduling them as a task (non-blocking).
- __Futures__: Represent the result of an asynchronous operation. They're created by the event loop when a coroutine is scheduled to run. They can be awaited to block execution until the result is available. Not to be confused with `concurrent.futures.Future` used with threads and processes.
- __Tasks__: Wrap coroutines. They're scheduled to run ASAP in the event loop. Tasks have some extra features like callbacks, cancelling, etc.

```python
import asyncio

async def log_seconds(n):
    for i in range(n):
        print(i)
        # Sleep emulates I/O and yields control to the loop
        await asyncio.sleep(1)

async def sim_fetch_data():
    await asyncio.sleep(3) # Emulates I/O
    return 'data from the webssss'

async def main():
    # Create a _task_ from a coroutine. It's scheduled to be run ASAP
    # by the event loop.
    task = asyncio.create_task(log_seconds(10))

    # Do some work. This will run for ~1 second.
    print('Working in main()...')
    for i in range(50_000_000): pass
    # It doesn't run still, since we've not stopped for I/O
    print('Still not logging seconds...')

    # We'll give it a chance to start logging
    await asyncio.sleep(0.01) # <-- I/O!

    # The above will log the 1st second and stop for I/O so, since
    # there aren't any other awaitables registered in the loop, 
    # control will come back here.

    # Create a _future_ from a coroutine. Again, will be run ASAP
    future = asyncio.ensure_future(sim_fetch_data())

    # Wait for the future to finish. This blocks execution until the
    # future is done, also yielding control back to the loop like I/O.
    # Since this one takes 3 seconds, we'll continue seeing seconds
    # logged by the earlier task. 
    data = await future
    
    # Now we've got the data, so when control gets back, we'll print it
    print(data)

    # Since we've got control and we're returning now, the loop will just stop and the program will exit, even if log_seconds hasn't finished.

asyncio.run(main())
```

Awaitables can also be awaited in parallel using `asyncio.gather()`. It takes a list of awaitables and returns a future that resolves to a list of results after _all_ awaitables are done.

```python
asyncio.gather(
    log_seconds(10),
    sim_fetch_data(),
    return_exceptions=True,  # If any awaitable raises an exception, it'll be included in the results
)
```

TODO: async iterators and context managers

### Debugging

```python
print(f'{var=}')  # prints `var=value`
breakpoint()  # drops to Pdb

```
