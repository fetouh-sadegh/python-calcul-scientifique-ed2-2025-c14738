# Chapter 18

This chapter covers parallel computing and the module `mpi4py`. Complex and time-consuming computational tasks can often be divided into subtasks that are carried out simultaneously when there is capacity for it. When the subtasks are independent of each other, executing them in parallel is especially efficient. Tasks where subtasks must wait for one another are less suited to parallel computing.

The guiding example throughout the chapter is computing an integral by a quadrature rule: when evaluating the integrand is time-consuming and the number of points is large, the problem is split into smaller subtasks distributed either to several computers or to the cores of a single multicore machine, and the results are then communicated back to a controlling processor that performs the final additions.

Topics covered:

- Multicore computers and computer clusters
- Message passing interface (MPI)

## 18.1 Multicore computers and computer clusters

Most modern computers are multicore computers. For instance, a processor with four cores allows performing four computational tasks in parallel. Four cores with two threads each are often counted as eight CPUs by system monitors, but for this chapter only the number of cores matters.

- The cores share a common memory (the RAM) and each has individual cache memory.
- Cache memory is used optimally by its core and accessed at high speed, while shared memory can be accessed by all cores of one CPU. On top of that there is the computer's RAM and finally the hard disk, which is also shared memory.
- A different setting for parallel computing is a **computer cluster**, where a task is divided into parallelizable subtasks sent to different computers, sometimes over long distances. Here, communication time can matter substantially. Using a cluster makes sense only if the time for processing subtasks is large compared to the communication time.

## 18.2 Message passing interface (MPI)

Programming for several cores or on a computer cluster with distributed memory requires special techniques. The chapter describes **message passing** and related tools standardized by the MPI standard. These tools are similar across programming languages such as C, C++, and FORTRAN, and are realized in Python by the module `mpi4py`.

### 18.2.1 Prerequisites

Install the module first by executing the following in a terminal:

```
conda install mpi4py
```

Import the module by adding this line to your Python script:

```python
import mpi4py as mpi
```

Parallelized code is executed from a terminal with the command `mpiexec`. Assuming the code is stored in `script.py`, running it on a computer with a four-core CPU is done by:

```
mpiexec -n 4 python script.py
```

To execute the same script on a cluster of two computers:

```
mpiexec --hostfile=hosts.txt python script.py
```

You must provide a file `hosts.txt` containing the names or IP addresses of the computers together with the number of their cores you want to bind to a cluster:

```
# Content of hosts.txt
192.168.1.25 :4   # master computer with 4 cores
192.168.1.101:2   # worker computer with 2 cores
```

The Python script has to be copied to all computers in the cluster.

## 18.3 Distributing tasks to different cores

When executed on a multicore computer, `mpiexec` copies the given Python script to the number of cores and runs each copy. As an example, the one-liner `print_me.py` containing `print("Hello it's me")`, when executed with `mpiexec -n 4 print_me.py`, generates the same message on screen four times, each sent from a different core.

To execute different tasks on different cores, we have to distinguish these cores in the script. To this end, we create a **communicator** instance, which organizes communication between the world (input/output units such as the screen, keyboard, or a file) and the individual cores. The cores are given identifying numbers called a **rank**:

```python
from mpi4py import MPI
comm = MPI.COMM_WORLD   # making a communicator instance
rank = comm.Get_rank()  # querying for the numeric identifier of the core
size = comm.Get_size()  # the total number of cores assigned
```

The communicator attribute `size` refers to the total number of processes specified in the `mpiexec` statement.

We can now give every core an individual task, as in the script `basicoperations.py`:

```python
from mpi4py import MPI
comm = MPI.COMM_WORLD   # making a communicator instance
rank = comm.Get_rank()  # querying for the numeric identifier of the core
size = comm.Get_size()  # the total number of cores assigned
a = 15
b = 2
if rank == 0:
    print(f'Core {rank} computes {a}+{b}={a+b}')
if rank == 1:
    print(f'Core {rank} computes {a}*{b}={a*b}')
if rank == 2:
    print(f'Core {rank} computes {a}**{b}={a**b}')
```

Executed with `mpiexec -n 3 python basicoperations.py`, we obtain three messages. All three processes got their individual tasks, executed in parallel. Printing to the screen is a bottleneck, as the screen is shared by all processes.

