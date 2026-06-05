# chapters 14-15

# Chapter 14

This chapter covers the options for handling data files. Depending on the data and the desired format, there are several ways to read and write. The main topics are: file handling, NumPy methods, pickling, shelves, reading and writing Matlab data files, and reading and writing images.

## 14.1 File handling

File input and output (I/O) is essential in many scenarios:

- Working with measured or scanned data stored in files that need to be read for analysis.
- Interacting with other programs: saving results so they can be imported into other applications, and vice versa.
- Storing information for future reference or comparisons.
- Sharing data and results with others, possibly on other platforms using other software.

### 14.1.1 Interacting with files

In Python, an object of the type `file` represents the contents of a physical file stored on a disk. A new file object is created with `open`:

```python
# creating a new file object from an existing file
myfile = open('measurement.dat', 'r')
print(myfile.read())
```

A file has to be closed before it can be re-read or used by other applications:

```python
myfile.close()  # closes the file object
```

This is not so simple, because an exception might be triggered before the call to `close` is executed, which would skip the closing code. The safe way is to use a context manager (the `with` keyword, see Section 12.1.3):

```python
with open('measurement.dat', 'r') as myfile:
    ...  # use myfile here
```

This ensures the file is closed when you exit the `with` block, even if an exception is raised inside it. Consider why the construct is desirable:

```python
myfile = open(name, 'w')
myfile.write('some data')
a = 1 / 0
myfile.write('other data')
myfile.close()
```

An exception is raised before the file is closed; the file remains open and there is no guarantee of what data is written or when. The proper way to achieve the same result is:

```python
with open(name, 'w') as myfile:
    myfile.write('some data')
    a = 1 / 0
    myfile.write('other data')
```

In that case, the file is cleanly closed just after the exception (here, `ZeroDivisionError`) is raised. There is no need to close the file explicitly.

### 14.1.2 Files are iterables

A file is, in particular, iterable (see Section 9.3). Files iterate over their lines:

```python
with open(name, 'r') as myfile:
    for line in myfile:
        data = line.split(';')
        print(f'time {data[0]} sec temperature {data[1]} C')
```

The lines of the file are returned as strings. The string method `split` converts a string to a list of strings:

```python
data = 'aa;bb;cc;dd;ee;ff;gg'
data.split(';')   # ['aa', 'bb', 'cc', 'dd', 'ee', 'ff', 'gg']

data = 'aa bb cc dd ee ff gg'
data.split(' ')   # ['aa', 'bb', 'cc', 'dd', 'ee', 'ff', 'gg']
```

Since `myfile` is iterable, we can also extract directly into a list:

```python
data = list(myfile)
```

### 14.1.3 File modes

The function `open` takes at least two arguments: the filename, and a string describing the way the file will be used. The basic modes are:

| Mode | Meaning |
| --- | --- |
| `'r'` | read only |
| `'r+'` | read / write |
| `'rb'` | read in byte mode |
| `'a'` | append (write to the end of the file) |
| `'w'` | (over-)write the file |
| `'wb'` | (over-)write the file in byte mode |

```python
with open('file1.dat', 'r') as ...    # read only
with open('file2.dat', 'r+') as ...   # read/write
with open('file3.dat', 'rb') as ...   # read in byte mode
with open('file4.dat', 'a') as ...    # append (write to the end of the file)
with open('file5.dat', 'w') as ...    # (over-)write the file
with open('file6.dat', 'wb') as ...   # (over-)write the file in byte mode
```

The modes `'r'`, `'r+'`, and `'a'` require that the file exists, whereas `'w'` creates a new file if none with that name exists. The following example appends data at the end of the file without modifying what is already there. Note the line break, `\n`:

```python
with open('file3.dat', 'a') as myfile:
    myfile.write('something new\n')
```

## 14.2 NumPy methods

NumPy has built-in methods for reading and writing NumPy array data to text files: `numpy.loadtxt` and `numpy.savetxt`.

### 14.2.1 savetxt

Writing an array to a text file is simple:

```python
savetxt(filename, data)
```

Two useful parameters, given as strings, are `fmt` and `delimiter`, which control the format and the delimiter between columns. The defaults are a space for the delimiter and `%.18e` for the format (exponential format with all digits):

