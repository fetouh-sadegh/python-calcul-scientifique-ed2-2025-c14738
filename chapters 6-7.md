# chapters 6-7

# Chapter 6

Plotting in Python is done with the `pyplot` part of the Matplotlib module. With Matplotlib, you can create high-quality figures and visualize your results. The examples in this chapter assume the module has been imported as:

```python
from matplotlib.pyplot import *
```

In IPython, it is recommended to run the magic command `%matplotlib` directly after starting the shell to prepare it for interactive plotting.

## 6.1 Making plots with basic plotting commands

This section creates plots by means of basic commands. It is the entry point for studying how to make graphical representations of mathematical objects and data.

### 6.1.1 Using the plot command and some of its variants

- The standard plotting function is `plot`. `plot(x, y)` creates a figure window with a plot of `y` as a function of `x`. The inputs are arrays (or lists) of equal length.
- `plot(y)` is a short form of `plot(range(len(y)), y)`: the values are plotted against their index.

Example plotting `sin(x)` with markers at every fourth point:

```python
# plot sin(x) for some interval
x = linspace(-2*pi, 2*pi, 200)
plot(x, sin(x))
# plot marker for every 4th point
samples = x[::4]
plot(samples, sin(samples), 'r*')
# add title and grid lines
title('Function sin(x) and some points plotted')
grid()
```

- The standard plot is a solid blue curve; each axis is automatically scaled to fit the values.
- Color and plot options are given after the first two arguments (here `r*` is red star-shaped markers).
- `title` puts a title string above the plot area.
- Calling `plot` multiple times overlays the plots in the same window. Use `figure()` for a new clean window; `figure(2)` switches between windows by integer number (creating one if it does not exist).

Multiple plots can be explained with `legend` together with `label` arguments. The following example fits polynomials with `polyfit` and `polyval`:

```python
# Polyfit example
x = range(5)
y = [1, 2, 1, 3, 5]
p2 = polyfit(x, y, 2)   # coefficients of degree 2 polynomial
p4 = polyfit(x, y, 4)   # coefficients of degree 4 polynomial
# plot the two polynomials and points
xx = linspace(-1, 5, 200)
plot(xx, polyval(p2, xx), label='fitting polynomial of degree 2')
plot(xx, polyval(p4, xx), label='interpolating polynomial of degree 4')
plot(x, y, '*')
# set the axis and legend
axis([-1, 5, 0, 6])
legend(loc='upper left', fontsize='small')
```

- `axis([xmin, xmax, ymin, ymax])` sets the axis range manually.
- `legend` takes optional arguments on placement and formatting.

Scatter plots and logarithmic plots:

```python
# create random 2D points
import numpy
x1 = 2*numpy.random.standard_normal((2, 100))
x2 = 0.8*numpy.random.standard_normal((2, 100)) + array([[6], [2]])
plot(x1[0], x1[1], '*')
plot(x2[0], x2[1], 'r*')
title('2D scatter plot')
```

```python
# log both x and y axis
x = linspace(0, 10, 200)
loglog(x, 2*x**2, label='quadratic polynomial', linestyle='-', linewidth=3)
loglog(x, 4*x**4, label='4th degree polynomial', linestyle='-.', linewidth=3)
loglog(x, 5*exp(x), label='exponential function', linewidth=3)
title('Logarithmic plots')
legend(loc='best')
```

### 6.1.2 Formatting

The appearance of figures and plots can be styled and customized. Important variables include `linewidth`, `xlabel` and `ylabel`, `color`, and `transparent`.

Example with several formatting keywords:

```python
k = 0.2
x = [sin(2*n*k) for n in range(20)]
plot(x, color='green', linestyle='dashed', marker='o',
     markerfacecolor='blue', markersize=12, linewidth=6)
```

You may use either the short string syntax `plot(..., 'ro-')` or the more explicit syntax `plot(..., marker='o', color='r', linestyle='-')`.

**Table 6.1: Some common plot formatting arguments**

| Setting | Short string example | Explicit keyword |
| --- | --- | --- |
| Color | `'r'`, `'g'`, `'b'`, `'k'` | `color='green'` |
| Line style | `'-'`, `'--'` (dashed), `'-.'`, `':'` | `linestyle='dashed'` |
| Marker | `'*'`, `'o'`, `'.'` | `marker='o'` |
| Combined | `'go'`, `'ro-'` | — |
| Line thickness | — | `linewidth=2` |
| Marker face color | — | `markerfacecolor='blue'` |
| Marker size | — | `markersize=12` |