### 18.3.1 Information exchange between processes

There are different ways to send and receive information between processes:

- Point-to-point communication
- One-to-all and all-to-one
- All-to-all

Speaking to a neighbor and letting information pass along a street illustrates the first type from daily life, while the daily news, spoken by one person and broadcast to a big group of listeners, illustrates the second.

### 18.3.2 Point-to-point communication

Point-to-point communication directs information flow from one process to a designated receiving process. It is applied in scientific computing, for instance in random-walk or particle-tracing applications on domains divided into subdomains.

The **ping-pong** example assumes two processors sending an integer back and forth, each increasing its value by one. We first create a communicator and check that we have two processes:

```python
from mpi4py import MPI
comm = MPI.COMM_WORLD   # making a communicator instance
rank = comm.Get_rank()  # querying for the numeric identifier of the core
size = comm.Get_size()  # the total number of cores assigned
if not (size == 2):
    raise Exception(f"This examples requires two processes. \
{size} processes given.")
```

Then we send information back and forth between the two processes:

```python
count = 0
text = ['Ping', 'Pong']
print(f"Rank {rank} activities:\n==================")
while count < 5:
    if rank == count % 2:
        print(f"In round {count}: Rank {rank} says {text[count%2]}"
              f" and sends the ball to rank {(rank+1)%2}")
        count += 1
        comm.send(count, dest=(rank+1) % 2)
    elif rank == (count+1) % 2:
        count = comm.recv(source=(rank+1) % 2)
```

- Information is sent by the communicator method `send`, provided with the information to send and the destination. The communicator translates the destination to a hardware address.
- The other machine receives via `comm.recv`, which requires information on where the data is expected from. Under the hood, it tells the sender that the information has been received by freeing the information buffer on the data channel. The sender awaits this signal before it can proceed.
- The two statements `if rank == count%2` and `elif rank == (count+1)%2` ensure that the processors alternate their sending and receiving tasks.

Saved as `pingpong.py` and executed with `mpiexec -n 2 python pingpong.py`, the script produces alternating Ping/Pong rounds 0 to 4.

What data can be sent or received? As `send` and `recv` communicate data in binary form, they **pickle** the data first (see Section 14.3: Pickling). Most Python objects can be pickled, but not lambda functions. Buffered data such as NumPy arrays can also be pickled, but a direct send of buffered data is more efficient.

Note that `send` and `recv` only communicate **references** to functions, so the references must exist on both the sending and receiving processors. The following script returns an error because `func` is not defined on the processor with rank 1:

```python
from mpi4py import MPI
comm = MPI.COMM_WORLD   # making a communicator instance
rank = comm.Get_rank()  # querying for the numeric identifier of the core
size = comm.Get_size()  # the total number of cores assigned
if rank == 0:
    def func():
        return 'Function called'
    comm.send(func, dest=1)
if rank == 1:
    f = comm.recv(source=0)   # <<<<<< This line reports an error
    print(f())
```

The error thrown by `recv` is `AttributeError: Can't get attribute 'func'`. The correct way is to define the function for both processors:

```python
from mpi4py import MPI
comm = MPI.COMM_WORLD   # making a communicator instance
rank = comm.Get_rank()  # querying for the numeric identifier of the core
size = comm.Get_size()  # the total number of cores assigned
def func():
    return 'Function called'
if rank == 0:
    comm.send(func, dest=1)
if rank == 1:
    f = comm.recv(source=0)
    print(f())
```

### 18.3.3 Sending NumPy arrays

The commands `send` and `recv` are high-level commands: they do under-the-hood work, deducing the datatype and the amount of buffer data needed and allocating memory, based on low-level C constructions. NumPy arrays themselves make use of these C-buffer-like objects, so when sending and receiving NumPy arrays you can gain efficiency by using the lower-level counterparts `Send` and `Recv` (mind the capitalization!).