```python
x = range(100)                          # 100 integers
savetxt('test.txt', x, delimiter=',')   # use comma instead of space
savetxt('test.txt', x, fmt='%d')        # integer format instead of float with e
```

### 14.2.2 loadtxt

Reading into an array from a text file is done with:

```python
filename = 'test.txt'
data = loadtxt(filename)
```

Because each row in an array must have the same length, each row in the text file must have the same number of elements. As with `savetxt`, the default type is float and the delimiter is a space. These can be set with the parameters `dtype` and `delimiter`. Another useful parameter is `comments`, which marks the symbol used for comments in the data file:

```python
data = loadtxt('test.txt', delimiter=';')   # data separated by semicolons
# read to integer type, comments in file begin with a hash character
data = loadtxt('test.txt', dtype=int, comments='#')
```

## 14.3 Pickling

The read and write methods above convert data to strings before writing. Complex types (such as objects and classes) cannot be written this way. With Python's module `pickle`, you can save any object, and also multiple objects, to a file. There are two main methods: `dump`, which saves a pickled representation of a Python object to a file, and `load`, which retrieves a pickled object from the file:

```python
import pickle
with open('file.dat', 'wb') as myfile:
    a = random.rand(20, 20)
    b = 'hello world'
    pickle.dump(a, myfile)   # first call: first object
    pickle.dump(b, myfile)   # second call: second object
```

```python
import pickle
with open('file.dat', 'rb') as myfile:
    numbers = pickle.load(myfile)   # restores the array
    text = pickle.load(myfile)      # restores the string
```

Note the order in which the two objects are returned. Besides the two main methods, it is sometimes useful to serialize a Python object to a string instead of a file, using `dumps` and `loads`:

```python
a = [1, 2, 3, 4]
pickle.dumps(a)   # returns a bytes object
b = {'a': 1, 'b': 2}
pickle.dumps(b)   # returns a bytes object
```

A good example of using `dumps` is when you need to write Python objects or NumPy arrays to a database. Besides the `pickle` module, there is an optimized version called `cPickle`, written in C, which is an option if you need fast reading and writing. The data produced by `pickle` and `cPickle` is identical and can be interchanged.

## 14.4 Shelves

Objects in dictionaries can be accessed by keys. There is a similar way to access particular data in a file by first assigning it a key, using the module `shelve`:

```python
from contextlib import closing
import shelve as sv

# opens a data file (creates it before if necessary)
with closing(sv.open('datafile')) as data:
    A = array([[1, 2, 3], [4, 5, 6]])
    data['my_matrix'] = A   # here we created a key
```

In Section 14.1.1, we saw that the built-in command `open` generates a context manager. In contrast, `sv.open` does not create a context manager by itself; the command `closing` from the module `contextlib` is needed to transform it into an appropriate context manager. Restoring the file is done as follows:

```python
from contextlib import closing
import shelve as sv

with closing(sv.open('datafile')) as data:   # opens a data file
    A = data['my_matrix']   # here we used the key
    ...
```

A shelve object has all dictionary methods, for example `keys` and `values`, and can be used in the same way as a dictionary. Note that changes are only written to the file after one of the methods `close` or `sync` has been called.

## 14.5 Reading and writing Matlab data files

SciPy has the ability to read and write data in Matlab's `.mat` file format using the module `scipy.io`. The commands are `loadmat` and `savemat`. To load data:

```python
import scipy.io
data = scipy.io.loadmat('datafile.mat')
```

The variable `data` now contains a dictionary, with keys corresponding to the variable names saved in the `.mat` file. The variables are in NumPy array format. Saving to `.mat` files involves creating a dictionary with all the variables you want to save (variable name and value), then calling `savemat`:

```python
data = {}
data['x'] = x
data['y'] = y
scipy.io.savemat('datafile.mat', data)
```

This saves the NumPy arrays `x` and `y` in Matlab's internal file format, preserving variable names.

## 14.6 Reading and writing images

The module `PIL.Image` (the Pillow module) comes with functions for handling images. The following reads a JPEG image, prints its shape and type, creates a resized image, and writes the new image to a file:

```python
import PIL.Image as pil   # imports the Pillow module

# read image to array
im = pil.open("test.jpg")
print(im.size)   # (275, 183)
# Number of pixels in horizontal and vertical directions

# resize image
im_big = im.resize((550, 366))
im_big_gray = im_big.convert("L")   # Convert to grayscale

im_array = array(im)
print(im_array.shape)
print(im_array.dtype)   # uint8

# write result to new image file
im_big_gray.save("newimage.jpg")
```