To set the color to green with marker `'o'`: `plot(x, 'go')`.

Histograms are produced with `hist`:

```python
# random vector with normal distribution
sigma, mu = 2, 10
x = sigma*numpy.random.standard_normal(10000) + mu
hist(x, 50, density=True)
z = linspace(0, 20, 200)
plot(z, (1/sqrt(2*pi*sigma**2))*exp(-(z-mu)**2/(2*sigma**2)), 'g')
# title with LaTeX formatting
title(fr'Histogram with $\mu={mu}, \sigma={sigma}$')
```

- Text can be formatted using LaTeX, enclosed within a pair of `$` signs.
- If string-formatting brackets interfere with LaTeX brackets, double them (`x_{1}` → `x_{{1}}`).
- Escape sequences such as `\tau` may be misinterpreted (`\t` is a tab); add `r` before the string to make it a raw string: `r'\tau'`.

Several plots can be placed in one figure window using `subplot`:

```python
def avg(x):
    """ simple running average """
    return (roll(x, 1) + x + roll(x, -1)) / 3

# sine function with noise
x = linspace(-2*pi, 2*pi, 200)
y = sin(x) + 0.4*numpy.random.standard_normal(200)
# make successive subplots
for iteration in range(3):
    subplot(3, 1, iteration + 1)
    plot(x, y, label='{:d} average{}'.format(iteration, 's' if iteration > 1 else ''))
    yticks([])
    legend(loc='lower left', frameon=False)
    y = avg(y)   # apply running average
subplots_adjust(hspace=0.7)
```

- `roll` shifts all values of the array.
- `subplot` takes three arguments: number of vertical plots, number of horizontal plots, and an index (counted row-wise).
- `subplots_adjust` adjusts the distance between subplots.

Saving figures with `savefig` (the format is set by the filename extension):

```python
savefig('test.pdf')                      # save to pdf
savefig('test.svg')                      # save to svg (editable format)
savefig('test.pdf', transparent=True)    # transparent background
savefig('test.pdf', bbox_inches='tight') # tight bounding box for LaTeX
```

### 6.1.3 Working with meshgrid and contours

To represent a scalar function over a rectangle, first generate a grid with `meshgrid`:

```python
n = ...  # number of discretization points along the x-axis
m = ...  # number of discretization points along the y-axis
X, Y = meshgrid(linspace(a, b, n), linspace(c, d, m))
```

`X` and `Y` are arrays with an `(n, m)` shape such that `X[i, j]` and `Y[i, j]` contain the coordinates of the grid points.

Level curves are drawn with `contour`. Example using Rosenbrock's banana function (global minimum at `(1, 1)`):

```python
rosenbrockfunction = lambda x, y: (1-x)**2 + 100*(y-x**2)**2
X, Y = meshgrid(linspace(-.5, 2., 100), linspace(-1.5, 4., 100))
Z = rosenbrockfunction(X, Y)
contour(X, Y, Z, logspace(-0.5, 3.5, 20, base=10), cmap='gray')
title('Rosenbrock Function: ')
xlabel('x')
ylabel('y')
```

- The fourth parameter gives the levels; here logarithmically spaced steps via `logspace`, and the colormap `gray`.
- If levels are not given, `contour` chooses appropriate levels itself.
- `contourf` does the same task but fills the plot with colors according to the levels.

Contour plots are ideal for visualizing the behavior of a numerical method. Depicting the steps of Powell's method toward the minimum:

```python
import scipy.optimize as so
rosenbrockfunction = lambda x, y: (1-x)**2 + 100*(y-x**2)**2
X, Y = meshgrid(linspace(-.5, 2., 100), linspace(-1.5, 4., 100))
Z = rosenbrockfunction(X, Y)
cs = contour(X, Y, Z, logspace(0, 3.5, 7, base=10), cmap='gray')
rosen = lambda x: rosenbrockfunction(x[0], x[1])
solution, iterates = so.fmin_powell(rosen, x0=array([0, -0.7]), retall=True)
x, y = zip(*iterates)
plot(x, y, 'ko')                  # plot black bullets
plot(x, y, 'k:', linewidth=1)     # plot black dotted lines
title("Steps of Powell's method to compute a minimum")
clabel(cs)
```

