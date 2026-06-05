# chapters 16-17

# Chapter 16

This chapter gives a brief introduction to using Python for symbolic computations through the module **SymPy**. While powerful commercial tools like Maple or Mathematica exist, it is often favorable to do symbolic calculations in the language you already use. SymPy is presented as a complement to NumPy and SciPy.

## 16.1 What are symbolic computations?

- **Numeric computations** are sequences of operations mainly on floating-point numbers. The result is always an *approximation* of the exact solution.
- **Symbolic computations** operate on formulas or symbols, transforming them (as in algebra or calculus) into other formulas. Numbers are inserted and a numeric evaluation is performed only at the very last step.

The difference is illustrated by computing a definite integral:

- Symbolically, the integrand is replaced by its primitive function, then the bounds are inserted to obtain a **closed-form expression** — the exact value, with no round-off error until final evaluation.
- Numerically, the integral is approximated directly by a method such as Simpson's rule:

```python
from scipy.integrate import quad
quad(lambda x: 1/(x**2+x+1), a=0, b=4)
```

This returns the value together with an estimate of the error bound.

### 16.1.1 Elaborating an example in SymPy

First, import the module and enable graphical formula display:

```python
from sympy import *
init_printing()
```

`init_printing()` makes sure formulas are presented graphically when possible. Then generate a symbol and define the integrand:

```python
x = symbols('x')
f = Lambda(x, 1/(x**2 + x + 1))
```

Here `x` is a Python object of type **Symbol** and `f` is a SymPy **Lambda** function (note the capital letter). Compute the integral symbolically:

```python
integrate(f(x), x)
```

We can check the result by differentiation:

```python
pf = Lambda(x, integrate(f(x), x))
diff(pf(x), x)
```

The result can be simplified:

```python
simplify(diff(pf(x), x))
```

The definite integral is obtained by inserting the bounds, then evaluated to a floating-point number:

```python
pf(4) - pf(0)
(pf(4) - pf(0)).evalf()   # returns 0.989661439612
```

## 16.2 Basic elements of SymPy

This section introduces the basic building blocks of SymPy. Familiarity with Python classes and data types is helpful.

### 16.2.1 Symbols — the basis of all formulas

The basic construction element of a formula is the **symbol**, created by the `symbols` command from a given string:

```python
x, y, mass, torque = symbols('x y mass torque')
```

This is a short form of:

```python
symbol_list = [symbols(l) for l in 'x y mass torque'.split()]
x, y, mass, torque = symbol_list
```

The variable name need not be identical to the symbol's string representation. Assumptions (such as integer) can be added:

```python
row_index = symbols('i', integer=True)
print(row_index**2)   # returns i**2
```

Entire sets of symbols can be defined compactly using slice-like ranges:

```python
integervariables = symbols('i:l', integer=True)
dimensions = symbols('m:n', integer=True)
realvariables = symbols('x:z', real=True)
```

Indexed variables can also be defined:

```python
A = symbols('A1:3(1:4)')
```

The rules for the index ranges follow those used for slices (Section 3.1.1).

### 16.2.2 Numbers

Python evaluates operations on numbers directly, introducing unavoidable rounding errors that would obstruct symbolic calculations. This is avoided by **sympifying** numbers:

```python
1/3                       # returns 0.3333333333333333
sympify(1)/sympify(3)     # returns 1/3
```

- `sympify` converts an integer to an object of type `sympy.core.numbers.Integer`.
- A fraction can also be represented directly as a rational number with `Rational(1, 3)`.

### 16.2.3 Functions

SymPy distinguishes between **defined** and **undefined** functions.

- **Undefined functions** are well-defined Python objects for generic functions with no special properties.
- **Defined functions** have special properties, e.g. `atan` or the `Lambda` function. Note the different names for implementations of the same mathematical function: `sympy.atan` vs `scipy.arctan`.

**Undefined functions** are created by giving `symbols` an extra class argument, or by the `Function` constructor:

```python
f, g = symbols('f g', cls=Function)
# equivalently:
f = Function('f')
g = Function('g')
```

With undefined functions, we can evaluate the general rules of calculus:

```python
x = symbols('x')
f, g = symbols('f g', cls=Function)
diff(f(x*g(x)), x)   # applies product rule and chain rule
```

An undefined function can be used as a function of several variables:

```python
x = symbols('x:3')
f(*x)
```

Note the use of the star operator to unpack a tuple. With a list comprehension we can build the gradient (list of partial derivatives):

```python
[diff(f(*x), xx) for xx in x]
# can also be written using the diff method:
[f(*x).diff(xx) for xx in x]
```

Another method is the Taylor series expansion:

```python
x = symbols('x')
f(x).series(x, 0, n=4)
```

This returns Taylor's formula together with the rest term expressed by the Landau symbol.

### 16.2.4 Elementary functions

Examples of elementary functions in SymPy are trigonometric functions and their inverses. `simplify` acts on expressions including such functions:

```python
x = symbols('x')
simplify(cos(x)**2 + sin(x)**2)   # returns 1
atan(x).diff(x) - 1./(x**2 + 1)   # returns 0
```

If you use SciPy and SymPy together, it is strongly recommended to use them in different namespaces:

```python
import numpy as np
import sympy as sym
# working with numbers
x = 3
y = np.sin(x)
# working with symbols
x = sym.symbols('x')
y = sym.sin(x)
```

### 16.2.5 Lambda functions

The SymPy counterpart to Python's anonymous functions is the command `Lambda`. Note the difference: `lambda` is a keyword while `Lambda` is a constructor. It takes two arguments: the symbol of the independent variable, and a SymPy expression to evaluate.

Example: air resistance (drag) as a function of speed:

```python
C, rho, A, v = symbols('C rho A v')
# C drag coefficient, A cross-sectional area, rho density, v speed
f_drag = Lambda(v, -Rational(1, 2)*C*rho*A*v**2)
```

This function can be evaluated by providing an argument:

```python
x = symbols('x')
f_drag(2)
f_drag(x/3)
```

Functions of several variables are created by providing a tuple as the first parameter of `Lambda`:

```python
x, y = symbols('x y')
t = Lambda((x, y), sin(x) + cos(2*y))
```

A call can be made either by directly providing several arguments, or by unpacking a tuple/list:

```python
t(pi, pi/2)        # returns -1
p = (pi, pi/2)
t(*p)              # returns -1
```

Matrix objects make it possible to define vector-valued functions, which enables computing Jacobians:

```python
F = Lambda((x, y), Matrix([sin(x) + cos(2*y), sin(x)*cos(y)]))
F(x, y).jacobian((x, y))
```

For more variables, a more compact form is convenient:

```python
x = symbols('x:2')
F = Lambda(x, Matrix([sin(x[0]) + cos(2*x[1]), sin(x[0])*cos(x[1])]))
F(*x).jacobian(x)
```

## 16.3 Symbolic linear algebra

Symbolic linear algebra is supported by SymPy's data type **matrix**.

### 16.3.1 Symbolic matrices

In its simplest form, `Matrix` converts a list of lists into a matrix. Example — a rotation matrix:

```python
phi = symbols('phi')
rotation = Matrix([[cos(phi), -sin(phi)],
                   [sin(phi),  cos(phi)]])
```

> **Note:** With SymPy matrices, the operator `*` performs **matrix multiplication**, not elementwise multiplication as with NumPy arrays.

The rotation matrix can be checked for orthogonality using matrix multiplication and the transpose:

```python
simplify(rotation.T*rotation - eye(2))   # returns a 2 x 2 zero matrix
simplify(rotation.T - rotation.inv())
```

This shows how a matrix is transposed (`.T`), how the identity matrix is created (`eye(2)`), and how the inverse is computed (`.inv()`).

Another way to set up a matrix is by providing a list of symbols and a shape:

```python
M = Matrix(3, 3, symbols('M:3(:3)'))
```

A third way is to generate entries by a given function, with the syntax `Matrix(number_of_rows, number_of_columns, function)`. Example — a Toeplitz matrix (constant diagonals):

```python
def toeplitz(n):
    a = symbols('a:' + str(2*n))
    f = lambda i, j: a[i-j+n-1]
    return Matrix(n, n, f)
```