PIL creates an image object that can easily be converted to a NumPy array. As an array object, images are stored with pixel values in the range 0...255 as 8-bit unsigned integers (`uint8`). The third shape value shows how many color channels the image has. A value of 3 means a color image, with values stored in this order: red `im_array[:, :, 0]`, green `im_array[:, :, 1]`, and blue `im_array[:, :, 2]`. A grayscale image would have only one channel.

The PIL module also contains many useful basic image processing functions, including filtering, transforms, measurements, and conversion from a NumPy array back to a PIL image object:

```python
new_image = pil.fromarray(im_array)
```

## 14.7 Summary

File handling is inevitable when dealing with measurements and other sources of large amounts of data, as well as for communication with other programs and tools. You learned to see a file as a Python object, like others, with important methods such as `readlines` and `write`. We showed how files can be protected by special modes, which may allow only read or write access. The way you write to a file often influences the speed of the process. We saw how data is stored by pickling or by using the `shelve` method.

# Chapter 15

In this chapter, we focus on two aspects of testing for scientific programming: *what* to test in scientific computing, and *how* to test. We distinguish between manual and automated testing. Manual testing is what every programmer does to quickly check that a program is doing what it should. Automated testing is the refined, automated variant of that idea.

## 15.1 Manual testing

During code development, you do a lot of small tests to check functionality. This could be called manual testing. Typically, you test whether a given function does what it is supposed to do by manually testing it in an interactive environment. For instance, suppose you implement the bisection algorithm, which finds a zero (root) of a scalar non-linear function. To start the algorithm, an interval has to be given with the property that the function takes different signs on the interval boundaries. You would then test that:

- A solution is found when the function has opposite signs at the interval boundaries.
- An exception is raised when the function has the same sign at the interval boundaries.

Manual testing, as necessary as it may seem, is unsatisfactory. Once you have convinced yourself that the code works, the tests made during development are often forgotten or even deleted. As soon as you change a detail and things no longer work correctly, you might regret that your earlier tests are no longer available.

## 15.2 Automatic testing

The correct way to develop any piece of code is to use automatic testing. The advantages are:

- The automated repetition of a large number of tests after every code refactoring and before any new versions are launched.
- Silent documentation of the use of the code.
- Documentation of the test coverage of your code: did things work before a change, or was a certain aspect never tested?

Changes in the program (in particular its structure) that do not affect its functionality are called *code refactoring*. We suggest developing tests in parallel with coding. Good design of tests is an art of its own, and rarely is there an investment that guarantees such a good pay-off in development time savings as the investment in good tests.

### 15.2.1 Testing the bisection algorithm

An implementation of the bisection algorithm can have the following form:

```python
def bisect(f, a, b, tol=1.e-8):
    """
    Implementation of the bisection algorithm
    f real valued function
    a,b interval boundaries (float) with the property
        f(a) * f(b) <= 0
    tol tolerance (float)
    """
    if f(a) * f(b) > 0:
        raise ValueError("Incorrect initial interval [a, b]")
    for i in range(100):
        c = (a + b) / 2.
        if f(a) * f(c) <= 0:
            b = c
        else:
            a = c
        if abs(a - b) < tol:
            return (a + b) / 2
    raise Exception('No root found within the given tolerance {tol}')
```

We assume this is stored in a file named `bisection.py`. As the first test case, we test that the zero of the function is found:

```python
def test_identity():
    result = bisect(lambda x: x, -1., 1.)
    expected = 0.
    assert allclose(result, expected), 'expected zero not found'

test_identity()
```

Here you meet the Python keyword `assert` for the first time. It raises the exception `AssertionError` if its first argument returns `False`. Its optional second argument is a string with additional information. We use the function `allclose` to test for equality of floats. We have to manually run the test in the line `test_identity()`.

A second test checks that `bisect` raises an exception when the function has the same sign on both ends of the interval (here we assume the exception is `ValueError`):

```python
def test_badinput():
    try:
        bisect(lambda x: x, 0.5, 1)
    except ValueError:
        pass
    else:
        raise AssertionError()

test_badinput()
```

An `AssertionError` is raised if the exception is not of the type `ValueError`.