- `fmin_powell` reports all iterates when `retall=True` is given.
- `contour` returns a contour set object (`cs`), used by `clabel` to annotate the levels.

### 6.1.4 Generating images and contours

Arrays can be visualized as images with `imshow`. The following computes a color matrix for the Mandelbrot fractal, using a fixed-point iteration that depends on a complex parameter:

```python
def mandelbrot(h, w, maxit=20):
    X, Y = meshgrid(linspace(-2, 0.8, w), linspace(-1.4, 1.4, h))
    c = X + Y*1j
    z = c
    exceeds = zeros(z.shape, dtype=bool)
    for iteration in range(maxit):
        z = z**2 + c
        exceeded = abs(z) > 4
        exceeds_now = exceeded & (logical_not(exceeds))
        exceeds[exceeds_now] = True
        z[exceeded] = 2   # limit the values to avoid overflow
    return exceeds

imshow(mandelbrot(400, 400), cmap='gray')
axis('off')
```

- `imshow` displays the matrix as an image; `axis('off')` turns off the axis.
- By default `imshow` uses interpolation. To replicate pixel values, use `interpolation='nearest'`:

```python
imshow(mandelbrot(40, 40), cmap='gray')
imshow(mandelbrot(40, 40), interpolation='nearest', cmap='gray')
```

## 6.2 Working with Matplotlib objects directly

So far we used the `pyplot` module. Sometimes we want to generate a figure that should be modified later by changing some of its attributes. This requires working with graphical objects in an object-oriented way.

### 6.2.1 Creating axes objects

When creating a plot to be modified later, we need references to a figure and an axes object, assigned to variables:

```python
fig = figure()
ax = subplot(111)
```

A figure can have several axes objects depending on the use of `subplot`. Plots are then associated with a given axes object:

```python
fig = figure(1)
ax = subplot(111)
x = linspace(0, 2*pi, 100)
# a function that modulates the amplitude of the sin function
amod_sin = lambda x: (1. - 0.1*sin(25*x))*sin(x)
# and plot both...
ax.plot(x, sin(x), label='sin')
ax.plot(x, amod_sin(x), label='mod sin')
```

- These two plot commands fill the list `ax.lines` with two `Line2D` objects.
- Using labels lets us identify objects later:

```python
for il, line in enumerate(ax.lines):
    if line.get_label() == 'sin':
        break
```

### 6.2.2 Modifying line properties

A line object is an element of the list `ax.lines`. All its properties are collected in a dictionary, obtained via `ax.lines[il].properties()` (keys include `marker`, `linestyle`, `linewidth`, `color`, `xdata`, `ydata`, `label`, `alpha`, etc.). Properties are changed by corresponding setter methods:

```python
ax.lines[il].set_linestyle('-.')
ax.lines[il].set_linewidth(2)
# we can even modify the data
ydata = ax.lines[il].get_ydata()
ydata[-1] = -0.5
ax.lines[il].set_ydata(ydata)
```

### 6.2.3 Making annotations

The axes method `annotate` sets an annotation at a given position and points, with an arrow, to another position. The arrow can be given properties in a dictionary:

```python
annot1 = ax.annotate('amplitude modulated\n curve', (2.1, 1.0), (3.2, 0.5),
                     arrowprops={'width': 2, 'color': 'k',
                                 'connectionstyle': 'arc3,rad=+0.5',
                                 'shrink': 0.05},
                     verticalalignment='bottom', horizontalalignment='left',
                     fontsize=15,
                     bbox={'facecolor': 'gray', 'alpha': 0.1, 'pad': 10})
annot2 = ax.annotate('corrupted data', (6.3, -0.5), (6.1, -1.1),
                     arrowprops={'width': 0.5, 'color': 'k', 'shrink': 0.1},
                     horizontalalignment='center', fontsize=12)
```

- The arrow points to the first coordinate; the text's left-bottom corner is the second. Coordinates default to the data coordinate system.
- `'shrink': 0.05` reduces the arrow size by 5% to keep a distance to the curve.
- `connectionstyle` shapes the arrow (e.g. a spline arc).
- An annotation assigned to a variable can be removed with `annot1.remove()`.