```python
from mpi4py import MPI
comm = MPI.COMM_WORLD   # making a communicator instance
rank = comm.Get_rank()  # querying for the numeric identifier of the core
size = comm.Get_size()  # the total number of cores assigned
import numpy as np
if rank == 0:
    A = np.arange(700)
    comm.Send(A, dest=1)
if rank == 1:
    A = np.empty(700, dtype=int)  # This is needed for memory allocation
                                  # of the buffer on Processor 1
    comm.Recv(A, source=0)        # Note the difference to recv in
                                  # providing the data.
    print(f'An array received with last element {A[-1]}')
```

- On both processors, memory for the buffer has to be allocated. On Processor 0 by creating an array with the data, and on Processor 1 an array with the same size and datatype but arbitrary data.
- The command `Recv` returns the buffer via its first argument. This is possible because NumPy arrays are mutable.

### 18.3.4 Blocking and non-blocking communication

The commands `send`/`recv` and their buffer counterparts `Send`/`Recv` are **blocking** commands. A command `send` is completed when the corresponding send buffer is freed, which depends on the communication architecture and the amount of data. The `send` is considered freed when the corresponding `recv` has got all the information. Without such a `recv`, it waits forever: this is a **deadlock**.

The following script has deadlock potential because both processes send simultaneously. If the amount of data is too big to be stored, `send` waits for a corresponding `recv` to empty the pipe, but `recv` is never invoked due to the waiting state:

```python
from mpi4py import MPI
comm = MPI.COMM_WORLD   # making a communicator instance
rank = comm.Get_rank()  # querying for the numeric identifier of the core
size = comm.Get_size()  # the total number of cores assigned
if rank == 0:
    msg = ['Message from rank 0', list(range(101000))]
    comm.send(msg, dest=1)
    print(f'Process {rank} sent its message')
    s = comm.recv(source=1)
    print(f'I am rank {rank} and got a {s[0]} with a list of \
length {len(s[1])}')
if rank == 1:
    msg = ['Message from rank 1', list(range(-101000, 1))]
    comm.send(msg, dest=0)
    print(f'Process {rank} sent its message')
    s = comm.recv(source=0)
    print(f'I am rank {rank} and got a {s[0]} with a list of \
length {len(s[1])}')
```

Executing this might not cause a deadlock on your computer if the amount of data is very small. The straightforward remedy is to **swap the order** of `recv` and `send` on one of the processors:

```python
if rank == 0:
    msg = ['Message from rank 0', list(range(101000))]
    comm.send(msg, dest=1)
    print(f'Process {rank} sent its message')
    s = comm.recv(source=1)
    print(f'I am rank {rank} and got a {s[0]} with a list of \
length {len(s[1])}')
if rank == 1:
    s = comm.recv(source=0)
    print(f'I am rank {rank} and got a {s[0]} with a list of \
length {len(s[1])}')
    msg = ['Message from rank 1', list(range(-101000, 1))]
    comm.send(msg, dest=0)
    print(f'Process {rank} sent its message')
```

### 18.3.5 One-to-all and all-to-one communication

When a complex task depending on a large amount of data is divided into subtasks, the data also has to be divided into portions relevant to each subtask, and the results have to be assembled into a final result.

Consider the scalar product of two vectors divided into subtasks. The steps are:

1. Creating the vectors `u` and `v`.
2. Dividing them into `m` subvectors with a balanced number of elements (equal sizes if `N` is divisible by `m`, otherwise some subvectors have more elements).
3. Communicating each subvector to "its" processor.
4. Performing the scalar product on the subvectors on each processor.
5. Gathering all results.
6. Summing up the results.

Steps 1, 2, and 6 run on one processor, the **root processor** (here rank 0). Steps 3, 4, and 5 are executed on all processors, including the root. For the communication in Step 3, `mpi4py` provides `scatter`, and for recollecting the results `gather`.

**Preparing the data for communication** — a script that splits a vector into `m` balanced pieces:

```python
def split_array(vector, n_processors):
    # splits an array into a number of subarrays
    # vector         one dimensional ndarray or a list
    # n_processors   integer, the number of subarrays to be formed
    n = len(vector)
    n_portions, rest = divmod(n, n_processors)  # division with remainder
    # get the amount of data per processor and distribute the rest on
    # the first processors so that the load is more or less equally
    # distributed
    # Construction of the indexes needed for the splitting
    counts = [0] + [n_portions + 1 \
        if p < rest else n_portions for p in range(n_processors)]
    counts = numpy.cumsum(counts)
    start_end = zip(counts[:-1], counts[1:])         # a generator
    slice_list = (slice(*sl) for sl in start_end)    # a generator comprehension
    return [vector[sl] for sl in slice_list]         # a list of subarrays
```