Executing `toeplitz(5)` gives a matrix where all elements along the sub- and superdiagonals are the same. Elements can be accessed and changed by indexes and slices:

```python
M[0, 2] = 0                     # changes one element
M[1, :] = Matrix(1, 3, [1, 2, 3])   # changes an entire row
```

### 16.3.2 Examples for linear algebra methods in SymPy

The basic task in linear algebra is to solve linear equation systems. Done symbolically for a matrix:

```python
A = Matrix(3, 3, symbols('A1:4(1:4)'))
b = Matrix(3, 1, symbols('b1:4'))
x = A.LUsolve(b)
```

The output of this relatively small problem is already barely readable; `simplify` helps detect canceling terms and collect common factors:

```python
simplify(x)
```

> **Note:** Symbolic computations become very slow when increasing matrix dimensions. For dimensions bigger than 15, memory problems might even occur.

## 16.4 Substitutions

Consider a simple symbolic expression:

```python
x, a = symbols('x a')
b = x + a
```

Setting `x = 0` does **not** change `b`: it only rebinds the Python variable `x` to the integer `0`. The symbol represented by the string `'x'` remains unaltered, and so does `b`.

Instead, altering an expression by replacing symbols with numbers, other symbols, or expressions is done with the `subs` method:

```python
x, a = symbols('x a')
b = x + a
c = b.subs(x, 0)
d = c.subs(a, 2*a)
print(c, d)   # returns (a, 2*a)
```

This method takes one or two arguments. The following are equivalent:

```python
b.subs(x, 0)
b.subs({x: 0})   # a dictionary as argument
```

A dictionary allows several substitutions in one step:

```python
b.subs({x: 0, a: 2*a})   # several substitutions in one
```

As dictionary items have no defined order, SymPy first makes the substitutions within the dictionary and then on the expression, so permuting the items does not affect the result:

```python
x, a, y = symbols('x a y')
b = x + a
b.subs({a: a*y, x: 2*x, y: a/y})
b.subs({y: a/y, a: a*y, x: 2*x})   # same result
```

A third alternative is a list of old-value/new-value pairs:

```python
b.subs([(y, a/y), (a, a*y), (x, 2*x)])
```

Entire expressions can be substituted for others:

```python
n, alpha = symbols('n alpha')
b = cos(n*alpha)
b.subs(cos(n*alpha), 2*cos(alpha)*cos((n-1)*alpha) - cos((n-2)*alpha))
```

For matrix elements, taking the Toeplitz matrix again, the substitution `T.subs(T[0,2], 0)` changes the symbol object at position `[0, 2]`, which automatically affects the two other places where it occurs. Alternatively, a variable can be created for the symbol:

```python
a2 = symbols('a2')
T.subs(a2, 0)
```

A more complex example turns the Toeplitz matrix into a tridiagonal one by generating a list of symbols, zipping them into pairs, and substituting:

```python
symbs = [symbols('a' + str(i)) for i in range(19) if i < 3 or i > 5]
substitutions = list(zip(symbs, len(symbs)*[0]))
T.subs(substitutions)
```

## 16.5 Evaluating symbolic expressions

In scientific computing there is often the need to first make symbolic manipulations and then convert the symbolic result into a floating-point number. The central tool is **`evalf`**:

```python
pi.evalf()   # returns 3.14159265358979
```

- The resulting object has data type **Float** (a SymPy type allowing arbitrary precision).
- The default precision is 15 digits but can be changed by passing a positive integer argument:

```python
pi.evalf(30)   # returns 3.14159265358979323846264338328
```

With arbitrary precision, numbers can be arbitrarily small, breaking the limits of classical floating-point representation. Evaluating a SymPy function with an input of type `Float` returns a `Float` with the same precision as the input.

### 16.5.1 Example: A study on the convergence order of Newton's method

An iterative method converges with order *q* if there is a positive constant *C* such that the error ratio is bounded. Newton's method has order 2 with a good initial value, and for certain problems even higher. Applied to a particular problem, the iteration converges **cubically** (q = 3): the number of correct digits triples from iteration to iteration. Demonstrating this is hardly possible with the standard 16-digit float type, so SymPy with high-precision evaluation is used:

```python
import sympy as sym
x = sym.Rational(1, 2)
xns = [x]
for i in range(1, 9):
    x = (x - sym.atan(x)*(1 + x**2)).evalf(3000)
    xns.append(x)
```

The extreme precision requirement (3,000 digits) enables evaluating seven terms of the sequence to demonstrate cubic convergence:

```python
import numpy as np
# Test for cubic convergence
print(np.array(np.abs(np.diff(xns[1:])) / np.abs(np.diff(xns[:-1]))**3,
               dtype=np.float64))
```

The result is a list of seven terms that suggest C ≈ 2/3:

```python
[0.41041618, 0.65747717, 0.6666665, 0.66666667, 0.66666667,
 0.66666667, 0.66666667]
```

### 16.5.2 Converting a symbolic expression into a numeric function

The numerical evaluation of symbolic expressions is done in three steps: symbolic computation, substituting values by numbers, and evaluating to a float with `evalf`. Symbolic computations are often done for **parameter studies**, which requires turning a symbolic expression into a numeric function.

This is demonstrated by an interpolation example introducing the command **`lambdify`**. The quadratic interpolation polynomial coefficients depend on a free parameter `t`:

```python
t = symbols('t')
x = [0, t, 1]
# The Vandermonde matrix
V = Matrix([[0, 0, 1], [t**2, t, 1], [1, 1, 1]])
y = Matrix([0, 1, -1])         # the data vector
a = simplify(V.LUsolve(y))     # the coefficients
# the leading coefficient as a function of the parameter
a2 = Lambda(t, a[0])
```

`lambdify` turns the expression into a numeric function. It takes two arguments, the independent variable and a SymPy function:

```python
leading_coefficient = lambdify(t, a2(t))
```

This function can now be plotted:

```python
import numpy as np
import matplotlib.pyplot as mp
t_list = np.linspace(-0.4, 1.4, 200)
ax = mp.subplot(111)
lc_list = [leading_coefficient(t) for t in t_list]
ax.plot(t_list, lc_list)
ax.axis([-.4, 1.4, -15, 10])
ax.set_xlabel('Free parameter $t$')
ax.set_ylabel('$a_2(t)$')
```

The plot clearly shows the singularities due to coinciding interpolation points.

## 16.6 Summary

This chapter introduced the world of symbolic computations and gave a glimpse of the power of SymPy. You learned how to set up symbolic expressions, work with symbolic matrices, and make simplifications. Working with symbolic functions and transforming them into numerical evaluations established the link to scientific computing and floating-point results. SymPy's full integration into Python, with its powerful constructs and legible syntax, makes it a strong tool — consider this chapter an appetizer rather than a complete menu.

# Chapter 17

This chapter is an add-on putting Python into the context of the operating system. It shows how Python is used in a command window (console), how system commands and Python commands interact, and how to build a new application. Three main desktop operating systems are in use — Windows 10, macOS, and Linux — and this chapter discusses Python within the **Linux** world, with all examples tested in Ubuntu. The principles apply to the other systems as well.

Topics covered:

- Running a Python program in a Linux shell
- The module `sys`
- How to execute Linux commands from Python

## 17.1 Running a Python program in a Linux shell

A terminal window comes with a command prompt. To execute the Python commands in a file named `myprogram.py`, there are two choices:

- Executing the command `python myprogram.py`
- Executing the command `myprogram.py` directly

The second variant needs some preparation. First, give permission to execute the file:

```
chmod myprogram.py o+x
```

`chmod` stands for changing the file mode. The modes `o+x` mean "give (+) the owner (o) rights to execute (x) that file."

Then find the location of the `python` command:

```
which python
```

This returns a path such as `/home/claus/anaconda3/bin/python`. This information is written in the first line of the script (after the **shebang** `#!`) to tell the system which program transforms the text file into an executable:

```python
#! /home/claus/anaconda3/bin/python
a = 3
b = 4
c = a + b
print(f'Summing up {a} and {b} gives {c}')
```