### 6.2.4 Filling areas between curves

Filling highlights differences between curves. It is done with the axes method `fill_between`:

```python
axf = ax.fill_between(x, sin(x), amod_sin(x), facecolor='gray')
```

The `where` parameter takes a Boolean array specifying additional filling conditions:

```python
axf = ax.fill_between(x, sin(x), amod_sin(x),
                      where=amod_sin(x)-sin(x) > 0, facecolor='gray')
```

- Remove a complete filling before trying the partial filling: `axf.remove()`.
- Related commands: `fill` for filling polygons, `fill_betweenx` for filling in horizontal directions.

### 6.2.5 Defining ticks and tick labels

Ticks and tick labels can be set manually. Note the LaTeX way of setting labels with Greek letters:

```python
ax.set_xticks(array([0, pi/2, pi, 3/2*pi, 2*pi]))
ax.set_xticklabels(('$0$', '$\pi/2$', '$\pi$', '$3/2 \pi$', '$2 \pi$'), fontsize=18)
ax.set_yticks(array([-1., 0., 1]))
ax.set_yticklabels(('$-1$', '$0$', '$1$'), fontsize=18)
```

It is good practice to increase the font size so the figure can be scaled down into a document without affecting readability.

### 6.2.6 Setting spines makes your plot more instructive

Spines are the lines with ticks and labels displaying the coordinates. By default Matplotlib places four (bottom, right, top, left). They can be deselected with `set_visible` and positioned in data coordinates with `set_position`:

```python
fig = figure(1)
ax = fig.add_axes((0., 0., 1, 1))
ax.spines["left"].set_position(('data', 0.))
ax.spines["bottom"].set_position(("data", 0.))
ax.spines["top"].set_visible(False)
ax.spines["right"].set_visible(False)
x = linspace(-2*pi, 2*pi, 200)
ax.plot(x, arctan(10*x), label=r'$\arctan(10 x)$')
ax.legend()
```

Spines carry ticks and tick labels. Matplotlib supports two sets of ticks, referred to as `'minor'` and `'major'`:

```python
ax.set_xticks([-2*pi, -pi, pi, 2*pi])
ax.set_xticklabels([r"$-2\pi$", r"$-\pi$", r"$\pi$", r"$2\pi$"])
ax.set_yticks([pi/4, pi/2], minor=True)
ax.set_yticklabels([r"$\pi/4$", r"$\pi/2$"], minor=True)
ax.set_yticks([-pi/4, -pi/2], minor=False)
ax.set_yticklabels([r"$-\pi/4$", r"$-\pi/2$"], minor=False)   # major label set
ax.tick_params(axis='both', which='major', labelsize=12)
ax.tick_params(axis='y', which='major', pad=-35)   # move labels to the right
ax.tick_params(axis='both', which='minor', labelsize=12)
```

## 6.3 Making 3D plots

The toolkit `mplot3d` provides 3D plotting of points, lines, contours, and surfaces. A 3D plot is generated by adding the keyword `projection='3d'` to the axes object:

```python
from mpl_toolkits.mplot3d import axes3d
fig = figure()
ax = fig.gca(projection='3d')
# plot points in 3D
class1 = 0.6 * random.standard_normal((200, 3))
ax.plot(class1[:, 0], class1[:, 1], class1[:, 2], 'o')
class2 = 1.2 * random.standard_normal((200, 3)) + array([5, 4, 0])
ax.plot(class2[:, 0], class2[:, 1], class2[:, 2], 'o')
class3 = 0.3 * random.standard_normal((200, 3)) + array([0, 3, 2])
ax.plot(class3[:, 0], class3[:, 1], class3[:, 2], 'o')
```

Surface plots use `plot_surface` (the `alpha` value sets transparency):

```python
X, Y, Z = axes3d.get_test_data(0.05)
fig = figure()
ax = fig.gca(projection='3d')
# surface plot with transparency 0.5
ax.plot_surface(X, Y, Z, alpha=0.5)
```

Contours can be projected onto any coordinate plane:

```python
fig = figure()
ax = fig.gca(projection='3d')
ax.plot_wireframe(X, Y, Z, rstride=5, cstride=5)
# plot contour projection on each axis plane
ax.contour(X, Y, Z, zdir='z', offset=-100)
ax.contour(X, Y, Z, zdir='x', offset=-40)
ax.contour(X, Y, Z, zdir='y', offset=40)
# set axis limits
ax.set_xlim3d(-40, 40)
ax.set_ylim3d(-40, 40)
ax.set_zlim3d(-100, 100)
# set labels
ax.set_xlabel('X axis')
ax.set_ylabel('Y axis')
ax.set_zlabel('Z axis')
```

- For 3D plots the 2D command `axis([...])` does not work; use the object-oriented versions `ax.set_xlim3d(-40, 40)`, etc.
- There is no `zlabel` command; use `ax.set_xlabel(...)`, `ax.set_ylabel(...)`, `ax.set_zlabel(...)`.

## 6.4 Making movies from plots

Data that evolves can be saved as a movie. One way uses the module `visvis`. Example of an evolving circle represented implicitly by the zero level set of a function:

```python
import visvis.vvmovie as vv
# create initial function values
x = linspace(-255, 255, 511)
X, Y = meshgrid(x, x)
f = sqrt(X*X + Y*Y) - 40   # radius 40
# evolve and store in a list
imlist = []
for iteration in range(200):
    imlist.append((f > 0)*255)
    f -= 1   # move outwards one pixel
vv.images2swf.writeSwf('circle_evolution.swf', imlist)
```

- `visvis` can also save GIF and (on some platforms) AVI animations.
- Another option is to use `savefig` to create one image per frame, then combine them with video software (e.g. Mencoder or ImageMagick):

```python
x = linspace(-255, 255, 511)
X, Y = meshgrid(x, x)
f = sqrt(X*X + Y*Y) - 40   # radius 40
for iteration in range(200):
    imshow((f > 0)*255, aspect='auto')
    gray()
    axis('off')
    savefig('circle_evolution_{:d}.png'.format(iteration))
    f -= 1
```

## 6.5 Summary

A graphical representation is the most compact form in which to present mathematical results or the behavior of an algorithm. This chapter provided the basic tools for plotting and introduced a more sophisticated, object-oriented way to work with graphical objects such as figures, axes, and lines. You learned how to make classical x/y plots, 3D plots, and histograms, an appetizer on making films, and how to modify plots as graphical objects with methods and attributes that can be set, deleted, or modified.

# Chapter 7

This chapter introduces functions, a fundamental building block in programming. It shows how to define them, how to handle input and output, how to properly use them, and how to treat them as objects.

## 7.1 Functions in mathematics and functions in Python

In mathematics, a function uniquely assigns to every element from the domain a corresponding element from the range. When working with functions, we distinguish two steps:

- The **definition** of the function (done once).
- The **evaluation** of the function, the computation of `f(x)` for a given value of `x` (done many times).

A Python function is defined with the keyword `def`:

```python
def subtract(x1, x2):
    return x1 - x2
```

- `subtract` is the function's name and `x1`, `x2` are its parameters. The colon indicates a block command. The returned value follows the `return` keyword.
- The function is called with its parameters replaced by input arguments:

```python
r = subtract(5.0, 4.3)   # result 0.7 is assigned to r
```

## 7.2 Parameters and arguments

When defining a function, its input variables are called **parameters**. The input used when executing the function is called its **argument**.

### 7.2.1 Passing arguments — by position and by keyword

Only the order of the arguments matters; arguments can be any object:

```python
z = 3
e = subtract(5, z)
```

Arguments can also be passed by keyword, using the parameter names:

```python
z = 3
e = subtract(x2=z, x1=5)
```

Both ways can be combined, but positional arguments must come first:

```python
plot(xp, yp, linewidth=2, label='y-values')
```

### 7.2.2 Changing arguments

Changing the value of a parameter inside the function normally has no effect outside it. This applies to all **immutable** arguments (strings, numbers, tuples):

```python
def subtract(x1, x2):
    z = x1 - x2
    x2 = 50.
    return z

a = 20.
b = subtract(10, a)   # returns -10
a                     # still has the value 20
```

The situation differs for **mutable** arguments (lists, dictionaries), which can be changed outside the function:

```python
def subtract(x):
    z = x[0] - x[1]
    x[1] = 50.
    return z

a = [10, 20]
b = subtract(a)   # returns -10
a                 # is now [10, 50.0]
```

The authors strongly dissuade changing input arguments inside a function.