This uses NumPy's cumulative sum `cumsum`, the generator `zip`, argument unpacking with `*`, generator comprehension, and the `slice` data type.

**The commands `scatter` and `gather`** — the entire script for the parallel scalar product:

```python
from mpi4py import MPI
import numpy as np
comm = MPI.COMM_WORLD
rank = comm.Get_rank()
nprocessors = comm.Get_size()
import splitarray as spa
if rank == 0:
    # Here we generate data for the example
    n = 150
    u = 0.1 * np.arange(n)
    v = -u
    u_split = spa.split_array(u, nprocessors)
    v_split = spa.split_array(v, nprocessors)
else:
    # On all processors we need variables with these names,
    # otherwise we would get an Exception "Variable not defined" in
    # the scatter command below
    u_split = None
    v_split = None
# These commands run now on all processors
u_split = comm.scatter(u_split, root=0)  # the data is portion wise
                                         # distributed from root
v_split = comm.scatter(v_split, root=0)
# Each processor computes its part of the scalar product
partial_dot = u_split @ v_split
# Each processor reports its result back to the root
partial_dot = comm.gather(partial_dot, root=0)
if rank == 0:
    # partial_dot is a list of all collected results
    total_dot = np.sum(partial_dot)
    print(f'The parallel scalar product of u and v'
          f' on {nprocessors} processors is {total_dot}.\n'
          f'The difference to the serial computation is \
{abs(total_dot - u@v)}')
```

Executed with `mpiexec -n 5 python parallel_dot.py`, the difference to the serial computation is `0.0`.

- To use `scatter`, the root processor provides a **list** with as many elements as there are processors; each element contains the data for one processor (including the root).
- The reversing process is `gather`: when all processors complete it, the root receives a list with as many elements as processors, each containing the resulting data of its corresponding processor.
- The art of parallel programming is to avoid bottlenecks: ideally all processors are busy and start/stop simultaneously, so the workload is distributed equally and the periods with the root working alone are short.

**A final data reduction operation — the command `reduce`.** The amount of data from all processors is often reduced to a single number in the last step. The command `reduce` does the gathering and summation in one step:

```python
# Each processor reports its result back to the root
# and these results are summed up
total_dot = comm.reduce(partial_dot, op=MPI.SUM, root=0)
if rank == 0:
    print(f'The parallel scalar product of u and v'
          f' on {nprocessors} processors is {total_dot}.\n'
          f'The difference to the serial computation \
is {abs(total_dot - u@v)}')
```

Other frequently applied reducing operations:

| Operation | Result |
| --- | --- |
| `MPI.SUM` | Sum of the partial results |
| `MPI.MAX` / `MPI.MIN` | Maximum or minimum of the partial results |
| `MPI.MAXLOC` / `MPI.MINLOC` | The argmax or argmin of the partial results |
| `MPI.PROD` | The product of the partial results |
| `MPI.LAND` / `MPI.LOR` | The logical and / logical or of the partial results |

**Sending the same message to all.** Another collective command is the broadcasting command `bcast`. In contrast to `scatter`, it sends the **same** data to all processors:

```python
data = comm.bcast(data, root=0)
```

Here the total data, not a list of portioned data, is sent. The root processor is the one that prepares the data to be broadcasted.

**Buffered data.** In an analogous manner, `mpi4py` provides the corresponding collective commands for buffer-like data such as NumPy arrays by capitalizing the command:

| Pickled (Python objects) | Buffered (NumPy arrays) |
| --- | --- |
| `scatter` | `Scatter` |
| `gather` | `Gather` |
| `reduce` | `Reduce` |
| `bcast` | `Bcast` |

## 18.4 Summary

In this chapter, we saw how to execute copies of the same script on different processors in parallel. Message passing allows the communication between these different processes. We saw point-to-point communication and the two collective communication types, one-to-all and all-to-one. The commands presented are provided by the Python module `mpi4py`, a Python wrapper to realize the MPI standard in C.

