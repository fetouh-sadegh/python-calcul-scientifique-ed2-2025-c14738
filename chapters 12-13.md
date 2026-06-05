# Chapter 12

This chapter covers errors and exceptions and how to find and fix them. Handling exceptions is an important part of writing reliable and usable code. We introduce the basic built-in exceptions, show how to raise and treat them, and present debugging with Python's built-in debugger.

Topics covered:

- What are exceptions?
- Finding errors: debugging

## 12.1 What are exceptions?

The first error programmers meet is a syntax error, meaning the code instructions are not correctly formatted. For example, a missing colon at the end of a `for` declaration:

```python
>>> for i in range(10)
  File "<stdin>", line 1
    for i in range(10)
                      ^
SyntaxError: invalid syntax
```

This is an example of an exception being raised. Exceptions in Python are derived (inherited) from a base class called `Exception`. Python comes with a number of built-in exceptions.

Two common examples: `ZeroDivisionError` is raised when you try to divide by zero, and the traceback prints the file name, line, and function name where the error occurred:

```python
def f(x):
    return 1 / x

>>> f(2.5)
0.4
>>> f(0)
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
  File "exception_tests.py", line 3, in f
    return 1 / x
ZeroDivisionError: integer division or modulo by zero
```

### Table 12.1: Some frequently used built-in exceptions

| Exception | Description |
| --- | --- |
| `IndexError` | Index is out of bounds, for example, `v[10]` when `v` only has five elements. |
| `KeyError` | A reference to an undefined dictionary key. |
| `NameError` | A name not found, for example, an undefined variable. |
| `LinAlgError` | Errors in the `linalg` module, for example, solving a system with a singular matrix. |
| `ValueError` | Incompatible data value, for example, using `dot` with incompatible arrays. |
| `IOError` | I/O operation fails, for example, file not found. |
| `ImportError` | A module or name is not found on import. |

Arrays can only contain elements of the same datatype. Assigning a value of an incompatible type raises `ValueError`:

```python
>>> a = arange(8.0)
>>> a
array([ 0., 1., 2., 3., 4., 5., 6., 7.])
>>> a[3] = 'string'
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
ValueError: could not convert string to float: string
```

### 12.1.1 Basic principles

We use exceptions by raising them with `raise` and catching them with `try` statements.

**Raising exceptions**

Creating an error is referred to as raising an exception. You can raise an exception of an unspecified type:

```python
raise Exception("Something went wrong")
```

Printing an error message is not recommended: printouts are easy to miss, and they render your code unusable by other code, which has no way of knowing an error occurred. It is always better to raise an exception with a descriptive message:

```python
raise Exception("The algorithm did not converge.")
```

A typical example checks the input inside a function before continuing. Note the use of the exception name (`TypeError`, `ValueError`) followed by the message; by specifying the type, the calling code can handle errors differently depending on the type raised:

```python
def factorial(n):
    if not (isinstance(n, (int, int32, int64))):
        raise TypeError("An integer is expected")
    if not (n >= 0):
        raise ValueError("A positive number is expected")
```

**Catching exceptions**

Dealing with an exception is referred to as catching an exception. Checking for exceptions is done with `try` and `except`. An exception stops the program execution flow and looks for the closest enclosing `try` block. If not caught, it keeps searching higher up the calling stack; if no block is found, execution stops and the standard traceback is displayed.

```python
n = -3
try:
    print(factorial(n))
except ValueError:
    print(factorial(-n))  # Here we catch the error
```

If the code inside the `try` block raises a `ValueError`, the exception is caught and the action in the `except` block is taken. If no exception occurs, the `except` block is skipped entirely.

The `except` statement can catch multiple exceptions grouped in a tuple:

```python
except (RuntimeError, ValueError, IOError):
```

A `try` block can also have multiple `except` statements, so exceptions can be handled differently depending on their type:

```python
try:
    f = open('data.txt', 'r')
    data = f.readline()
    value = float(data)
except FileNotFoundError as FnF:
    print(f'{FnF.strerror}: {FnF.filename}')
except ValueError:
    print("Could not convert data to float.")
```

Here `FileNotFoundError` is assigned to a variable `FnF` with the keyword `as`, giving access to more details such as `FnF.strerror` and `FnF.filename`. Each error type can have its own set of attributes.

The `try`-`except` combination can be extended with optional `else` and `finally` blocks. Combining `try` with `finally` is useful when cleanup work needs to happen at the end, for example making sure a file is closed:

```python
try:
    f = open('data.txt', 'r')
    # some function that does something with the file
    process_file_data(f)
except:
    ...
finally:
    f.close()
```

This ensures the file is closed no matter what exceptions are thrown. Exceptions not handled inside the `try` statement are saved and raised after the `finally` block.

### 12.1.2 User-defined exceptions

Besides the built-in exceptions, you can define your own. User-defined exceptions should inherit from the base class `Exception`. Here is a simple example:

```python
class MyError(Exception):
    def __init__(self, expr):
        self.expr = expr
    def __str__(self):
        return str(self.expr)

try:
    x = random.rand()
    if x < 0.5:
        raise MyError(x)
except MyError as e:
    print("Random number too small", e.expr)
else:
    print(x)
```

A random number is generated. If it is below 0.5, an exception is raised and a message is printed. If no exception is raised, the number is printed. The block under `else` is executed only if no exception occurs. It is recommended that you name your exceptions with names that end in `Error`, like the standard built-in exceptions.

### 12.1.3 Context managers — the with statement

The `with` statement encapsulates the `try ... finally` structure in one simple command, which is useful when working with contexts such as files or databases:

```python
with open('data.txt', 'w') as f:
    process_file_data(f)
```

This tries to open the file, run the operations, and close the file. If anything goes wrong, the file is closed properly and then the exception is raised. It is equivalent to:

```python
f = open('data.txt', 'w')
try:
    # some function that does something with the file
    process_file_data(f)
except:
    ...
finally:
    f.close()
```

Context managers are Python objects with two special methods, `__enter__` and `__exit__`. Any object of a class implementing these two methods can be used as a context manager. The `__enter__` method implements the initialization (for example, opening a file); if it has a return statement, the returned object is accessed using the construct `as`. The `__exit__` method contains the cleanup instructions (for example, closing a file).

Some NumPy functions can be used as context managers. For example, `load` supports the context manager for some file formats, and `errstate` can specify floating-point error handling behavior within a block:

```python
import numpy as np  # note, sqrt in NumPy and SciPy
# behave differently in that example
with errstate(invalid='ignore'):
    print(np.sqrt(-1))  # prints 'nan'
with errstate(invalid='warn'):
    print(np.sqrt(-1))  # prints 'nan' and
    # 'RuntimeWarning: invalid value encountered in sqrt'
with errstate(invalid='raise'):
    print(np.sqrt(-1))  # prints nothing and raises FloatingPointError
```

## 12.2 Finding errors: debugging

Errors in software code are sometimes referred to as bugs. Debugging is the process of finding and fixing bugs. The most efficient way is to use a tool called a debugger. Having unit tests in place is a good way to identify errors early.

### 12.2.1 Bugs

There are typically two kinds of bugs:

- An exception is raised and not caught.
- The code does not function properly.

The first case is usually easier to fix. The second can be more difficult, as the problem can be a faulty idea or solution, a faulty implementation, or a combination of the two.

### 12.2.2 The stack

When an exception is raised, you see the call stack. The call stack contains the trace of all the functions that called the code where the exception was raised. A simple stack example:

```python
def f():
    g()
def g():
    h()
def h():
    1 // 0

f()
```

The stack, in this case, is `f`, `g`, and `h`. The output looks like this:

```python
Traceback (most recent call last):
  File "stack_example.py", line 11, in <module>
    f()
  File "stack_example.py", line 3, in f
    g()
  File "stack_example.py", line 6, in g
    h()
  File "stack_example.py", line 9, in h
    1 // 0
ZeroDivisionError: integer division or modulo by zero
```

A stack trace reports on the active stack at a certain point in the execution. This is sometimes called post-mortem analysis. You can also invoke a stack trace manually by raising an exception to analyze a piece of code where you suspect there is an error:

```python
def f(a):
    g(a)
def g(a):
    h(a)
def h(a):
    raise Exception(f'An exception just to provoke a strack trace and a value a={a}')

f(23)
```

This returns a traceback ending with:

```python
Exception: An exception just to provoke a strack trace and a value a=23
```

### 12.2.3 The Python debugger

Python comes with its own built-in debugger called `pdb`. The easiest way to use it is to enable stack tracing at the point in your code that you want to investigate:

```python
import pdb

def complex_to_polar(z):
    pdb.set_trace()
    r = sqrt(z.real ** 2 + z.imag ** 2)
    phi = arctan2(z.imag, z.real)
    return (r, phi)

z = 3 + 5j
r, phi = complex_to_polar(z)
print(r, phi)
```

The command `pdb.set_trace()` starts the debugger and enables tracing of subsequent commands. The debugger prompt is indicated with `(Pdb)`. It stops execution and lets you inspect variables, modify variables, and step through commands:

```python
> debugging_example.py(7)complex_to_polar()
-> r = sqrt(z.real ** 2 + z.imag ** 2)
(Pdb)
```

Stepping through commands is done with the command `n` (next):

```python
(Pdb) n
> debugging_example.py(8)complex_to_polar()
-> phi = arctan2(z.imag, z.real)
(Pdb) n
> debugging_example.py(9)complex_to_polar()
-> return (r, phi)
```

The command `l` (list) shows the current line with the surrounding code. The inspection of variables is done with the command `p` (print) followed by the variable name; the command `c` (continue) continues execution:

```python
(Pdb) p z
(3+5j)
(Pdb) n
> debugging_example.py(8)complex_to_polar()
-> phi = arctan2(z.imag, z.real)
(Pdb) p r
5.830951894845300
(Pdb) c
(5.830951894845300, 1.0303768265243125)
```

Changing a variable mid-execution is also possible. Simply assign the new value at the debugger prompt and step or continue:

```python
(Pdb) z = 2j
(Pdb) z
2j
(Pdb) c
(2.0, 1.5707963267948966)
```

### 12.2.4 Overview — debug commands

Note that any Python command also works (for example, assigning values to variables). If you want to inspect a variable with a name that coincides with one of the debugger's short commands, for example `h`, you must use `!h` to display the variable.

### Table 12.2: The most common debug commands

| Command | Action |
| --- | --- |
| `h` | Help (without arguments, it prints available commands) |
| `l` | Lists the code around the current line |
| `q` | Quit (exits the debugger and the execution stops) |
| `c` | Continues execution |
| `r` | Continues execution until the current function returns |
| `n` | Continues execution until the next line |
| `p <expression>` | Evaluates and prints the expression in the current context |

### 12.2.5 Debugging in IPython

IPython comes with a version of the debugger called `ipdb`. There is a command in IPython, `%pdb`, that automatically turns on the debugger in case of an exception. This is useful when experimenting with new ideas or code:

```python
In [1]: %pdb  # this is a so-called IPython magic command
Automatic pdb calling has been turned ON
In [2]: a = 10
In [3]: b = 0
In [4]: c = a / b
```

When the `ZeroDivisionError` is raised, the debugger prompt shows `ipdb>` instead, to indicate that the debugger is running.

## 12.3 Summary