Another useful test is the *edge case* test. Here we test arguments or user input which are likely to create mathematically undefined situations or unforeseen states. For instance, what happens if both bounds are equal?

```python
def test_equal_boundaries():
    result = bisect(lambda x: x, 0., 0.)
    expected = 0.
    assert allclose(result, expected), \
        'test equal interval bounds failed'

def test_reverse_boundaries():
    result = bisect(lambda x: x, 1., -1.)
    expected = 0.
    assert allclose(result, expected), \
        'test reverse interval bounds failed'

test_equal_boundaries()
test_reverse_boundaries()
```

A test checks that the program unit does what is demanded by its specification. In the preceding example, we assumed the specification states that in the reverse case the two values should tacitly be interchanged. An alternative would be to specify that this situation is wrong input, in which case we would have tested for an appropriate exception, for example `ValueError`.

### 15.2.2 Using the unittest module

The Python module `unittest` greatly facilitates automated testing. It requires that we rewrite our previous tests to be compatible. The first test would be rewritten in a class:

```python
from bisection import bisect
import unittest

class TestIdentity(unittest.TestCase):
    def test(self):
        result = bisect(lambda x: x, -1.2, 1., tol=1.e-8)
        expected = 0.
        self.assertAlmostEqual(result, expected)

if __name__ == '__main__':
    unittest.main()
```

The differences from the previous implementation: the test is now a method and part of a class that must inherit from `unittest.TestCase`. The test method's name must start with `test`. We may now use one of the assertion tools of `unittest`, namely `assertAlmostEqual`. Finally, the tests are run using `unittest.main`. We recommend writing the tests in a file separate from the code to be tested, which is why it starts with an `import`. A passing test returns:

```
Ran 1 test in 0.002s

OK
```

If we run it with a loose tolerance (e.g. `1.e-3`), a failure is reported:

```
F
======================================================================
FAIL: test (__main__.TestIdentity)
----------------------------------------------------------------------
Traceback (most recent call last):
  File "<ipython-input-11-e44778304d6f>", line 5, in test
    self.assertAlmostEqual(result, expected)
AssertionError: 0.00017089843750002018 != 0.0 within 7 places
----------------------------------------------------------------------
Ran 1 test in 0.004s

FAILED (failures=1)
```

Tests can and should be grouped together as methods of a test class:

```python
import unittest
from bisection import bisect

class TestIdentity(unittest.TestCase):
    def identity_fcn(self, x):
        return x

    def test_functionality(self):
        result = bisect(self.identity_fcn, -1.2, 1., tol=1.e-8)
        expected = 0.
        self.assertAlmostEqual(result, expected)

    def test_reverse_boundaries(self):
        result = bisect(self.identity_fcn, 1., -1.)
        expected = 0.
        self.assertAlmostEqual(result, expected)

    def test_exceeded_tolerance(self):
        tol = 1.e-80
        self.assertRaises(Exception, bisect, self.identity_fcn,
                          -1.2, 1., tol)

if __name__ == '__main__':
    unittest.main()
```

In the last test we used the method `unittest.TestCase.assertRaises`. It tests whether an exception is correctly raised. Its first parameter is the exception type (e.g. `ValueError`, `Exception`), its second argument is the function expected to raise the exception, and the remaining arguments are the arguments for this function. The command `unittest.main()` creates an instance of the class and executes those methods starting with `test`.

### 15.2.3 Test setUp and tearDown methods

The class `unittest.TestCase` provides two special methods, `setUp` and `tearDown`, which run before and after every call to a test method. This is needed, for instance, when testing generators, which are exhausted after every test. We demonstrate this by testing a program that checks the line in a file in which a given string occurs for the first time:

```python
class StringNotFoundException(Exception):
    pass

def find_string(file, string):
    for i, lines in enumerate(file.readlines()):
        if string in lines:
            return i
    raise StringNotFoundException(
        f'String {string} not found in File {file.name}.')
```

We assume this code is saved in a file named `find_in_file.py`. A test has to prepare and open a file, and remove it after the test:

```python
import unittest
import os   # used for, for example, deleting files
from find_in_file import find_string, StringNotFoundException

class TestFindInFile(unittest.TestCase):
    def setUp(self):
        file = open('test_file.txt', 'w')
        file.write('bird')
        file.close()
        self.file = open('test_file.txt', 'r')

    def tearDown(self):
        self.file.close()
        os.remove(self.file.name)

    def test_exists(self):
        line_no = find_string(self.file, 'bird')
        self.assertEqual(line_no, 0)

    def test_not_exists(self):
        self.assertRaises(StringNotFoundException, find_string,
                          self.file, 'tiger')

if __name__ == '__main__':
    unittest.main()
```

Before each test `setUp` is run, and after each test `tearDown` is executed. They guarantee that the test data is restored before the next test is executed.

If your tests do *not* change the test data, you may want to save time by setting up the data only once. This is done by the class method `setUpClass` (see Section 8.4 on class attributes and class methods):

```python
import unittest

class TestExample(unittest.TestCase):
    @classmethod
    def setUpClass(cls):
        cls.A = ...

    def Test1(self):
        A = self.A
        # assert something
        ...

    def Test2(self):
        A = self.A
        # assert something else
```

### 15.2.4 Parameterizing tests

We frequently want to repeat the same test with different datasets. With `unittest`, this requires us to automatically generate test cases with the corresponding methods injected. We first construct a test case with one or several methods that will be used when we later set up test methods. We use the bisection method again, and check that the returned values are really zeros of the given function:

```python
class Tests(unittest.TestCase):
    def checkifzero(self, fcn_with_zero, interval):
        result = bisect(fcn_with_zero, *interval, tol=1.e-8)
        function_value = fcn_with_zero(result)
        expected = 0.
        self.assertAlmostEqual(function_value, expected)
```

Then we dynamically create test functions as attributes of this class:

```python
test_data = [
    {'name': 'identity', 'function': lambda x: x,
     'interval': [-1.2, 1.]},
    {'name': 'parabola', 'function': lambda x: x**2 - 1,
     'interval': [0, 10.]},
    {'name': 'cubic', 'function': lambda x: x**3 - 2 * x**2,
     'interval': [0.1, 5.]},
]

def make_test_function(dic):
    return lambda self: \
        self.checkifzero(dic['function'], dic['interval'])

for data in test_data:
    setattr(Tests, f"test_{data['name']}", make_test_function(data))

if __name__ == '__main__':
    unittest.main()
```

The data is provided as a list of dictionaries. The function `make_test_function` dynamically generates a test function, which uses a particular data dictionary to perform the test with the previously defined method `checkifzero`. Finally, the command `setattr` is used to make these test functions methods of the class `Tests`.

### 15.2.5 Assertion tools

The following table summarizes the most important assertion tools for raising an `AssertionError`, and the related modules:

| Assertion tool and application example | Module |
| --- | --- |
| `assert 5 == 5` | (Python keyword) |
| `assertEqual(5.27, 5.27)` | `unittest.TestCase` |
| `assertAlmostEqual(5.24, 5.2, places=1)` | `unittest.TestCase` |
| `assertTrue(5 > 2)` | `unittest.TestCase` |
| `assertFalse(2 < 5)` | `unittest.TestCase` |
| `assertRaises(ZeroDivisionError, lambda x: 1/x, 0.)` | `unittest.TestCase` |
| `assertIn(3, {3, 4})` | `unittest.TestCase` |
| `assert_array_equal(A, B)` | `numpy.testing` |
| `assert_array_almost_equal(A, B, decimal=5)` | `numpy.testing` |
| `assert_allclose(A, B, rtol=1.e-3, atol=1.e-5)` | `numpy.testing` |

*Table 15.1: Assertion tools in Python, unittest, and NumPy*

### 15.2.6 Float comparisons

Two floating-point numbers should not be compared with the `==` comparison, because the result of a computation is often slightly off due to rounding errors. There are numerous tools to test the equality of floats for testing purposes.

First, `allclose` checks that two arrays are almost equal. It can be used in a test function:

```python
self.assertTrue(allclose(computed, expected))
```

Here `self` refers to a `unittest.TestCase` instance. There are also testing tools in the `numpy.testing` package, imported with:

```python
import numpy.testing
```

Testing that two scalars or two arrays are equal is done with `numpy.testing.assert_array_almost_equal` or `numpy.testing.assert_allclose`. These methods differ in the way they describe the required accuracy (see Table 15.1).

As an example, the QR factorization decomposes a given matrix into a product of an orthogonal matrix and an upper triangular matrix:

```python
import scipy.linalg as sl
A = rand(10, 10)
[Q, R] = sl.qr(A)
```

Is the method applied correctly? We can check that `Q` is indeed an orthogonal matrix, and perform a sanity test that `Q @ R == A`:

```python
import numpy.testing as npt
npt.assert_allclose(
    Q.T @ Q, identity(Q.shape[0]), atol=1.e-12)
npt.assert_allclose(Q @ R, A)
```

All this can be collected into a `unittest` test case:

```python
import unittest
import numpy.testing as npt
from scipy.linalg import qr
from scipy import *

class TestQR(unittest.TestCase):
    def setUp(self):
        self.A = rand(10, 10)
        [self.Q, self.R] = qr(self.A)

    def test_orthogonal(self):
        npt.assert_allclose(
            self.Q.T @ self.Q, identity(self.Q.shape[0]),
            atol=1.e-12)

    def test_sanity(self):
        npt.assert_allclose(self.Q @ self.R, self.A)

if __name__ == '__main__':
    unittest.main()
```

Note that in `assert_allclose` the parameter `atol` defaults to zero, which often causes problems when working with matrices having small elements.

### 15.2.7 Unit and functional tests

Up to now, we have only used *functional tests*. A functional test checks whether the functionality is correct; for the bisection algorithm, that the algorithm actually finds a zero when there is one. It is still possible to make a *unit test* for the bisection algorithm, which demonstrates how unit testing often leads to more compartmentalized implementation.

In the bisection method, we would like to check, for instance, that at each step the interval is chosen correctly. This is impossible with the current implementation because the algorithm is hidden inside the function. One remedy is to run only one step of the algorithm. Since all the steps are similar, we might argue that we have tested all the possible steps. We also need to inspect the current bounds `a` and `b`, so we add the number of steps as a parameter and change the return interface:

```python
def bisect(f, a, b, n=100):
    ...
    for iteration in range(n):
        ...
    return a, b
```

We may now add a unit test:

```python
def test_midpoint(self):
    a, b = bisect(identity, -2., 1., 1)
    self.assertAlmostEqual(a, -0.5)
    self.assertAlmostEqual(b, 1.)
```

### 15.2.8 Debugging

Debugging is sometimes necessary while testing, in particular if it is not immediately clear why a given test does not pass. This is made difficult by the design of `unittest.TestCase`, which prevents easy instantiation of test case objects. The solution is to create a special instance for debugging purposes only. Suppose that, in the class `TestIdentity`, we want to test the method `test_functionality`:

```python
test_case = TestIdentity(methodName='test_functionality')
test_case.debug()
```

This runs the individual test and allows for debugging.

### 15.2.9 Test discovery

If you write a Python package, various tests might be spread throughout the package. The module `discover` finds, imports, and runs these test cases. The basic call from the command line is:

```
python -m unittest discover
```

It starts looking for test cases in the current directory and recurses the directory tree downward to find Python objects with the `'test'` string in their name. The command takes optional arguments; the most important are `-s` to modify the start directory and `-p` to define the pattern to recognize the tests:

```
python -m unittest discover -s '.' -p 'Test*.py'
```

## 15.3 Measuring execution time

To make decisions on code optimization, you often have to compare several code alternatives and decide which should be preferred based on execution time. We present three simple ways to measure execution time.

### 15.3.1 Timing with a magic function

The easiest way to measure the execution time of a single statement is to use IPython's magic function `%timeit`. (The IPython shell adds extra functions, called *magic functions*, to standard Python.)

As the execution time of a single statement can be extremely short, the statement is placed in a loop and executed several times. By taking the minimum measured time, you make sure that other tasks running on the computer do not influence the result too much.

Consider four alternative ways to extract nonzero elements from an array:

```python
A = zeros((1000, 1000))
A[53, 67] = 10

def find_elements_1(A):
    b = []
    n, m = A.shape
    for i in range(n):
        for j in range(m):
            if abs(A[i, j]) > 1.e-10:
                b.append(A[i, j])
    return b

def find_elements_2(A):
    return [a for a in A.reshape((-1,)) if abs(a) > 1.e-10]

def find_elements_3(A):
    return [a for a in A.flatten() if abs(a) > 1.e-10]

def find_elements_4(A):
    return A[where(0.0 != A)]
```