Only the most essential commands and concepts were described here. Concepts left out include grouping processes and tagging information, which are important for special and challenging applications too particular for this introduction.

# Chapter 19

In this chapter we present some comprehensive and longer examples together with a brief introduction to the theoretical background and the complete implementation. The goal is to show how the concepts defined throughout the book are used in practice.

Topics covered:

- Polynomials
- The polynomial class
- Spectral clustering
- Solving initial value problems

## 19.1 Polynomials

First, we demonstrate the power of the Python constructs presented so far by designing a class for polynomials. This class differs conceptually from the class `numpy.poly1d`. We give some theoretical background leading to a list of requirements, then the code with comments.

### 19.1.1 Theoretical background

A polynomial is defined by its degree, representation, and coefficients. The polynomial representation written as a linear combination of monomials is called the **monomial representation**. Alternatively, the polynomial can be written in:

- **Newton representation** with coefficients and points.
- **Lagrange representation** with coefficients and points, using the cardinal functions.

There are infinitely many representations, but we restrict ourselves to these three typical ones.

A polynomial can be determined from **interpolation conditions** with given distinct values and arbitrary values as input.

- In the **Lagrange** formulation, the interpolation polynomial is directly available, as its coefficients are the interpolation data.
- The coefficients in **Newton** representation are obtained by a recursion formula, the **divided differences** formula.
- The coefficients in **monomial** representation are obtained by solving a linear system (a Vandermonde system).

A matrix that has a given polynomial (or a multiple of it) as its characteristic polynomial is called a **companion matrix**. Its eigenvalues are the zeros (roots) of the polynomial. An algorithm for computing the zeros sets up the companion matrix and then computes the eigenvalues with `scipy.linalg.eig`.

### 19.1.2 Tasks

The programming tasks:

1. Write a class `PolyNomial` with the attributes `points`, `degree`, `coeff`, and `basis`, where `points` is a list of tuples, `degree` is the degree of the interpolation polynomial, `coeff` contains the coefficients, and `basis` is a string stating which representation is used.
2. Provide a method for evaluating the polynomial at a given point.
3. Provide a method `plot` that plots the polynomial over a given interval.
4. Write a method `__add__` that returns a polynomial that is the sum of two polynomials (only in the monomial case can the sum be computed by just summing the coefficients).
5. Write a method that computes the coefficients of the polynomial in monomial form.
6. Write a method that computes the polynomial's companion matrix.
7. Write a method that computes the zeros of the polynomial via the eigenvalues of the companion matrix.
8. Write a method that computes the polynomial that is the k-th derivative of the given polynomial.
9. Write a method that checks whether two polynomials are equal (zero leading coefficients should not matter).

### 19.1.3 The polynomial class

We design a polynomial base class based on a monomial formulation. The polynomial can be initialized either by giving its coefficients with respect to the monomial basis or by giving a list of interpolation points:

```python
import scipy.linalg as sl
import matplotlib.pyplot as mp

class PolyNomial:
    base = 'monomial'

    def __init__(self, **args):
        if 'points' in args:
            self.points = array(args['points'])
            self.xi = self.points[:, 0]
            self.coeff = self.point_2_coeff()
            self.degree = len(self.coeff) - 1
        elif 'coeff' in args:
            self.coeff = array(args['coeff'])
            self.degree = len(self.coeff) - 1
            self.points = self.coeff_2_point()
        else:
            self.points = array([[0, 0]])
            self.xi = array([1.])
            self.coeff = self.point_2_coeff()
            self.degree = 0
```

`__init__` uses the `**args` construction (see Section 7.2.5: Variable number of arguments). If no arguments are given, a zero polynomial is assumed.

If the polynomial is given by interpolation points, the coefficients are computed by solving a Vandermonde system:

```python
def point_2_coeff(self):
    return sl.solve(vander(self.x), self.y)
```

From given coefficients, interpolation points are constructed:

```python
def coeff_2_point(self):
    points = [[x, self(x)] for x in linspace(0, 1, self.degree + 1)]
    return array(points)
```

The command `self(x)` does a polynomial evaluation via the `__call__` method, which uses the NumPy command `polyval`:

```python
def __call__(self, x):
    return polyval(self.coeff, x)
```