### 7.2.3 Access to variables defined outside the local namespace

Python functions can access variables defined in enclosing program units, called **global variables** (as opposed to **local variables**):

```python
import numpy as np   # here the variable np is defined
def sqrt(x):
    return np.sqrt(x)   # we use np inside the function
```

This feature should not be abused. The following is an example of what *not* to do, because changing `a` tacitly changes the function's behavior:

```python
a = 3
def multiply(x):
    return a * x   # bad style: access to the variable a defined outside
```

It is much better to provide the variable as a parameter:

```python
def multiply(x, a):
    return a * x
```

### 7.2.4 Default arguments

A **default value** is given in the function definition; if the function is called without that argument, Python uses the default. The calls below for the Frobenius norm of the identity matrix are equivalent:

```python
import scipy.linalg as sl
sl.norm(identity(3))
sl.norm(identity(3), ord='fro')
sl.norm(identity(3), 'fro')
```

Default arguments are given by assigning a value to a parameter in the definition:

```python
def subtract(x1, x2=0):
    return x1 - x2
```

Positional arguments must be given first; keyword arguments may be omitted if they have default values.

**Beware of mutable default arguments** — defaults are set upon function definition, so changing a mutable default has a side effect:

```python
def my_list(x1, x2=[]):
    x2.append(x1)
    return x2

my_list(1)   # returns [1]
my_list(2)   # returns [1, 2]
```

### 7.2.5 Variable number of arguments

Lists and dictionaries can be used to call functions with a variable number of arguments via starred arguments:

```python
data = [[1, 2], [3, 4]]
style = dict({'linewidth': 3, 'marker': 'o', 'color': 'green'})
plot(*data, **style)
```

- A name prefixed by `*` (e.g. `*data`) unpacks a list to provide **positional** arguments.
- A name prefixed by `**` (e.g. `**style`) unpacks a dictionary to **keyword** arguments.
- The reverse process packs positional arguments into a list and keyword arguments into a dictionary; in a function definition this is indicated by parameters prefixed with `*` and `**`. You will often find `*args` and `**kwargs` in documentation.

## 7.3 Return values

A function in Python always returns a single object. To return more than one object, they are packed and returned as a single tuple:

```python
def complex_to_polar(z):
    r = sqrt(z.real ** 2 + z.imag ** 2)
    phi = arctan2(z.imag, z.real)
    return (r, phi)   # here the return object is formed
```

The returned tuple can be unpacked in a single line:

```python
z = 3 + 5j
r, phi = complex_to_polar(z)
```

- If a function has no `return` statement, it returns `None`. This is common when the function modifies a mutable argument in place or prints/writes output:

```python
def append_to_list(L, x):
    L.append(x)
```

- The list methods `append`, `extend`, `reverse`, and `sort` modify in place and return `None`.
- Execution stops at the first `return` statement; lines after it are dead code:

```python
def function_with_dead_code(x):
    return 2 * x
    y = x ** 2   # these two lines ...
    return y     # ... are never executed!
```

## 7.4 Recursive functions

Many mathematical functions are defined recursively. Recursion makes the relation to the mathematical definition clear, but should be used with care in scientific computing, as the iterative approach is usually more efficient.

Chebyshev polynomials are defined by a three-term recursion:

```python
def chebyshev(n, x):
    if n == 0:
        return 1.
    elif n == 1:
        return x
    else:
        return 2. * x * chebyshev(n - 1, x) - chebyshev(n - 2, x)

chebyshev(5, 0.52)   # returns 0.39616645119999994
```

- This example wastes computation time: the number of evaluations grows exponentially with the recursion level, and most are duplicates. A technique called **memoization** combines recursion with caching to avoid replicated evaluations.
- A recursive function has a **level parameter** (here `n`) controlling two parts: the **base case** (the first two `if` branches) and the **recursive body** (where the function calls itself with smaller-level parameters).
- The number of levels is the **recursion depth**. Too large a depth raises:

```python
RuntimeError: maximum recursion depth exceeded
```

This error also occurs when the initialization step is missing.

## 7.5 Function documentation

Document functions using a string at the beginning, called a **docstring**:

```python
def newton(f, x0):
    """
    Newton's method for computing a zero of a function
    on input:
    f  (function) given function f(x)
    x0 (float) initial guess
    on return:
    y  (float) the approximated zero of f
    """
    ...
```