The key concepts in this chapter were exceptions and errors. We showed how an exception is raised to be caught later in another program unit. You can define your own exceptions and equip them with messages and current values of given variables. Code may also return unexpected results without throwing an exception; the technique to localize the source of the erroneous result is called debugging. We introduced debugging methods using the stack, the Python debugger `pdb`, and debugging in IPython.

# Chapter 13

This chapter covers Python modules. Modules are files containing functions and class definitions. The concept of a namespace and the scope of variables across functions and modules are also explained.

Topics covered:

- Namespaces
- The scope of a variable
- Modules

## 13.1 Namespaces

Names of Python objects, such as the names of variables, classes, functions, and modules, are collected in namespaces. Modules and classes have their own named namespaces with the same name as these objects. These namespaces are created when a module is imported or a class is instantiated. The lifetime of a module's namespace is as long as the current Python session; that of a class instance lasts until the instance is deleted.

Functions create a local namespace when they are executed (invoked). It is deleted when the function stops execution with a regular return or an exception. Local namespaces are unnamed.

The concept of namespaces puts a variable name in its context. For example, there are several functions with the name `sin`, distinguished by the namespace they belong to:

```python
import math
import numpy
math.sin
numpy.sin
```

They are different: `numpy.sin` is a universal function accepting lists or arrays as input, while `math.sin` takes only floats.

A list with all the names in a particular namespace can be obtained with `dir(<name of the namespace>)`. It contains two special names, `__name__` and `__doc__`. The former refers to the name of the module and the latter to its docstring:

```python
math.__name__  # returns math
math.__doc__   # returns 'This module provides access to ....'
```

There is a special namespace, `__builtin__`, which contains names available in Python without any import. Its name need not be given when referring to a built-in object:

```python
'float' in dir(__builtin__)  # returns True
float is __builtin__.float   # returns True
```

## 13.2 The scope of a variable

A variable defined in one part of a program does not need to be known in other parts. All program units to which a certain variable is known are called the scope of that variable. Consider two nested functions:

```python
e = 3
def my_function(in1):
    a = 2 * e
    b = 3
    in1 = 5
    def other_function():
        c = a
        d = e
        return dir()
    print(f"""
my_function's namespace: {dir()}
other_function's namespace: {other_function()}
""")
    return a
```

The execution of `my_function(3)` results in:

```python
my_function's namespace: ['a', 'b', 'in1', 'other_function']
other_function's namespace: ['a', 'c', 'd']
```

The variable `e` is in the namespace of the program unit that encloses `my_function`. For the two functions, `e` is a global variable: it is not in the local namespace and not listed by `dir()`, but its value is available.

It is good practice to pass information to a function only by its parameter list. By assigning it a value, a variable automatically becomes a local variable:

```python
e = 3
def my_function():
    e = 4
    a = 2
    print(f"my_function's namespace: {dir()}")
```

This can be seen when executing:

```python
e = 3
my_function()
e  # has the value 3
```

The output gives the local variables of `my_function`:

```python
my_function's namespace: ['a', 'e']
```

Now `e` became a local variable; this code has two variables `e` belonging to different namespaces.

By using the `global` declaration statement, a variable defined in a function can be made global, that is, accessible even outside the function:

```python
def fun():
    def fun1():
        global a
        a = 3
    def fun2():
        global b
        b = 2
        print(a)
    fun1()
    fun2()  # prints a
    print(b)
```

It is advisable to avoid this construct and the use of `global`. Code using `global` is hard to debug and maintain. The use of classes makes the use of `global` mainly obsolete.

## 13.3 Modules

In Python, a module is simply a file containing classes and functions. By importing the file in your session or script, the functions and classes become usable.

### 13.3.1 Introduction

Python comes with many libraries by default, and you may install more for specific purposes. NumPy, SciPy, and Matplotlib are important examples. To use a library, you may either load only certain objects from a library:

```python
from numpy import array, vander
```

load the entire library:

```python
from numpy import *
```

or give access to an entire library by creating a namespace with the library name:

```python
import numpy
...
numpy.array(...)
```