Measuring with `%timeit` gives:

```
In [50]: %timeit -n 50 -r 3 find_elements_1(A)
50 loops, best of 3: 585 ms per loop
In [51]: %timeit -n 50 -r 3 find_elements_2(A)
50 loops, best of 3: 514 ms per loop
In [52]: %timeit -n 50 -r 3 find_elements_3(A)
50 loops, best of 3: 519 ms per loop
In [53]: %timeit -n 50 -r 3 find_elements_4(A)
50 loops, best of 3: 7.29 ms per loop
```

The parameter `-n` controls how often the statement is executed before time is measured, and the `-r` parameter controls the number of repetitions.

### 15.3.2 Timing with the Python module timeit

Python provides the module `timeit`, which can be used to measure execution time. It requires that a timer object first be constructed from two strings: a string with setup commands and a string with the commands to be executed. The array and function definitions are written in a string called `setup_statements`, and four timing objects are constructed:

```python
import timeit
setup_statements = """
from scipy import zeros
from numpy import where
A = zeros((1000, 1000))
A[57, 63] = 10.

def find_elements_1(A):
    b = []
    n, m = A.shape
    for i in range(n):
        for j in range(m):
            if abs(A[i, j]) > 1.e-10:
                b.append(A[i, j])
    return b

def find_elements_2(A):
    return [a for a in A.reshape((-1,)) if abs(a) > 1.e-10]

def find_elements_3(A):
    return [a for a in A.flatten() if abs(a) > 1.e-10]

def find_elements_4(A):
    return A[where(0.0 != A)]
"""

experiment_1 = timeit.Timer(stmt='find_elements_1(A)',
                            setup=setup_statements)
experiment_2 = timeit.Timer(stmt='find_elements_2(A)',
                            setup=setup_statements)
experiment_3 = timeit.Timer(stmt='find_elements_3(A)',
                            setup=setup_statements)
experiment_4 = timeit.Timer(stmt='find_elements_4(A)',
                            setup=setup_statements)
```

The timer objects have a method `repeat`. It takes the two parameters `repeat` and `number`. It executes the statement in a loop, measures the time, and repeats this experiment `repeat` times:

```python
t1 = experiment_1.repeat(3, 5)
t2 = experiment_2.repeat(3, 5)
t3 = experiment_3.repeat(3, 5)
t4 = experiment_4.repeat(3, 5)

# Results per loop in ms
min(t1) * 1000 / 5   # 615 ms
min(t2) * 1000 / 5   # 543 ms
min(t3) * 1000 / 5   # 546 ms
min(t4) * 1000 / 5   # 7.26 ms
```

In contrast to `%timeit`, here we obtain lists of all the measurements. As the computing time may vary depending on the overall load of the computer, the minimal value in such a list can be considered a good approximation to the computation time needed to execute the statement.

### 15.3.3 Timing with a context manager

The third method serves to show another application of a context manager. We first construct a context manager object for measuring the elapsed time:

```python
import time

class Timer:
    def __enter__(self):
        self.start = time.time()
        # return self

    def __exit__(self, ty, val, tb):
        end = time.time()
        self.elapsed = end - self.start
        print(f'Time elapsed {self.elapsed} seconds')
        return False
```

The `__enter__` and `__exit__` methods make this class a context manager. The `__exit__` method's parameters `ty`, `val`, and `tb` are `None` in the normal case. If an exception is raised during execution, they take the exception type, its value, and traceback information. The return value `False` indicates that the exception has not been caught so far. We use it to measure the execution time of one of the alternatives:

```python
with Timer():
    find_elements_1(A)
```

This displays a message like `Time elapsed 15.0129795074 ms`. If the timing result should be accessible in a variable, the `__enter__` method must return the `Timer` instance (uncomment the `return` statement) and a `with ... as ...` construction has to be used:

```python
with Timer() as t1:
    find_elements_1(A)
t1.elapsed   # contains the result
```

## 15.4 Summary

No program development without testing! We showed the importance of well-organized and documented tests. Some professionals even start development by first specifying tests. A useful tool for automatic testing is the module `unittest`, which we explained in detail. While testing improves the reliability of code, profiling is needed to improve performance. Alternative ways to code may result in large performance differences. We showed how to measure computation time and how to localize bottlenecks in your code.