Two convenience methods are decorated with the `property` decorator so the result is accessed as if it were an attribute (`p.x` rather than `p.x()`):

```python
@property
def x(self):
    return self.points[:, 0]

@property
def y(self):
    return self.points[:, 1]
```

A `__repr__` method is defined for a quick check of the results:

```python
def __repr__(self):
    txt = f'Polynomial of degree {self.degree} \n'
    txt += f'with coefficients {self.coeff} \n in {self.base} basis.'
    return txt
```

A method for plotting the polynomial, using `vectorize` (see Section 4.8):

```python
margin = .05
plotres = 500

def plot(self, ab=None, plotinterp=True):
    if ab is None:  # guess a and b
        x = self.x
        a, b = x.min(), x.max()
        h = b - a
        a -= self.margin * h
        b += self.margin * h
    else:
        a, b = ab
    x = linspace(a, b, self.plotres)
    y = vectorize(self.__call__)(x)
    mp.plot(x, y)
    mp.xlabel('$x$')
    mp.ylabel('$p(x)$')
    if plotinterp:
        mp.plot(self.x, self.y, 'ro')
```

The `__call__` method is specific to the monomial representation and has to be changed for other representations. The same holds for the companion matrix:

```python
def companion(self):
    companion = eye(self.degree, k=-1)
    companion[0, :] -= self.coeff[1:] / self.coeff[0]
    return companion
```

Once the companion matrix is available, the zeros are given by its eigenvalues:

```python
def zeros(self):
    companion = self.companion()
    return sl.eigvals(companion)
```

### 19.1.4 Usage examples of the polynomial class

First, create a polynomial instance from given interpolation points:

```python
p = PolyNomial(points=[(1, 0), (2, 3), (3, 8)])
```

The coefficients with respect to the monomial basis are available as an attribute:

```python
p.coeff  # returns array([ 1., 0., -1.]) (rounded)
```

The default plot is obtained by `p.plot((-3.5, 3.5))`. Finally, compute the zeros:

```python
pz = p.zeros()  # returns array([-1.+0.j, 1.+0.j])
p(pz)           # returns array([0.+0.j, 0.+0.j])
```

### 19.1.5 Newton polynomial

The class `NewtonPolynomial` defines a polynomial with respect to the Newton basis. It inherits common methods from the base class (for example, `plot`, `zeros`, and parts of `__init__`) using `super` (see Section 8.5: Subclasses and inheritance):

```python
class NewtonPolynomial(PolyNomial):
    base = 'Newton'

    def __init__(self, **args):
        if 'coeff' in args:
            try:
                self.xi = array(args['xi'])
            except KeyError:
                raise ValueError('Coefficients need to be given'
                                 'together with abscissae values xi')
        super(NewtonPolynomial, self).__init__(**args)
```

Once the interpolation points are given, the coefficients are computed by divided differences, programmed as a generator (see Section 9.3.1: Generators and Section 9.4: List-filling patterns):

```python
def point_2_coeff(self):
    return array(list(self.divdiff()))

def divdiff(self):
    xi = self.xi
    row = self.y
    yield row[0]
    for level in range(1, len(xi)):
        row = (row[1:] - row[:-1]) / (xi[level:] - xi[:-level])
        if allclose(row, 0):  # check: elements of row nearly zero
            self.degree = level - 1
            break
        yield row[0]
```

A brief check:

```python
# here we define the interpolation data: (x, y) pairs
pts = array([[0., 0], [.5, 1], [1., 0], [2, 0.]])
pN = NewtonPolynomial(points=pts)  # creates an instance of the
                                   # polynomial class
pN.coeff  # returns the coefficients array([0., 2., -4., 2.66666667])
print(pN)
```

`print` executes the `__repr__` of the base class and returns:

```
Polynomial of degree 3
with coefficients [ 0. 2. -4. 2.66666667]
in Newton basis.
```

The polynomial evaluation differs from the base class. `NewtonPolynomial.__call__` overrides `PolyNomial.__call__`:

```python
def __call__(self, x):
    # first compute the sequence 1, (x-x_1), (x-x_1)(x-x_2),...
    nps = hstack([1., cumprod(x - self.xi[:self.degree])])
    return self.coeff @ nps
```

