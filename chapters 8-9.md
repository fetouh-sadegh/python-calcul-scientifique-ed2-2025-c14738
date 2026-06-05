# chapters 8-9

# Chapter 8

Classes are the core of object-oriented programming. They group together objects that share the same methods, just like mathematical objects (functions, rational numbers, intervals, sequences) share operations. This chapter shows how the object-oriented concepts of classes, instances, inheritance, methods, attributes, and operator overloading are realized in Python.

Topics covered:

- Introduction to classes
- Bound and unbound methods
- Class attributes and class methods
- Subclasses and inheritance
- Encapsulation
- Classes as decorators

## 8.1 Introduction to classes

This section introduces the most common terms of classes and their realization in Python, using a guiding example.

### 8.1.1 A guiding example: Rational numbers

We illustrate the concept of classes with rational numbers, that is, numbers of the form `p/q`, where `p` and `q` are integers.

We use rational numbers here only as an example of the class concept. For real work with rational numbers in Python, use the standard `fractions` module.

### 8.1.2 Defining a class and making an instance

A class is defined by a block command with the keyword `class`, the name of the class, and statements in the block:

```python
class RationalNumber:
    pass
```

An instance (an object of type `RationalNumber`) is created by using the class name like a function. `type(r)` returns `<class '__main__.RationalNumber'>`:

```python
r = RationalNumber()
if isinstance(r, RationalNumber):
    print('Indeed, it belongs to the class RationalNumber')
```

So far this object has no data and no methods.

### 8.1.3 The __init__ method

The special method `__init__` is used to initialize the instance with defining data. Here the data are the numerator and denominator:

```python
class RationalNumber:
    def __init__(self, numerator, denominator):
        self.numerator = numerator
        self.denominator = denominator
```

The instantiation works as follows:

```python
q = RationalNumber(10, 20)   # Defines a new object
q.numerator                  # returns 10
q.denominator                # returns 20
```

Creating `RationalNumber(10, 20)` does two things:

1. It first creates an empty object `q`.
2. It then applies the `__init__` function, that is, `q.__init__(10, 20)` is executed.

The first parameter of `__init__` refers to the new object itself. By convention it is named `self`.

### 8.1.4 Attributes and methods

Objects bound to an instance are called **attributes**. They are referenced with the dot syntax `<object>.attribute`:

```python
q = RationalNumber(3, 5)   # instantiation
q.numerator                # attribute access
q.denominator
```

Once an instance is defined, attributes may be set, changed, or deleted, using the same syntax as for regular variables:

```python
q = RationalNumber(3, 5)
r = RationalNumber(7, 3)
q.numerator = 17
del r.denominator
```

Changing or deleting an attribute may have undesired side effects (see Section 8.2). As functions are objects too, functions used as attributes are called **methods** of the instance, called as `<object>.method(<arguments...>)`. For example, a method that converts a rational number to a float:

```python
class RationalNumber:
    ...
    def convert2float(self):
        return float(self.numerator) / float(self.denominator)
```

It is used with a regular function call:

```python
q = RationalNumber(10, 20)   # Defines a new object
q.convert2float()            # returns 0.5
```

This is equivalent to `RationalNumber.convert2float(q)`. The object instance is inserted as the first argument. Calling `q.convert2float(15)` would therefore raise:

```
TypeError: convert2float() takes exactly 1 argument (2 given)
```

because it is equivalent to `RationalNumber.convert2float(q, 15)`.

### 8.1.5 Special methods

The special method `__repr__` defines how the object is represented in the Python interpreter:

```python
class RationalNumber:
    ...
    def __repr__(self):
        return f'{self.numerator} / {self.denominator}'
```

With this method, just typing `q` returns `10 / 20`.

To add two rational numbers, a first (non-special) attempt could be:

```python
class RationalNumber:
    ...
    def add(self, other):
        p1, q1 = self.numerator, self.denominator
        if isinstance(other, int):
            p2, q2 = other, 1
        else:
            p2, q2 = other.numerator, other.denominator
        return RationalNumber(p1 * q2 + p2 * q1, q1 * q2)
```

It would be nicer to write `q + p`. This is done with the special method `__add__`. Just renaming `add` to `__add__` allows using the plus sign:

```python
q = RationalNumber(1, 2)
p = RationalNumber(1, 3)
q + p   # RationalNumber(5, 6)
```

The expression `q + p` is an alias for `q.__add__(p)`. Table 8.1 lists special methods for binary operators:

| Operator | Method | Operator | Method |
| --- | --- | --- | --- |
| `+` | `__add__` | `+=` | `__iadd__` |
| `*` | `__mul__` | `*=` | `__imul__` |
| `-` | `__sub__` | `-=` | `__isub__` |
| `/` | `__truediv__` | `/=` | `__itruediv__` |
| `//` | `__floordiv__` | `//=` | `__ifloordiv__` |
| `**` | `__pow__` | | |
| `==` | `__eq__` | `!=` | `__ne__` |
| `<=` | `__le__` | `<` | `__lt__` |
| `>=` | `__ge__` | `>` | `__gt__` |
| `()` | `__call__` | `[]` | `__getitem__` |

The implementation of these operators for a new class is called **operator overloading**. Another example checks whether two rational numbers are the same:

```python
class RationalNumber:
    ...
    def __eq__(self, other):
        return self.denominator * other.numerator == \
               self.numerator * other.denominator
```

Used like this:

```python
p = RationalNumber(1, 2)
q = RationalNumber(2, 4)
p == q   # True
```

**Reverse operations.** By default `+` invokes the left operand's `__add__`. In `5 + p`, the operands are commuted and `int.__add__` is invoked, which cannot handle rational numbers. If the left operand's method raises an exception, the reverse method (here `__radd__`) of the right operand is called. If it does not exist, a `TypeError` is raised. We equip the class with `__radd__`:

```python
class RationalNumber:
    ...
    def __radd__(self, other):
        return self + other
```

Note that `__radd__` interchanges the order of the arguments; `self` is the `RationalNumber` and `other` is the object to be converted.

**Methods mimicking function calls and iterables.** Using a class instance with parentheses or brackets, `()` or `[]`, invokes `__call__` or `__getitem__`, giving the instance the behavior of a function or of an iterable:

```python
class Polynomial:
    ...
    def __call__(self, x):
        return self.eval(x)
```

```python
p = Polynomial(...)   # Creating a polynomial object
p(3.)                 # value of p at 3.
```

The special method `__getitem__` makes sense when the class provides an iterator. A three-term recursion, important for orthogonal polynomials, can be set up as a class:

```python
import itertools

class Recursion3Term:
    def __init__(self, a0, a1, u0, u1):
        self.coeff = [a1, a0]
        self.initial = [u1, u0]

    def __iter__(self):
        u1, u0 = self.initial
        yield u0   # (see also Iterators, Chapter 9)
        yield u1
        a1, a0 = self.coeff
        while True:
            u1, u0 = a1 * u1 + a0 * u0, u1
            yield u1

    def __getitem__(self, k):
        return list(itertools.islice(self, k, k + 1))[0]
```

The `__iter__` method defines a generator object, allowing an instance to be used as an iterator, while `__getitem__` enables direct access as if it were a list:

```python
r3 = Recursion3Term(-0.35, 1.2, 1, 1)
for i, r in enumerate(r3):
    if i == 7:
        print(r)   # returns 0.194167
        break

r3[7]   # returns 0.194167
```

## 8.2 Attributes that depend on each other

If an attribute is changed, dependent attributes are not updated automatically. Consider a triangle defined from three points:

```python
class Triangle:
    def __init__(self, A, B, C):
        self.A = array(A)
        self.B = array(B)
        self.C = array(C)
        self.a = self.C - self.B
        self.b = self.C - self.A
        self.c = self.B - self.A

    def area(self):
        return abs(cross(self.b, self.c)) / 2
```

```python
tr = Triangle([0., 0.], [1., 0.], [0., 1.])
tr.area()   # returns 0.5
```

If we change point `B`, the edges `a` and `c` are not updated and the area is wrong:

```python
tr.B = [12., 0.]
tr.area()   # still returns 0.5, should be 6 instead
```

A remedy is a **setter** method (executed when an attribute is changed) and a **getter** method (executed when a value is requested).

### 8.2.1 The function property

The special function `property` links an attribute to a getter, setter, and deleter method, and an optional docstring:

```python
attribute = property(fget=get_attr, fset=set_attr,
                     fdel=del_attr, doc=string)
```

Modified `Triangle` class so that `tr.B = <something>` invokes the setter `set_B`:

```python
class Triangle:
    def __init__(self, A, B, C):
        self._A = array(A)
        self._B = array(B)
        self._C = array(C)
        self._a = self._C - self._B
        self._b = self._C - self._A
        self._c = self._B - self._A

    def area(self):
        return abs(cross(self._c, self._b)) / 2.

    def set_B(self, B):
        self._B = B
        self._a = self._C - self._B
        self._c = self._B - self._A

    def get_B(self):
        return self._B

    def del_Pt(self):
        raise Exception('A triangle point cannot be deleted')

    B = property(fget=get_B, fset=set_B, fdel=del_Pt)
```

Now changing `B` updates the dependent attributes, and deletion is prevented:

```python
tr.B = [12., 0.]
tr.area()   # returns 6.0
del tr.B    # raises an exception
```

The underscore prefix is a convention to indicate attributes not designed to be accessed directly. They are not private as in other languages; they are just not intended for direct access.

## 8.3 Bound and unbound methods

The nature of a method changes after an instance is created:

```python
class A:
    def func(self, arg):
        pass
```

```python
A.func          # <unbound method A.func>
instA = A()     # we create an instance
instA.func      # <bound method A.func of ...>
```

Calling `A.func(3)` raises `TypeError: func() missing 1 required positional argument: 'arg'`, while `instA.func(3)` works as expected. On creation of an instance, the method is **bound** to the instance, and `self` gets the instance assigned as its value. Class methods (Section 8.4.2) are different: they are always bound.

## 8.4 Class attributes and class methods

Class attributes and class methods allow access to methods and data before an instance is created.

### 8.4.1 Class attributes

Attributes specified in the class declaration are **class attributes**:

```python
class Newton:
    tol = 1e-8   # this is a class attribute

    def __init__(self, f):
        self.f = f   # this is not a class attribute
        ...
```

Class attributes are useful as default values. Both instances share the value initialized in the class:

```python
N1 = Newton(f)
N2 = Newton(g)
N1.tol   # 1e-8
N2.tol   # 1e-8
```

Altering the class attribute affects all corresponding attributes of all instances:

```python
Newton.tol = 1e-10
N1.tol   # 1e-10
N2.tol   # 1e-10
```

But altering `tol` for one instance detaches it from the class attribute:

```python
N2.tol = 1.e-4
N1.tol   # still 1.e-10
Newton.tol = 1e-5
N1.tol   # 1.e-5
N2.tol   # 1.e-4  (detached, not altered)
```

### 8.4.2 Class methods

Class methods are always bound methods, bound to the class itself. The decorator `@classmethod` precedes the method definition. The first argument refers to the class itself and is, by convention, called `cls`:

```python
class B:
    @classmethod
    def func(cls, *args):
        ...
```

Class methods are useful for executing commands before an instance is created, for instance a preprocessing step. Here a class method prepares data (interpolation points) before creating an instance:

```python
class Polynomial:
    def __init__(self, coeff):
        self.coeff = array(coeff)

    @classmethod
    def by_points(cls, x, y):
        degree = x.shape[0] - 1
        coeff = polyfit(x, y, degree)
        return cls(coeff)

    def __eq__(self, other):
        return allclose(self.coeff, other.coeff)
```

The class method allows transforming interpolation data to coefficients even when no instance is available:

```python
p1 = Polynomial.by_points(array([0., 1.]), array([0., 1.]))
p2 = Polynomial([1., 0.])
print(p1 == p2)   # prints True
```

## 8.5 Subclasses and inheritance

This section introduces abstract classes, subclasses, and inheritance, using one-step methods for solving a differential equation. The generic ordinary initial value problem is given by a right-hand side function `f`, an initial value `x0`, and an interval. A one-step method constructs the solution by recursion steps, with a **step function** characterizing the individual methods (Explicit Euler, Midpoint rule, Runge-Kutta 4).

First, an abstract class with the abstract description of the method:

```python
class OneStepMethod:
    def __init__(self, f, x0, interval, N):
        self.f = f
        self.x0 = x0
        self.interval = [t0, te] = interval
        self.grid = linspace(t0, te, N)
        self.h = (te - t0) / N

    def generate(self):
        ti, ui = self.grid[0], self.x0
        yield ti, ui
        for t in self.grid[1:]:
            ui = ui + self.h * self.step(self.f, ui, ti)
            ti = t
            yield ti, ui

    def solve(self):
        self.solution = array(list(self.generate()))

    def plot(self):
        plot(self.solution[:, 0], self.solution[:, 1])

    def step(self, f, u, t):
        raise NotImplementedError()
```

This abstract class is used as a template for the individual methods:

```python
class ExplicitEuler(OneStepMethod):
    def step(self, f, u, t):
        return f(u, t)

class MidPointRule(OneStepMethod):
    def step(self, f, u, t):
        return f(u + self.h / 2 * f(u, t), t + self.h / 2)
```

`OneStepMethod` is the **parent class**. All its methods and attributes are inherited by the subclasses unless overridden. `step` is redefined in the subclasses, while `generate` is generic and inherited. Usage:

```python
def f(x, t):
    return -0.5 * x

euler = ExplicitEuler(f, 15., [0., 10.], 20)
euler.solve()
euler.plot()

midpoint = MidPointRule(f, 15., [0., 10.], 20)
midpoint.solve()
midpoint.plot()
```

Common parameter lists can be avoided with the star operator:

```python
argument_list = [f, 15., [0., 10.], 20]
euler = ExplicitEuler(*argument_list)
midpoint = MidPointRule(*argument_list)
```

The abstract class is never used to create an instance, because `step` raises `NotImplementedError`. To access a parent's methods or attributes, use `super`. This is useful when the child extends the parent's `__init__`:

```python
class ExplicitEuler(OneStepMethod):
    def __init__(self, *args, **kwargs):
        self.name = 'Explicit Euler Method'
        super(ExplicitEuler, self).__init__(*args, **kwargs)

    def step(self, f, u, t):
        return f(u, t)
```

Using `super` instead of the explicit parent name allows changing the parent class without changing all references.

## 8.6 Encapsulation

Sometimes inheritance is impractical or impossible, which motivates **encapsulation**. We encapsulate Python functions in a new class `Function` and provide it with relevant methods:

```python
class Function:
    def __init__(self, f):
        self.f = f

    def __call__(self, x):
        return self.f(x)

    def __add__(self, g):
        def sum(x):
            return self(x) + g(x)
        return type(self)(sum)

    def __mul__(self, g):
        def prod(x):
            return self.f(x) * g(x)
        return type(self)(prod)

    def __radd__(self, g):
        return self + g

    def __rmul__(self, g):
        return self * g
```

`__add__` and `__mul__` return an instance of the same class via `return type(self)(sum)`, a more general form of `return Function(sum)`. We can construct Chebyshev polynomials as instances:

```python
T5 = Function(lambda x: cos(5 * arccos(x)))
T6 = Function(lambda x: cos(6 * arccos(x)))
```

Their orthogonality can be checked by integration:

```python
import scipy.integrate as sci
weight = Function(lambda x: 1 / sqrt((1 - x ** 2)))
[integral, errorestimate] = \
    sci.quad(weight * T5 * T6, -1, 1)
# (6.510878470473995e-17, 1.3237018925525037e-14)
```

Without encapsulation, multiplying functions as simply as `weight * T5 * T6` would not have been possible.

## 8.7 Classes as decorators

Since a class with a `__call__` method behaves like a function, classes can be used as decorators. As an example, a decorator that prints all input parameters before the function is invoked:

```python
class echo:
    text = 'Input parameters of {name}\n' + \
           'Positional parameters {args}\n' + \
           'Keyword parameters {kwargs}\n'

    def __init__(self, f):
        self.f = f

    def __call__(self, *args, **kwargs):
        print(self.text.format(name=self.f.__name__,
                               args=args, kwargs=kwargs))
        return self.f(*args, **kwargs)
```

```python
@echo
def line(m, b, x):
    return m * x + b

line(2., 5., 3.)
line(2., 5., x=3.)
```

The second call produces:

```
Input parameters of line
Positional parameters (2.0, 5.0)
Keyword parameters {'x': 3.0}
11.0
```

