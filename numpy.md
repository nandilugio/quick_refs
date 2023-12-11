NumPy
+++++

[NumPy docs](https://numpy.org/doc/stable/index.html).

[NumPy](https://numpy.org/) is a library for scientific computing in Python. It:
- is based on an efficient multidimensional array implementation using C and BLAS (Basic Linear Algebra Subprograms) libraries. Arrays are stored in contiguous memory, and operations are optimized for them. Programming in NumPy avoids looping in favor of those optimized operations, and performance is much closer to C than Python.
- is the basis for many other libraries like SciPy, Matplotlib, Pandas, TensorFlow, PyTorch, etc.

The [NumPy n-dimensional arrays `np.ndarray`](https://numpy.org/doc/stable/reference/arrays.html) are:
- homogeneous: all elements have the same [data type `a.dtype`](https://numpy.org/doc/stable/reference/arrays.dtypes.html), which can be one of many NumPy optimized types, python types, etc.
- multidimensional: have a number of dimensions `a.ndim`, eg. 3, and a `a.shape`, eg. `(60,10,3)` for eg. 3D coords of 10 points in 60 frames.
- mutable: can be reshaped, transposed, etc.
- operation results do upcasting: the result is upcasted to the most general type, eg. `int` + `float` = `float`.

**NOTE:** We'll assume `import numpy as np` throughout the doc.

```python
a = np.array([[1,2,3],[4,5,6]])
# array([[1, 2, 3],
#        [4, 5, 6]])

a.dtype # type of elements
# dtype('int64')  # others are: int32, float64, complex128, bool, etc.
a.ndim == 2  # number of dimensions
a.shape == (2, 3)  # sizes per dimension
a.size == 6  # number of elements
a.itemsize == 8  # element size in bytes
```

Constructing arrays
-------------------

There are many constructors, like `np.array()`, building from _array\_like_ objects, like (nested) lists, tuples, etc. Others are:

- `np.empty()`, for arrays of uninitialized values (careful! whatever there is in memory).
- `np.zeros()` and `np.ones()`, for arrays of zeros and ones, respectively.
- `np.arange()`, for arrays of evenly spaced values (steps).
- `np.linspace()`, for arrays of evenly spaced values, with a specified number of elements.
- `rng = np.random.default_rng(1); rng.random()`, for arrays of random values.
- `np.r_[]` and `np.c_[]`, for arrays of concatenated values, row-wise and column-wise, respectively.

See all in the [NumPy docs](https://numpy.org/doc/stable/reference/routines.array-creation.html).

```python
np.arange(4, 10, 2)  # start, stop, step
# array([4, 6, 8])
np.linspace(0, 1, 5)  # start, stop, count
# array([0.  , 0.25, 0.5 , 0.75, 1.  ])
np.r_[1:4, 0, 4]  # see more at https://numpy.org/doc/stable/reference/generated/numpy.r_.html
# array([1, 2, 3, 0, 4])
```

Data types
----------

See them in the [NumPy docs](https://numpy.org/doc/stable/reference/arrays.dtypes.html).

NumPy has many optimized data types, all instances of `np.dtype`, that specify the byte coding for that type in memory. Here are some examples:

| dtype object   | Type string  | Array protocol | Python equivalent   |
|----------------|--------------|----------------|---------------------|
| -              | 'bool'       | '?',           | bool                |
| np.int32       | 'int32'      | 'i4'           | -                   |
| np.int64       | 'int64'      | 'i8'           | int                 |
| np.uint32      | 'uint32'     | 'u4'           | -                   |
| np.float64     | 'float64'    | 'f8'           | float               |
| np.complex128  | 'complex128' | 'c16'          | complex             |
| -              | 'bytes'      | 'S'            | bytes               |
| -              | 'unicode'    | 'U'            | str (Python 3)      |
| np.datetime64  | 'datetime64' | 'M8'           | (datetime.datetime )|
| np.timedelta64 | 'timedelta64'| 'm8'           | (datetime.timedelta)|
| -              | 'object'     | 'O'            | object              |

The _type strings_ and _array protocol strings_ can be used to build a type object as `np.dtype('int32')`, to specify the type of an array as `np.array([1,2,3], dtype='int32')`, etc.

The array protocol strings can be prepended with `<` or `>` to specify the endianness, eg. `np.dtype('<i4')` for little-endian 32-bit integer. The default is native endianness.

Most of the types had a named dtype object but were deprecated, so where you'd have used `np.int` you now use plain python `int`, or the concrete `np.int64`. When really needed, aliases like `np.int_` exist.

### Structured data types

TODO

Indexing and slicing
--------------------

### Python style

Python style indexing and slicing work as expected. Indexes are zero-based, and slices are half-open, meaning `a[1:3]` is `[a[1], a[2]]`.

```python
pya = [[1,2,3],[4,5,6],[7,8,9]]
npa = np.array(pya)
pya == npa.tolist()  # to be clear

pya[1] == npa[1].tolist() == [4,5,6]
pya[1][2] == npa[1][2] == 6  # note how each indexing operation strips a dimension
pya[1:][1] == npa[1:][1].tolist() == [7,8,9]  # slicing first [1:3] is last 2 rows (no dim stripping), then [1] is second row _of those_
```

In Python, `a[i, j]` is exactly the same as `a[(i, j)]`. It can be handy to put `i` and `j` in a tuple and then do the indexing with that.

```python
i = (1,2)
pya[i] == a[1,2] == 6
```

### N-dimensional NumPy style

Then NumPy introduces n-dimensional indexing, using tuples of indexes, one per dimension eg. `a[2,3]`. The slicing operator `:` still works, and there's also `...` representing as many `:` as needed to complete the indexing tuple.

```python
a = np.arange(3*3*3).reshape(3,3,3)
# array([[[ 0,  1,  2],
#         [ 3,  4,  5],
#         [ 6,  7,  8]],
#
#        [[ 9, 10, 11],
#         [12, 13, 14],
#         [15, 16, 17]],
#
#        [[18, 19, 20],
#         [21, 22, 23],
#         [24, 25, 26]]])

# Regular indexing

a[1]  # strips one dimension
# array([[ 9, 10, 11],
#        [12, 13, 14],
#        [15, 16, 17]])
a[1,2]  # strips two dimensions
# array([15, 16, 17])
a[1,2,1]  # strips all 3 dimensions
# 16
a[1,2,1] == a[1][2][1]  # because of dimension stripping! See below

# Slicing

a[1,:]  # all rows, second column. Same as a[1] above: missing indexes to the right are assumed to be `:` (complete slices)
# array([[ 9, 10, 11],
#        [12, 13, 14],
#        [15, 16, 17]])
a[..., 1]  # same as a[:,:,1]
# array([[ 1,  4,  7],
#        [10, 13, 16],
#        [19, 22, 25]])

v = np.arange(10)
# array([0, 1, 2, 3, 4, 5, 6, 7, 8, 9])
v[1:3]  # half-open as regular python: [1,3) so not including the 3-indexed element
# array([1, 2])
v[3:-2] # same as v[-7:8]. Negative indexes count from the end
# array([3, 4, 5, 6, 7])
v[:]  # start and stop default to beginning and end
# array([0, 1, 2, 3, 4, 5, 6, 7, 8, 9])
v[1:7:2]  # start], stop[, step
# array([1, 3, 5])
v[::-2]  # step can also be negative (reverse)
# array([9, 7, 5, 3, 1])
```

**/!\\** Note that indexing repeatedly (`[x][y]`) is normally not the same as n-dimensional indexing (`[x,y]`)! 

```python
# This is true _only_ because the 1st indexing collapses the array to 1D, yielding [2,3,4], not [[2,3,4]]
a[1][2] == a[1,2]

# When slicing though:
a[1:3][1] == a[1:3,1]  # [1:3] is last 2 rows, then [1] is second row _of those_. No dim collapse!

# The following expressions are thus _not_ equivalent:
a[:][1]  # a[:] is equal to a, and a[1] is the second row
# array([4, 5, 6])
a[:,1]  # a[:,1] is the second column. This is n-dimensional indexing: all rows, second column
# array([2, 5])
```

### Slices and views

Slices can be assigned to, but only if the shape of the slice matches the shape of the right-hand side. Slices can be multidimensional, and can be used to assign to multiple elements at once.

```python
a = np.array([[1,2,3],[4,5,6],[7,8,9]])

a[1,2] = 0
# a is now
# array([[1, 2, 3],
#        [4, 5, 0],
#        [7, 8, 9]])

a[1,:] = [0,0,0]
# a is now
# array([[1, 2, 3],
#        [0, 0, 0],
#        [7, 8, 9]])
```
**/!\\** NumPy slices are views of the original array, not copies! modifying them modifies the original array. Continuing the example above:

```python
b = a[1,:]
b[1] = 1
# a is now
# array([[1, 2, 3],
#        [0, 1, 0],
#        [7, 8, 9]])
```

### By arrays of indexes

On vectors it's more obvious:

```python
a = np.arange(5) ** 2
# array([ 0,  1,  4,  9, 16])

a[[0,2,4]]
# array([ 0,  4, 16])

a[[0,2,4]] = 0
# a is now
# array([0, 1, 0, 9, 0])
```

On multidimensional arrays, we can pass one vector-like per dimension:

```python
a = np.arange(6).reshape(3,2)
# array([[0, 1],
#        [2, 3],
#        [4, 5]])

i = [0,2]  # row indexes
j = [1,0]  # column indexes

a[i]  # same as a[i,:] or a[i,...]
# array([[0, 1],
#        [4, 5]])

a[i,j]
# array([1, 4])

a[i,j] = 0
# a is now
# array([[0, 0],
#        [2, 3],
#        [0, 5]])
```

A useful application of this is to map related values, eg. colors codes to rgb values in the following example:

```python
color_map = np.array([[0,  0,  0  ],   # black
                      [127,127,127],   # gray
                      [255,255,255]])  # white

# The following is a 12x10 image of a "7", a bit in the style of the MNIST dataset images.
# It's encoded 0=black, 1=gray, 2=white. Note those numbers are indexes into the color_map array
image = np.array([[0, 0, 0, 0, 0, 0, 0, 0, 0, 0],
                  [0, 0, 1, 1, 1, 1, 1, 1, 1, 0],
                  [0, 0, 1, 2, 2, 2, 2, 2, 1, 0],
                  [0, 0, 1, 1, 1, 1, 1, 2, 1, 0],
                  [0, 0, 0, 0, 0, 0, 1, 2, 1, 0],
                  [0, 0, 0, 0, 1, 1, 1, 2, 1, 0],
                  [0, 0, 0, 0, 1, 2, 2, 2, 1, 0],
                  [0, 0, 0, 0, 1, 1, 1, 2, 1, 0],
                  [0, 0, 0, 0, 0, 0, 1, 2, 1, 0],
                  [0, 0, 0, 0, 0, 0, 1, 2, 1, 0],
                  [0, 0, 0, 0, 0, 0, 1, 2, 1, 0],
                  [0, 0, 0, 0, 0, 0, 1, 1, 1, 0]])
image.shape == (12,10)

rendered_image = color_map[image]  # 12x10x3 array of rgb values
rendered_image.shape == (12,10,3)
```

### By boolean arrays

```python
a = np.arange(5) ** 2
# array([ 0,  1,  4,  9, 16])

a[[False, False, True, False, True]]
# array([ 4, 16])

# This notation is handy for filtering
a[a > 2]  # a > 2 is a boolean array of the same shape as a
# array([ 4,  9, 16])

# Handy too: all elements > 2 are set to 0
a[a > 2] = 0
# a is now
# array([0, 1, 0, 0, 0])
```

On multidimensional arrays, we can pass one boolean array per dimension:

```python
a = np.arange(6).reshape(3,2)
# array([[0, 1],
#        [2, 3],
#        [4, 5]])

i = np.array([True, False, True])  # row indexes
j = np.array([False, True])        # column indexes

a[i]  # same as a[i,:] or a[i,...]
# array([[0, 1],
#        [4, 5]])

a[i,j]
# array([1, 5])

a[i,j] = 0
# a is now
# array([[0, 0],
#        [2, 3],
#        [4, 0]])
```
Indexing with strings
---------------------

TODO

The `ix_()` function
--------------------

TODO

Iterating
---------

Iterating in python is possible, although normally avoided in favor of more efficient NumPy operations.

```python
# Regular iteration goes over 1st dimension (here rows)
for row in a:
    print(row)

# We can also iterate over all elements
for elem in a.flat:  # `flat` is an iterator
    print(elem)
```

Operating on arrays
-------------------

### Operators

Arithmetic and comparison operators, as well as in-place operators like `+=` or `*=` work element-wise as expected. There are NumPy [universal functions `ufunc`](https://numpy.org/doc/stable/reference/ufuncs.html) for those operators. See below for more on `ufunc`s and interesting properties of those like broadcasting.

```python
a = np.array([2,2])
b = np.array([2,3])
a == b
# array([ True, False])
b > 2
# array([False,  True])
a + b
# array([4, 5])
a += 1
# a is now array([3, 3])
a += np.array([3,6])
# a is now array([6, 9])
```

Matrix product is done with `np.dot()` or the `@` operator.

```python
a = np.array([[1,2],[3,4]])
b = np.array([[5,6],[7,8]])
a @ b  # or np.dot(a, b)
# array([[19, 22],
#        [43, 50]])
```

### Methods and functions

Other operations are implemented either:

- as methods of the [`ndarray` class](https://numpy.org/doc/stable/reference/arrays.ndarray.html#array-methods), normally applying to the array as though it were a list of numbers, regardless of its shape, unless specifying the `axis` argument, along which to perform the operation:

```python
a = np.array([[1,2,3],[4,5,6]])

a.sum()
# 21
a.sum(axis=0)
# array([5, 7, 9])
a.sum(axis=1)
# array([ 6, 15])

a.T  # transpose
# array([[1, 4],
#        [2, 5],
#        [3, 6]])

a.reshape(3,2)  # reshape to 3 rows, 2 columns
# array([[1, 2],
#        [3, 4],
#        [5, 6]])

a.resize(3,2)  # same as above but in-place
```

- as [universal functions (`ufunc`)](https://numpy.org/doc/stable/reference/ufuncs.html) in the `np` module, normally applying element-wise to the array:

```python
np.sin(a)
# array([[ 0.84147098,  0.90929743,  0.14112001],
#        [-0.7568025 , -0.95892427, -0.2794155 ]])
```

NumPy also implements [broadcasting](https://numpy.org/doc/stable/user/basics.broadcasting.html), which is a set of rules for applying binary `ufunc`s to arrays of different shapes. The smaller array is "broadcast" across the larger array so that they have compatible shapes. Broadcasting provides a means of vectorizing array operations so that looping occurs in C instead of Python. It does not actually create multiple copies of the smaller array, so it's memory efficient.

```python
a = np.array([[1,2,3],[4,5,6]])
b = np.array([7,8,9])  # note below how b is broadcasted to array([[7,8,9],[7,8,9]])
a + b  # the `+` operator is equivalent to the `np.add()` universal function
# array([[ 8, 10, 12],
#        [11, 13, 15]])
```

- as other functions inplementing very diverse range of operations:

```python
np.reshape(a, (3,2))  # flattens and then reshape to 3 rows, 2 columns
# array([[1, 2],
#        [3, 4],
#        [5, 6]])

np.where(a < 5, a, -1)  # filters according to predicate. Use -1 as default
# array([[ 1,  2,  3],
#        [ 4, -1, -1]])
# TODO: a < 5 is a boolean array right? So appart from the default value, is this equivalent to a[a < 5]?

np.vstack((a, b))  # stack vertically. There's also hstack(), vsplit(), hsplit(), etc.
# array([[1, 2, 3],
#        [4, 5, 6],
#        [7, 8, 9]])
```

There are many modules inplementing a variety of operations, like `linalg` for linear algebra:

```python
np.linalg.det(a)
# -2.0000000000000004
np.linalg.inv(a)
# array([[-2. ,  1. ],
#        [ 1.5, -0.5]])
```

See them all in the [NumPy docs](https://numpy.org/doc/stable/reference/routines.html).

References and copying
----------------------

Python already passes objects by reference. Also, python slices are shallow copies, meaning are a new container with references to the original elements.

NumPy slices are views referencing the original array, so modifying them modifies the original array. `np.copy()` and `np.copyto()` can be used to make full copies, and `np.view()` to make a view of the whole array, which can then be eg. trasposed without affecting the original array but still sharing the data.

```python
# Python
pya = [1,2,3]
pyb = pya[:]  # new container with numbers, so an effective full copy
pyb[1] = 0
# pya is still [1,2,3]

# NumPy
npa = np.array([1,2,3])
npb = npa[:]  # a view!
npb[1] = 0
# npa is now array([1,0,3])

# Python and NumPy behave the same in this example
a = [[1,2,3],[4,5,6]]  # or a = np.array([[1,2,3],[4,5,6]])
b = a[:]  # new container with references to the nested lists
b[1][1] = 0
# a is now [[1,2,3],[4,0,6]]
# or array([[1,2,3],[4,0,6]])

# Views can be created explicitly
a = np.arangge(3)
v = a.view()
v is not a
v.base is a
a.flags.owndata and not v.flags.owndata

# Reshaping, and other transformations not affecting data, doesn't affect the original
v.reshape(3,1)
v.shape == (3,1)
a.shape == (3,)
```








TODO structured arrays


TODO (AI generated, check!): 

NumPy also implements [masked arrays](https://numpy.org/doc/stable/reference/maskedarray.html), which are arrays with a mask, ie. a boolean array of the same shape, where `True` values indicate that the corresponding element of the original array is invalid. Masked arrays are useful when we want to perform operations on arrays but ignore some elements, like invalid values, or values outside a certain range.

```python
a = np.array([1,2,3,4,5])
m = np.array([True, False, False, True, False])
ma = np.ma.array(a, mask=m)
ma
# masked_array(data=[1, 2, 3, 4, 5],
#              mask=[ True, False, False,  True, False],
#        fill_value=999999)
ma.mean()
# 3.0
```