The companion matrix also overrides the parent method, making use of Boolean arrays:

```python
def companion(self):
    degree = self.degree
    companion = eye(degree, k=-1)
    diagonal = identity(degree, dtype=bool)
    companion[diagonal] = self.x[:degree]
    companion[:, -1] -= self.coeff[:degree] / self.coeff[degree]
    return companion
```

## 19.2 Spectral clustering

An interesting application of eigenvectors is clustering data. Using the eigenvectors of a matrix derived from a distance matrix, unlabeled data can be separated into groups. Spectral clustering methods get their name from the use of the spectrum of this matrix.

A distance matrix (the pairwise distance between data points) is a symmetric matrix. Given a distance matrix `M`, we create the **Laplacian matrix** `L = I - D M D`, where `I` is the identity matrix and `D` is the diagonal matrix containing the (inverse square root of the) row sums of `M`. The data clusters are obtained from the eigenvectors of `L`. In the simplest case of two classes, the first eigenvector (the one corresponding to the largest eigenvalue) is often enough to separate the data.

A simple two-class clustering example:

```python
import scipy.linalg as sl
# create some data points
n = 100
x1 = 1.2 * random.randn(n, 2)
x2 = 0.8 * random.randn(n, 2) + tile([7, 0], (n, 1))
x = vstack((x1, x2))
# pairwise distance matrix
M = array([[sqrt(sum((x[i] - x[j])**2))
            for i in range(2*n)]
           for j in range(2*n)])
# create the Laplacian matrix
D = diag(1 / sqrt(M.sum(axis=0)))
L = identity(2*n) - dot(D, dot(M, D))
# compute eigenvectors of L
S, V = sl.eig(L)
# As L is symmetric the imaginary parts in the eigenvalues are
# only due to negligible numerical errors
S = S.real
V = V.real
```

The eigenvector corresponding to the largest eigenvalue gives the grouping (for example by thresholding at 0):

```python
largest = abs(S).argmax()
plot(V[:, largest])
```

For more difficult datasets and more classes, one usually takes the eigenvectors corresponding to the largest eigenvalues and then clusters the data with another method, using the eigenvectors instead of the original data points. A common choice is the **k-means** clustering algorithm:

```python
import scipy.linalg as sl
import scipy.cluster.vq as sc
# simple 4 class data
x = random.rand(1000, 2)
ndx = ((x[:, 0] < 0.4) | (x[:, 0] > 0.6)) & \
      ((x[:, 1] < 0.4) | (x[:, 1] > 0.6))
x = x[ndx]
n = x.shape[0]
# pairwise distance matrix
M = array([[sqrt(sum((x[i] - x[j])**2)) for i in range(n)]
           for j in range(n)])
# create the Laplacian matrix
D = diag(1 / sqrt(M.sum(axis=0)))
L = identity(n) - dot(D, dot(M, D))
# compute eigenvectors of L
_, _, V = sl.svd(L)
k = 4
# take k first eigenvectors
eigv = V[:k, :].T
# k-means
centroids, dist = sc.kmeans(eigv, k)
clust_id = sc.vq(eigv, centroids)[0]
```

- The eigenvectors are computed here using the singular value decomposition `sl.svd`. As `L` is symmetric, the result is the same as with `sl.eig`, but `svd` returns the eigenvectors already ordered corresponding to the ordering of the eigenvalues.
- `svd` returns three arrays, the left and right singular vectors `U` and `V`, and the singular values `S`. As we do not need `U` and `S`, we use throw-away variables: `_, _, V = sl.svd(L)`.

The result is plotted with:

```python
for i in range(k):
    ndx = where(clust_id == i)[0]
    plot(x[ndx, 0], x[ndx, 1], 'o')
axis('equal')
```

## 19.3 Solving initial value problems

We consider the mathematical task of numerically solving a system of ordinary differential equations for given initial values. The solution is a function; a numerical method computes approximations at discrete points within the interval of interest. We collect the data describing the problem in a class:

```python
class IV_Problem:
    """
    Initial value problem (IVP) class
    """
    def __init__(self, rhs, y0, interval, name='IVP'):
        """
        rhs       'right hand side' function of the ordinary differential
                  equation f(t,y)
        y0        array with initial values
        interval  start and end value of the interval of independent
                  variables; often initial and end time
        name      descriptive name of the problem
        """
        self.rhs = rhs
        self.y0 = y0
        self.t0, self.tend = interval
        self.name = name
```

As an example, the differential equation describes a mathematical **pendulum**: the angle with respect to the vertical axis, `g` is the gravitation constant, and `l` its length. The initial angle is given and the initial angular velocity is zero:

```python
def rhs(t, y):
    g = 9.81
    l = 1.
    yprime = array([y[1], g / l * sin(y[0])])
    return yprime

pendulum = IV_Problem(rhs, array([pi / 2, 0.]), [0., 10.],
                      'mathem. pendulum')
```

There can be different views on the problem, leading to different class designs. For example, the interval or the initial values might be considered part of the solution process rather than the problem definition.

The solution process is modeled as another class:

```python
class IVPsolver:
    """
    IVP solver class for explicit one-step discretization methods
    with constant step size
    """
    def __init__(self, problem, discretization, stepsize):
        self.problem = problem
        self.discretization = discretization
        self.stepsize = stepsize

    def one_stepper(self):
        yield self.problem.t0, self.problem.y0
        ys = self.problem.y0
        ts = self.problem.t0
        while ts <= self.problem.tend:
            ts, ys = self.discretization(self.problem.rhs, ts, ys,
                                         self.stepsize)
            yield ts, ys

    def solve(self):
        return list(self.one_stepper())
```

We define two discretization schemes. The **explicit Euler method**:

```python
def expliciteuler(rhs, ts, ys, h):
    return ts + h, ys + h * rhs(ts, ys)
```

The classical **Runge-Kutta four-stage method (RK4)**:

```python
def rungekutta4(rhs, ts, ys, h):
    k1 = h * rhs(ts, ys)
    k2 = h * rhs(ts + h/2., ys + k1/2.)
    k3 = h * rhs(ts + h/2., ys + k2/2.)
    k4 = h * rhs(ts + h, ys + k3)
    return ts + h, ys + (k1 + 2*k2 + 2*k3 + k4) / 6.
```

With these, we create instances to obtain the discretized versions of the pendulum ODE, solve them, and plot the solution and the angle difference:

```python
pendulum_Euler = IVPsolver(pendulum, expliciteuler, 0.001)
pendulum_RK4 = IVPsolver(pendulum, rungekutta4, 0.001)

sol_Euler = pendulum_Euler.solve()
sol_RK4 = pendulum_RK4.solve()
tEuler, yEuler = zip(*sol_Euler)
tRK4, yRK4 = zip(*sol_RK4)

subplot(1, 2, 1), plot(tEuler, yEuler), \
    title('Pendulum result with Explicit Euler'), \
    xlabel('Time'), ylabel('Angle and angular velocity')
subplot(1, 2, 2), plot(tRK4, abs(array(yRK4) - array(yEuler))), \
    title('Difference between both methods'), \
    xlabel('Time'), ylabel('Angle and angular velocity')
```

It is worthwhile discussing alternative class designs: what should go in separate classes, and what should be bundled together.

- We strictly separated the mathematical problem from the numerical method.
- Where should the initial values go: part of the problem, part of the solver, or input parameters for the `solve` method? Looping over various initial values (as in parameter identification) is eased by leaving them as input parameters, while simulating different model variants with the same initial values makes coupling them to the problem preferable.
- We presented only solvers with a constant, given step size. Is the design appropriate for a future extension to adaptive methods, where a tolerance rather than a step size is given? Adaptive methods need to reject steps from time to time, which may conflict with the generator-based stepping mechanism in `IVPsolver.one_stepper`.
- You are encouraged to check the design of the two SciPy tools for solving initial values, namely `scipy.integrate.ode` and `scipy.integrate.odeint`.

## 19.4 Summary

Most of what was explained in this book was bundled into the three longer examples in this chapter. These examples mimic code development and give prototypes, which you are encouraged to alter and confront with your own ideas.

Code in scientific computing can have its own flavor due to its strong relationship with mathematically defined algorithms, and it is often wise to keep the relationship between code and formula visible. Python has techniques for this, as you have seen.