Classes allow more possibilities than function decorators, as they can collect data. Each decorated function creates a new instance of the decorator class, and data collected by one instance can be shared with another through class attributes. The following decorator counts function calls:

```python
class CountCalls:
    """
    Decorator that keeps track of the number of times
    a function is called.
    """
    instances = {}

    def __init__(self, f):
        self.f = f
        self.numcalls = 0
        self.instances[f] = self

    def __call__(self, *args, **kwargs):
        self.numcalls += 1
        return self.f(*args, **kwargs)

    @classmethod
    def counts(cls):
        """
        Return a dict of {function: # of calls} for all
        registered functions.
        """
        return dict([(f.__name__, cls.instances[f].numcalls)
                     for f in cls.instances])
```

The class attribute `CountCalls.instances` stores the counters for each instance:

```python
@CountCalls
def line(m, b, x):
    return m * x + b

@CountCalls
def parabola(a, b, c, x):
    return a * x ** 2 + b * x + c

line(3., -1., 1.)
parabola(4., 5., -1., 2.)
CountCalls.counts()   # returns {'line': 1, 'parabola': 1}
parabola.numcalls     # returns 1
```

## 8.8 Summary

Object-oriented programming is one of the most important programming concepts in modern computer science. In this chapter we defined objects as instances of classes, providing them with methods and attributes. The first parameter of methods, usually denoted `self`, plays a special role. We saw methods that define basic operations such as `+` and `*` for custom classes (operator overloading). Python allows hiding attributes and accessing them through getter and setter methods built with the important function `property`. We also covered class attributes, class methods, inheritance via `super`, encapsulation, and classes used as decorators.

# Chapter 9

This chapter presents iterations using loops and iterators, with examples for lists and generators. Iteration is one of the most fundamental operations a computer performs. Traditionally it is achieved with a `for` loop, but a `for` loop only needs one element of the list at a time. It is therefore desirable to use objects that create elements on demand, one at a time, instead of providing a complete list. This is what **iterators** achieve.

Topics covered:

- The `for` statement
- Controlling the flow inside the loop
- Iterable objects
- List-filling patterns
- When iterators behave as lists
- Iterator objects
- Infinite iterations

## 9.1 The for statement

The primary aim of the `for` statement is to traverse a list:

```python
for s in ['a', 'b', 'c']:
    print(s)   # a b c
```

The loop variable is available after the loop has terminated. To repeat a task a defined number of times, use `range`:

```python
for iteration in range(n):   # repeat the following code n times
    ...
```

Rather than indexing into a list, iterate directly over it for clarity:

```python
for element in my_list:
    ...
```

If you need the index variable, use `enumerate`:

```python
for k, element in enumerate(my_list):
    ...
```

A similar construction for arrays is `ndenumerate`:

```python
a = ones((3, 5))
for k, el in ndenumerate(a):
    print(k, el)
# prints something like this: (1, 3) 1.0
```

## 9.2 Controlling the flow inside the loop

The commands `break` and `continue` jump out of the loop or skip to the next iteration. The keyword `break` terminates the loop before it is completely executed. Two situations can occur: the loop is completely executed, or it is left when reaching `break`. For the first case, special actions can be defined in an `else` block, executed only if the whole list is traversed. This is useful for iterating algorithms not guaranteed to succeed:

```python
maxIteration = 10000
for iteration in range(maxIteration):
    residual = compute()   # some computation
    if residual < tolerance:
        break
else:   # only executed if the for loop is not broken
    raise Exception("The algorithm did not converge")
print(f"The algorithm converged in {iteration + 1} steps")
```

## 9.3 Iterable objects

A `for` loop picks the elements of a list one at a time, so there is no need to store the whole list in memory. The mechanism that allows this is the **iterator**. An **iterable object** produces objects to be passed to a loop, and may be used inside a loop as if it were a list:

```python
for element in obj:
    ...
```

The objects may be produced on the fly. A typical iterable is the object returned by `range`: the successive integers are produced on the fly when needed, not created all at once:

```python
for iteration in range(100000000):
    # Note: the 100000000 integers are not created at once
    if iteration > 10:
        break
```

To get a real list, form it explicitly: `l = list(range(100000000))`.