- `help(newton)` displays the docstring together with the call signature.
- The docstring is saved as the attribute `newton.__doc__`.
- The minimal information to provide is the purpose of the function and the description of the input and output objects. Tools such as Sphinx can collect docstrings to generate full documentation.

## 7.6 Functions are objects

Functions are objects like everything else in Python. They can be passed as arguments, renamed, or deleted:

```python
def square(x):
    """
    Return the square of x
    """
    return x ** 2

square(4)   # 16
sq = square   # now sq is the same as square
sq(4)         # 16
del square    # square doesn't exist anymore
print(newton(sq, .2))   # passing as argument
```

Passing functions as arguments is very common in scientific computing (e.g. `fsolve` in `scipy.optimize`, `quad` in `scipy.integrate`). Make sure the passed function `f` has exactly the form described in the docstring of the receiving function.

### 7.6.1 Partial application

**Partial application** is the process of defining a new function by fixing (freezing) one or several parameters of a function. It is easily created with `functools.partial`:

```python
import functools
def sin_omega(t, freq):
    return sin(2 * pi * freq * t)

def make_sine(frequency):
    return functools.partial(sin_omega, freq=frequency)

fomega = make_sine(0.25)
fomega(3)   # returns -1
```

### 7.6.2 Using closures

Since functions are objects, partial applications can be realized by writing a function that returns a new function with a reduced number of input arguments:

```python
def make_sine(freq):
    "Make a sine function with frequency freq"
    def mysine(t):
        return sin_omega(t, freq)
    return mysine
```

Here the inner function `mysine` has access to the variable `freq`, which is neither a local variable nor passed via the argument list. This construction is a **closure**.

## 7.7 Anonymous functions — the keyword lambda

The keyword `lambda` defines **anonymous functions**, that is, functions without a name described by a single expression. We demonstrate using SciPy's `quad`, which takes the function to integrate and the integration bounds:

```python
import scipy.integrate as si
si.quad(lambda x: x ** 2 + 5, 0, 1)
```

The syntax is:

```python
lambda parameter_list: expression
```

A lambda definition can only consist of a single expression and cannot contain loops. Lambda functions are objects and can be assigned to variables:

```python
parabola = lambda x: x ** 2 + 5
parabola(3)   # gives 14
```

### 7.7.1 The lambda construction is always replaceable

The lambda construction is only syntactic sugar; any lambda may be replaced by an explicit function definition:

```python
parabola = lambda x: x**2 + 5
# the following code is equivalent
def parabola(x):
    return x ** 2 + 5
```

Lambda functions provide a third way to make closures, for example computing the integral of a sine for various frequencies:

```python
import scipy.integrate as si
for iteration in range(3):
    print(si.quad(lambda x: sin_omega(x, iteration*pi), 0, pi/2.))
```

## 7.8 Functions as decorators

A **decorator** is a syntax element that conveniently alters the behavior of a function without changing the function's own definition. Consider a function determining the sparsity of a matrix, which fails on objects (like lists) lacking a `reshape` method:

```python
def how_sparse(A):
    return len(A.reshape(-1).nonzero()[0])
```

A helper function modifies any one-parameter function so that it first casts its input to an array:

```python
def cast2array(f):
    def new_function(obj):
        fA = f(array(obj))
        return fA
    return new_function
```

The modified function `how_sparse = cast2array(how_sparse)` can then be applied to any object castable to an array. The same is achieved by decorating the definition:

```python
@cast2array
def how_sparse(A):
    return len(A.reshape(-1).nonzero()[0])
```

A decorator is a callable object that modifies the definition of the function to be decorated. The main purposes are:

- To increase code readability by separating parts not directly serving the function's functionality (e.g. memoizing).
- To put common preamble and epilogue parts of a family of similar functions in a common place (e.g. type checking).
- To easily switch additional functionalities on and off (e.g. test prints or tracing).

It is recommended to also consider `functools.wraps`.

## 7.9 Summary

Functions are ideal tools for making your program modular, and they reflect mathematical thinking. You learned the syntax of function definitions and how to distinguish between defining and calling a function. We considered functions as objects that can be modified by other functions, the notion of the scope of a variable, and how information is passed into a function by parameters. Finally, we introduced the keyword `lambda` for defining anonymous functions on the fly.