The first line is just a comment to Python but is read by Linux after the shebang `#!`. Instead of an absolute path, a logical path can be used to make the code more portable:

```python
#! /usr/bin/env python   # a logical path to Python (mind the blank)
```

The program can then be executed directly in the console:

```
./example.py
```

The prefix `./` tells the operating system to look in the current directory. Without it, Linux expects the file in one of the directories listed in the search path.

## 17.2 The module sys

The module `sys` provides tools to communicate from a Python script with system commands: providing the script with arguments from the command line and outputting results to the console.

### 17.2.1 Command-line arguments

Consider this code saved in `demo_cli.py`:

```python
#! /usr/bin/env python
import sys
text = f"""
You called the program {sys.argv[0]}
with the {len(sys.argv)-1} arguments, namely {sys.argv[1:]}"""
print(text)
```

After `chmod o+x demo_cli.py`, the script can be executed with arguments. The arguments given in the console are accessible via the list `sys.argv`:

- The first element, `sys.argv[0]`, is the name of the script.
- The other elements are the given arguments as strings.

Arguments are given to the *call* of the script. They should not be confused with user input *during* execution.

### 17.2.2 Input and output streams

The command `print` displays a message in the terminal. Its counterpart is the command `input`, which prompts for data. The module `sys` makes it possible to treat the keyboard as a file object for input (e.g. `readline`, `readlines`) and the console as a file object for output (e.g. `write`, `writelines`).

The information flow is organized in UNIX by three streams:

| Stream | Description | Python object |
| --- | --- | --- |
| **STDIN** | standard input stream | `sys.stdin` |
| **STDOUT** | standard output stream | `sys.stdout` |
| **STDERR** | standard error stream | `sys.stderr` |

Example — a script `pyinput.py` that sums up some numbers:

```python
#!/usr/bin/env python3
from numpy import *
import sys
# Terminate input by CTRL+D
a = array(sys.stdin.readlines()).astype(float)
print(f'Input: {a}, Sum {sum(a)}')
```

`sys.stdin.readlines()` establishes a generator. `array` iterates it until the user inputs an end-of-input symbol — **CTRL-D** on Linux or **CTRL-Z** on Windows.

#### Redirecting streams

The standard input expects a stream from the keyboard, but can be redirected from a file with the redirection symbol `<`. No modification of the script is required:

```
./pyinput.py < intest.txt
```

Output is displayed in the terminal by default but can be redirected to a file with `>`. The file is overwritten if it exists; to append, use `>>`:

```
./pyinput.py < intest.txt > result.txt
```

To separate output from error or warning messages, Linux provides two output channels, `sys.stdout` and `sys.stderr`. Both refer to the terminal by default, but error messages can be redirected separately. Example modified to generate an error when no input is given:

```python
#!/usr/bin/env python3
from numpy import *
import sys
# Terminate input by CTRL+D
a = array(sys.stdin.readlines()).astype(float)
print(f'Input: {a}, Sum {sum(a)}')
if a.size == 0:
    sys.stderr.write('No input given\n')
```

Error messages are those from uncaught exceptions, even syntax errors, and text explicitly written to `sys.stderr`. The different redirection situations are summarized in Table 17.1:

| Task | Python Object | Redirect Symbol | Alt. Symbol |
| --- | --- | --- | --- |
| Data input | `sys.stdin` | `<` | |
| Data output | `sys.stdout` | `>` | `1>` |
| Data output append to file | `sys.stdout` | `>>` | `1>>` |
| Error output | `sys.stderr` | `2>` | |
| Error output append to file | `sys.stderr` | `2>>` | |
| All output | `sys.stdout`, `sys.stderr` | `&>` | |
| All output append to file | `sys.stdout`, `sys.stderr` | `&>>` | |

#### Building a pipe between a Linux command and a Python script

Instead of passing data between programs via a file, a **Linux pipe** lets data flow in a direct stream from one command to another, using the pipe symbol `|`. A pure Linux example checks whether a computer is connected to the home network by scanning the output of `ifconfig`:

- `ifconfig` — the entire network information.
- `grep` — find lines containing a pattern (`-A1` also shows one following line).
- `wc` — counting (`-l` counts lines).