Iterable objects have a method `__iter__`. The datatypes met so far that are iterable objects are:

- lists
- tuples
- strings
- range objects
- dictionaries
- arrays
- `enumerate` and `ndenumerate` objects

By executing `__iter__` on an iterable object, an **iterator** is created (done tacitly by a `for` loop). An iterator has a `__next__` method, which returns the next element of a sequence:

```python
l = [1, 2]
li = l.__iter__()
li.__next__()   # returns 1
li.__next__()   # returns 2
li.__next__()   # raises StopIteration exception
```

### 9.3.1 Generators

You can create your own iterator using the keyword `yield`. A generator for odd numbers smaller than `n`:

```python
def odd_numbers(n):
    "generator for odd numbers less than n"
    for k in range(n):
        if k % 2 == 1:
            yield k
```

Then use it as follows:

```python
g = odd_numbers(10)
for k in g:
    ...   # do something with k

for k in odd_numbers(10):
    ...   # do something with k
```

### 9.3.2 Iterators are disposable

A salient feature of iterators is that they may be used only once. To use it again, you must create a new iterator object. An iterable object can create new iterators as many times as necessary:

```python
L = ['a', 'b', 'c']
iterator = iter(L)
list(iterator)   # ['a', 'b', 'c']
list(iterator)   # [] empty list, because the iterator is exhausted

new_iterator = iter(L)   # new iterator, ready to be used
list(new_iterator)       # ['a', 'b', 'c']
```

Each time a generator object is called, it creates a new iterator. When that iterator is exhausted, you must call the generator again:

```python
g = odd_numbers(10)
for k in g:
    ...   # do something with k
# now the iterator is exhausted:
for k in g:   # nothing will happen!!
    ...
# to loop through it again, create a new one:
g = odd_numbers(10)
for k in g:
    ...
```

### 9.3.3 Iterator tools

A couple of handy iterator tools:

| Tool | Description |
| --- | --- |
| `enumerate` | Enumerates another iterator, yielding pairs `(iteration, element)`. |
| `reversed` | Creates an iterator from a list by going through it backward. |
| `itertools.count` | A possibly infinite iterator of integers. |
| `itertools.islice` | Truncates an iterator using the familiar slicing syntax. |

```python
A = ['a', 'b', 'c']
for iteration, x in enumerate(A):
    print(iteration, x)   # (0, 'a') (1, 'b') (2, 'c')
```

```python
A = [0, 1, 2]
for elt in reversed(A):
    print(elt)   # result: 2 1 0
```

```python
for iteration in itertools.count():
    if iteration > 100:
        break   # without this, the loop goes on forever
    print(f'integer: {iteration}')
```

`islice` can create a finite iterator from an infinite one:

```python
from itertools import count, islice
for iteration in islice(count(), 10):
    # same effect as range(10)
    ...
```

Combining `islice` with an infinite generator:

```python
def odd_numbers():
    k = -1
    while True:   # this makes it an infinite generator
        k += 1
        if k % 2 == 1:
            yield k

list(itertools.islice(odd_numbers(), 10, 30, 8))
# returns [21, 37, 53]
```

### 9.3.4 Generators of recursive sequences

For a sequence given by an induction formula, such as the Fibonacci sequence, generators offer a nifty solution:

```python
def fibonacci(u0, u1):
    """
    Infinite generator of the Fibonacci sequence.
    """
    yield u0
    yield u1
    while True:
        u0, u1 = u1, u1 + u0
        # we shifted the elements and compute the new one
        yield u1
```

```python
# sequence of the 100 first Fibonacci numbers:
list(itertools.islice(fibonacci(0, 1), 100))
```

### 9.3.5 Examples for iterators in mathematics

**Arithmetic geometric mean.** The AGM iteration generates two sequences that converge to the same value, used here to compute complete elliptic integrals of the first kind:

```python
def arithmetic_geometric_mean(a, b):
    """
    Generator for the arithmetic and geometric mean
    a, b initial values
    """
    while True:   # infinite loop
        a, b = (a + b) / 2, sqrt(a * b)
        yield a, b
```

A first version of the elliptic integral relies on the (fast) convergence of the AGM iteration to terminate:

```python
def elliptic_integral(k, tolerance=1.e-5):
    """
    Compute an elliptic integral of the first kind.
    """
    a_0, b_0 = 1., sqrt(1 - k ** 2)
    for a, b in arithmetic_geometric_mean(a_0, b_0):
        if abs(a - b) < tolerance:
            return pi / (2 * a)
```

In practical computing we must make sure the algorithm stops, because theoretical convergence might fail in limited-precision arithmetic. The safe version uses `itertools.islice` with a maximum number of iterations:

```python
from itertools import islice

def elliptic_integral(k, tolerance=1e-5, maxiter=100):
    """
    Compute an elliptic integral of the first kind.
    """
    a_0, b_0 = 1., sqrt(1 - k ** 2)
    for a, b in islice(arithmetic_geometric_mean(a_0, b_0), maxiter):
        if abs(a - b) < tolerance:
            return pi / (2 * a)
    else:
        raise Exception("Algorithm did not converge")
```

As an application, the period of a pendulum:

```python
def pendulum_period(L, theta, g=9.81):
    return 4 * sqrt(L / g) * elliptic_integral(sin(theta / 2))
```

**Convergence acceleration.** A generator may take another generator as input. Aitken's Δ²-method accelerates a converging sequence:

```python
def Euler_accelerate(sequence):
    """
    Accelerate the iterator in the variable `sequence`.
    """
    s0 = sequence.__next__()   # Si
    s1 = sequence.__next__()   # Si+1
    s2 = sequence.__next__()   # Si+2
    while True:
        yield s0 - ((s1 - s0) ** 2) / (s2 - 2 * s1 + s0)
        s0, s1, s2 = s1, s2, sequence.__next__()
```

As an example, a series converging to π:

```python
def pi_series():
    sum = 0.
    j = 1
    for i in itertools.cycle([1, -1]):
        yield sum
        sum += i / j
        j += 2
```

The accelerated version and its first `N` elements:

```python
Euler_accelerate(pi_series())
list(itertools.islice(Euler_accelerate(pi_series()), N))
```

Here we stacked three generators: `pi_series`, `Euler_accelerate`, and `itertools.islice`.

## 9.4 List-filling patterns

This section compares different ways to fill lists, differing in efficiency and readability.

### 9.4.1 List filling with the append method

A ubiquitous pattern computes elements and stores them in a list:

```python
L = []
for k in range(n):
    # call various functions here
    # that compute "result"
    L.append(result)
```

Disadvantages: the number of iterations is decided in advance, and the code mixes generation, stopping, and storage; it also assumes you want the whole history when you might only need, for instance, the sum.

### 9.4.2 List from iterators

Iterators separate generating the values from the stopping condition and storage:

```python
def result_iterator():
    for k in itertools.count():   # infinite iterator
        # call various functions here
        # that compute "result"
        ...
        yield result
```

Storing the first `n` values, or summing them:

```python
L = list(itertools.islice(result_iterator(), n))   # no append needed!

# make sure that you do not use numpy.sum here
s = sum(itertools.islice(result_iterator(), n))
```

Generating elements until a condition is fulfilled uses `itertools.takewhile`, which iterates the generator as long as its Boolean function evaluates to `True`:

```python
L = list(itertools.takewhile(lambda x: abs(x) > 1.e-8,
                             result_iterator()))
```

When the result at each step does not depend on previous elements, a list comprehension may be used:

```python
L = [some_function(k) for k in range(n)]
```

List comprehensions cannot help when values depend on previously computed values.

### 9.4.3 Storing generated values

Filling lists from iterators fails if the algorithm may throw an exception: if the iterator raises along the way, the list will not be available, not even the partial result. Consider a quickly diverging power sequence:

```python
import itertools

def power_sequence(u0):
    u = u0
    while True:
        yield u
        u = u ** 2
```

Executing `list(itertools.islice(power_sequence(2.), 20))` raises an `OverflowError` and no list is available. The only way around this is to use `append` wrapped in an exception-catching block:

```python
generator = power_sequence(2.)
L = []
for iteration in range(20):
    try:
        L.append(next(generator))
    except Exception:
        break
```

## 9.5 When iterators behave as lists

Some list operations also work on iterators.

### 9.5.1 Generator expressions