Prefixing a function with the namespace distinguishes it from other objects with the same name. The name of a namespace can be specified together with the `import` command:

```python
import numpy as np
...
np.array(...)
```

Which alternative you use affects readability and the possibilities for mistakes. A common mistake is shadowing:

```python
from scipy.linalg import eig
A = array([[1, 2], [3, 4]])
(eig, eigvec) = eig(A)
...
(c, d) = eig(B)  # raises an error
```

A way to avoid this is to use `import` instead of `from` and access the command by referring to the namespace, here `sl`:

```python
import scipy.linalg as sl
A = array([[1, 2], [3, 4]])
(eig, eigvec) = sl.eig(A)  # eig and sl.eig are different objects
...
(c, d) = sl.eig(B)
```

Importing objects with `from scipy import *` does not make the module from which they are imported evident.

### Table 13.1: Examples of modules and corresponding imported functions

| Libraries | Methods |
| --- | --- |
| `numpy` | `array`, `arange`, `linspace`, `vstack`, `hstack`, `dot`, `eye`, `identity`, and `zeros`. |
| `scipy.linalg` | `solve`, `lstsq`, `eig`, and `det`. |
| `matplotlib.pyplot` | `plot`, `legend`, and `cla`. |
| `scipy.integrate` | `quad`. |
| `copy` | `copy` and `deepcopy`. |

### 13.3.2 Modules in IPython

A typical scenario is working on a file with function or class definitions that you change within a development cycle. To load the contents into the shell, you may use `import`, but the file is loaded only once; changing the file has no effect on later imports. That is where IPython's magic command `run` enters the stage.

**The IPython magic command — run**

IPython has a special magic command named `run` that executes a file as if you were running it directly in Python, independently of what is already defined in IPython. This is the recommended method to test a script intended as a standalone program. You must import everything you need in the executed file, the same as if executing it from the command line:

```python
from numpy import array
...
a = array(...)
```

This script file is executed in Python by `exec(open('myfile.py').read())`. Alternatively, in IPython the magic command `run myfile` can be used to make sure the script runs independently of previous imports. Everything defined in the file is imported into the IPython workspace.

### 13.3.3 The variable __name__

In any module, the special variable `__name__` is defined as the name of the current module. In the command line (in IPython), this variable is set to `__main__`. This allows the following trick:

```python
# module
import ...
class ...

if __name__ == "__main__":
    # perform some tests here
```

The tests are run only when the file is directly run, not when it is imported. When imported, `__name__` takes the name of the module instead of `__main__`.

### 13.3.4 Some useful modules

The list of useful Python modules is vast. The following is a short segment focused on modules related to mathematical and engineering applications.

### Table 13.2: A non-exhaustive list of useful Python packages for engineering applications

| Module | Description |
| --- | --- |
| `scipy` | Functions used in scientific computing |
| `numpy` | Support arrays and related methods |
| `matplotlib` | Plotting and visualization |
| `functools` | Partial application of functions |
| `itertools` | Iterator tools to provide special capabilities, such as slicing to generators |
| `re` | Regular expressions for advanced string handling |
| `sys` | System-specific functions |
| `os` | Operating system interfaces such as directory listing and file handling |
| `datetime` | Representing dates and date increments |
| `time` | Returning wall clock time |
| `timeit` | Measuring execution time |
| `sympy` | Computer arithmetic package (symbolic computations) |
| `pickle` | Pickling, a special file input and output format |
| `shelves` | Shelves, a special file input and output format |
| `contextlib` | Tools for context managers |

The book advises against the use of the mathematics module `math` and favors `numpy` instead, because many of NumPy's functions, such as `sin`, operate on arrays, while the corresponding functions in `math` don't.

## 13.4 Summary

We introduced namespaces and discussed the difference between `import` and `from ... import *`. The scope of a variable was made more complete, and now you fully understand what importing means and the importance of that concept.