```
ifconfig | grep -A1 wlp0s20f3 | grep 192.168 | wc -l
```

This displays just `1` when executed in the home network. The standard output (`stdout`) of one command becomes the standard input (`stdin`) of the next, with no intermediate display or files. This applies directly to Python scripts. Here Python generates a message from the count:

```python
#!/usr/bin/env python3
import sys
count = sys.stdin.readline()[0]
status = '' if count == '1' else 'not'
print(f"I am {status} at home")
```

The pipe can then be extended by adding this script at the end of the chain.

## 17.3 How to execute Linux commands from Python

This section covers how to execute Linux commands within a Python program.

### 17.3.1 The modules subprocess and shlex

To execute system commands within Python, import the module `subprocess`. Its high-level tool is `run`; the more sophisticated tool is `Popen`, used to mimic Linux pipes.

#### A complete process: subprocess.run

Demonstrated with `ls` (`ls -l` displays extended information). The simplest usage takes one argument, a list with the command split into strings:

```python
import subprocess as sp
res = sp.run(['ls', '-l'])
```

The module `shlex` provides a tool for performing this split, respecting empty spaces in filenames:

```python
import shlex
command_list = shlex.split('ls -l')   # returns ['ls', '-l']
```

`run` displays the result and returns a `subprocess.CompletedProcess` object. To process the output in Python, set the optional parameter `capture_output` to `True`, which makes the `stdout` and `stderr` streams available as attributes of the return object:

```python
import subprocess as sp
import shlex
command_list = shlex.split('ls -l')   # returns ['ls', '-l']
res = sp.run(command_list, capture_output=True)
print(res.stdout.decode())
```

The method `decode` converts a byte string to a standard string. If the command returns an error, `res.returncode` is nonzero and `res.stderr` contains the error message. The Pythonic way is to raise an error instead, by setting `check=True`:

```python
import subprocess as sp
import shlex
command = 'ls -y'
command_list = shlex.split(command)   # returns ['ls', '-y']
try:
    res = sp.run(command_list, capture_output=True, check=True)
    print(res.stdout.decode())
except sp.CalledProcessError:
    print(f"{command} is not a valid command")
```

#### Creating processes: subprocess.Popen

`subprocess.run` starts a process and waits until it ends. For example, `xclock` opens a window with a clock until it is closed:

```python
import subprocess as sp
res = sp.run(['xclock'])   # waits until the window is closed
```

`subprocess.Popen` is different: it creates a `Popen` object, so the process itself becomes a Python object that need not be completed to be accessible:

```python
import subprocess as sp
p = sp.Popen(['xclock'])
```

The process is completed by a user action on the window or by explicitly terminating it:

```python
p.terminate()
```

With `Popen`, we can construct Linux pipes in Python. The following corresponds to the UNIX pipe `ls -l | grep 'apr'` (displaying files last accessed in April):

```python
import subprocess as sp
p1 = sp.Popen(['ls', '-l'], stdout=sp.PIPE)
cp2 = sp.run(['grep', 'apr'], stdin=p1.stdout, capture_output=True)
print(cp2.stdout.decode())
```

The object `sp.PIPE` takes the output of the first process `p1` and passes it as `stdin` to `run`, which returns a `CompletedProcess` object `cp2` with the `stdout` attribute of type `bytes`. Alternatively, two `Popen` processes can be used, with the second pipe displayed through the method `communicate`:

```python
import subprocess as sp
p1 = sp.Popen(['ls', '-l'], stdout=sp.PIPE)
p2 = sp.Popen(['grep', 'apr'], stdin=p1.stdout, stdout=sp.PIPE)
print(p2.communicate()[0].decode())
```

The method `communicate` returns a tuple with the output on `stdout` and `stderr`.

## 17.4 Summary

This chapter demonstrated the interaction of a Python script with system commands. Either a Python script can be called as if it were a system command, or a Python script can itself create system processes. The chapter is based on Linux systems such as Ubuntu and serves as a demonstration of concepts and possibilities. It allows putting scientific computing tasks in an application context, where often different software — and even hardware components — have to be combined.