There is an equivalent of list comprehension for generators, called a **generator expression**:

```python
g = (n for n in range(1000) if not n % 100)
# generator for 100, 200, ..., 900
```

This is useful for sums or products, which are incremental:

```python
sum(n for n in range(1000) if not n % 100)
# returns 4500 (sum is here the built-in function)
```

Python allows omitting the enclosing parentheses when the generator is the only argument of a function. As an example, a partial sum of the Riemann zeta function:

```python
sum(1 / n ** s for n in itertools.islice(itertools.count(1), N))
```

Equivalently, defining a generator:

```python
def generate_zeta(s):
    for n in itertools.count(1):
        yield 1 / n ** s

def zeta(N, s):
    # make sure that you do not use scipy.sum here
    return sum(itertools.islice(generate_zeta(s), N))
```

This demonstrates generators elegantly but is not the most accurate or efficient way to evaluate zeta.

### 9.5.2 Zipping iterators

A list can be created out of two or more iterators by zipping them together:

```python
xg = x_iterator()   # some iterator
yg = y_iterator()   # another iterator
for x, y in zip(xg, yg):
    print(x, y)
```

The zipped iterator stops as soon as one of the iterators is exhausted, the same behavior as `zip` on lists.

## 9.6 Iterator objects

Any object may be made iterable via the method `__iter__`, which should return an iterator. Here `__iter__` is a generator:

```python
class OdeStore:
    """
    Class to store results of ode computations
    """
    def __init__(self, data):
        "data is a list of the form [[t0, u0], [t1, u1],...]"
        self.data = data

    def __iter__(self):
        "By default, we iterate on the values u0, u1,..."
        for t, u in self.data:
            yield u
```

```python
store = OdeStore([[0, 1], [0.1, 1.1], [0.2, 1.3]])
for u in store:
    print(u)
# result: 1, 1.1, 1.3
list(store)   # [1, 1.1, 1.3]
```

Using iterator features on a non-iterable object raises an exception:

```python
list(3)
# TypeError: 'int' object is not iterable

for iteration in 3:
    pass
# TypeError: 'int' object is not iterable
```

## 9.7 Infinite iterations

Infinite iterations are obtained with an infinite iterator, a `while` loop, or recursion. In practical cases some condition stops the iteration, but unlike finite iterations, you cannot tell from a cursory examination whether they will stop.

### 9.7.1 The while loop

The `while` loop repeats a code block until a condition is fulfilled:

```python
while condition:
    <code>
```

It is equivalent to:

```python
for iteration in itertools.count():
    if not condition:
        break
    <code>
```

The danger is being trapped in an infinite loop if the condition is never fulfilled (e.g., a non-converging Newton iteration). Finite iterators are often better suited; the following construction often advantageously replaces a `while` loop:

```python
maxit = 100
for nb_iterations in range(maxit):
    ...
else:
    raise Exception(f"No convergence in {maxit} iterations")
```

Advantages: the code is guaranteed to execute in finite time, and `nb_iterations` contains the number of iterations needed for convergence.

### 9.7.2 Recursion

Recursion occurs when a function calls itself. The recursion depth (number of iterations) brings your computer to its limits:

```python
def f(N):
    if N == 0:
        return 0
    return f(N - 1)
```

For large `N` too much memory is used and the interpreter may crash. Python raises an exception when too high a recursion depth is detected; the limit can be changed:

```python
import sys
sys.setrecursionlimit(1000)
```

Choosing too high a number may imperil stability. The current limit is obtained with `sys.getrecursionlimit()`. By comparison, a non-recursive loop runs tens of millions of iterations without problem:

```python
for iteration in range(10000000):
    pass
```

If possible, recursion should be avoided in Python when an iterative alternative exists: a recursion of depth N involves N simultaneous function calls (overhead), and it is an infinite iteration hard to bound. In some special cases (tree traversal) recursion is unavoidable, and for small depths it may be preferred for readability.

## 9.8 Summary

In this chapter we studied iterators, a programming construct very near to the mathematical description of iterative methods. You saw the keyword `yield` and met finite and infinite iterators. We showed that an iterator can be exhausted, and introduced more special aspects such as generator (iterator) comprehension and recursive iterators, demonstrated with examples.
