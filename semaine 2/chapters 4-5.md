# chapters 4-6

# Chapter 4

Linear algebra is a key part of computational mathematics, dealing with vectors and matrices. NumPy provides all the tools needed to work with them.

Main tasks:

- Creating or modifying matrices and vectors (using slicing)
- Performing dot operations (scalar product, matrix-vector, and matrix-matrix products)
- Solving linear problems

## 4.1Overview of the array type

This section introduces the use of arrays in Python with NumPy, focusing on how to create and manipulate vectors and matrices.

---

**4.1.1 Vectors and Matrices**

- Import NumPy: `from numpy import *`
- Vectors are created using `array([..])`. Example:
    
    `v = array([1., 2., 3.])`
    
- Basic operations include scalar multiplication/division, linear combinations, elementwise operations, and functions like `norm` and `dot`.

Examples:

```python
python
CopierModifier
2 * v1, v1 / 2, 3 * v1 + 2 * v2
dot(v1, v2), v1 @ v2
v1 * v2, v1 + v2, cos(v1)

```

- Matrices are created using lists of lists:
    
    `M = array([[1., 2.], [0., 1.]])`
    
- Vectors, row matrices, and column matrices are distinct:
    
    ```python
    python
    CopierModifier
    R = v.reshape((1, 3))  # Row matrix
    C = v.reshape((3, 1))  # Column matrix
    
    ```
    

---

**4.1.2 Indexing and Slicing**

- Similar to Python lists, but matrices allow multi-dimensional indexing:
    
    ```python
    python
    CopierModifier
    v[0], v[1:], M[0, 0], M[1], M[1:]
    v[0] = 10
    v[:2] = [0, 1]  # Valid
    v[:2] = [1, 2, 3]  # Error!
    
    ```
    

---

**4.1.3 Linear Algebra Operations**

- `dot()` is used for:
    - Matrix-vector multiplication: `dot(M, v)` or `M @ v`
    - Scalar product: `dot(v, w)` or `v @ w`
    - Matrix-matrix multiplication: `dot(M, N)` or `M @ N`
- Solving linear systems:
    
    ```python
    python
    CopierModifier
    from numpy.linalg import solve
    x = solve(A, b)
    allclose(dot(A, x), b)  # Checks solution
    
    ```
    

---

This overview introduces how to use arrays for linear algebra, including basic operations, reshaping, indexing, dot products, and solving systems of equations. Further details are provided in later sections.

## **4.2 Mathematical Preliminaries**

This section provides the mathematical foundation needed to understand how arrays function in NumPy, particularly emphasizing the connection between array indexing and function evaluation.

---

**4.2.1 Arrays as Functions**

Arrays can be viewed mathematically as functions:

- A vector behaves like a function from a set of indices to values.
- A matrix can be seen as a function of two variables (row, column) mapping to a value.
    
    This view helps later when understanding advanced topics like broadcasting.
    

---

**4.2.2 Elementwise Operations**

Operations in NumPy are **elementwise**, meaning they apply point-by-point:

- Like with functions f(x)⋅g(x)f(x) \cdot g(x)f(x)⋅g(x), elementwise multiplication means:
    
    ```python
    python
    CopierModifier
    A * B  # elementwise multiplication, not dot product
    
    ```
    

This rule applies to all scalar operations: +, -, *, /, etc.

---

**4.2.3 Shape and Number of Dimensions**

- **Scalar** = function with no arguments → shape `()`
- **Vector** = function with one argument → shape `(n,)`
- **Matrix** = function with two arguments → shape `(m, n)`
- **Higher-order tensor** = more than two arguments

Use:

```python
python
CopierModifier
shape(array)  # returns the shape
ndim(array)   # returns the number of dimensions

```

---

**4.2.4 The Dot Operations**

Linear algebra operations (dot products) are **reductions**:

- Combine dimensions of two arrays in a structured way
- Examples:
    - Vector · vector → scalar
    - Matrix · vector → vector
    - Matrix · matrix → matrix

The key rule: dimensions must match (e.g., inner dimensions in matrix multiplication).

Example:

```python
python
CopierModifier
M = array([[cos(pi/3), -sin(pi/3)],
           [sin(pi/3),  cos(pi/3)]])
v = array([1., 0.])
y = M @ v  # matrix-vector multiplication

```

- `dot(M, v)` and `M @ v` are equivalent, but `@` is more convenient.
- `dot` casts input types (e.g., lists), while `@` requires arrays.
- is always elementwise and **not** the dot product.

---

This section links arrays with their mathematical counterparts and focuses on the critical role of the dot product as a reduction operation in linear algebra.

## 4.3 The Array Type

In NumPy, the main object used to represent vectors, matrices, and higher-dimensional tensors is the `ndarray`. This section describes its key properties and how arrays are created and interpreted.

### 4.3.1 Array Properties

Each NumPy array (`ndarray`) is characterized by three key attributes:

| Name | Description |
| --- | --- |
| **shape** | Tuple indicating the size in each dimension. Tells if the array is a vector, matrix, or tensor. Accessed with `.shape`. |
| **dtype** | Specifies the data type of the array's elements (e.g., `float`, `int`, `complex`). Accessed with `.dtype`. |
| **strides** | Tells how many bytes to move in memory to get to the next element in each dimension. Used for memory layout and array views. Accessed with `.strides`. |

**Example:**

```python
python
CopierModifier
A = array([[1, 2, 3], [3, 4, 6]])
A.shape    # (2, 3)
A.dtype    # dtype('int64')
A.strides  # (24, 8)

```

- The stride `(24, 8)` means moving 24 bytes for the next row, and 8 bytes for the next column.
- This shows it's stored row-wise (C-style).

### 4.3.2 Creating Arrays from Lists

Arrays are commonly created from Python lists using the `array()` function.

**Basic Syntax:**

```python
python
CopierModifier
V = array([1., 2., 1.], dtype=float)     # Real-valued vector
V = array([1., 2., 1.], dtype=complex)   # Complex-valued vector

```

If no `dtype` is specified, NumPy tries to infer the appropriate type:

```python
python
CopierModifier
array([1, 2])          → dtype: int
array([1., 2])         → dtype: float
array([1. + 0j, 2.])   → dtype: complex

```

**Warning:** NumPy silently casts values, which can lead to unexpected results:

```python
python
CopierModifier
a = array([1, 2, 3])  # Integer array
a[0] = 0.5            # Truncated to 0
# a becomes array([0, 2, 3])

```

The same truncation can happen from complex → float → int.

### Multiline Array Syntax in Python

Thanks to Python's syntax rules, you can format arrays over multiple lines for better readability:

```python
python
CopierModifier
# Identity matrix
Id = array([[1., 0.],
            [0., 1.]])

```

This is functionally identical to writing everything on one line, but easier to read.

## 4.4 Accessing Array Entries

In NumPy, array entries are accessed using a single pair of brackets with one index per dimension, unlike nested lists which require multiple brackets.

```python
python
CopierModifier
M = array([[1., 2.], [3., 4.]])
M[0, 0]    # First row, first column → 1.0
M[-1, 0]   # Last row, first column → 3.0

```

---

### 4.4.1 Basic Array Slicing

Array slicing extends list slicing to multiple dimensions:

- `M[i, :]` → Entire row `i`
- `M[:, j]` → Entire column `j`
- `M[2:4, :]` → Rows 2 and 3 (slicing is exclusive at the end)
- `M[2:4, 1:4]` → Submatrix: rows 2 and 3, columns 1 to 3

**Default behavior:**

- `M[3]` → Same as `M[3, :]`, returns the 4th row
- `M[1:3]` → Same as `M[1:3, :]`, returns rows 2 and 3

**Slices are views:** changing the slice changes the original array:

```python
python
CopierModifier
v = array([1., 2., 3.])
v1 = v[:2]
v1[0] = 0.
# Now v is array([0., 2., 3.])

```

### General Slicing Rules (Table 4.3)

| Access Type | ndim | Kind |
| --- | --- | --- |
| `index, index` | 0 | scalar |
| `slice, index` | 1 | vector |
| `index, slice` | 1 | vector |
| `slice, slice` | 2 | matrix |

### Example Results for `M.shape = (4, 4)` (Table 4.4)

| Access | Shape | ndim | Kind |
| --- | --- | --- | --- |
| `M[:2, 1:-1]` | (2, 2) | 2 | matrix |
| `M[1, :]` | (4,) | 1 | vector |
| `M[1, 1]` | () | 0 | scalar |
| `M[1:2, :]` | (1, 4) | 2 | matrix |
| `M[1:2, 1:2]` | (1, 1) | 2 | matrix |

---

### 4.4.2 Altering an Array Using Slices

You can update array contents via direct indexing or slicing:

```python
python
CopierModifier
M[1, 3] = 2.0                # Update a scalar
M[2, :] = [1., 2., 3.]       # Replace a full row (vector)
M[1:3, :] = array([[1., 2., 3.], [-1., -2., -3.]])  # Replace rows 1 & 2

```

**Column matrix vs. vector:**

```python
python
CopierModifier
# OK: shape matches
M[1:4, 2:3] = array([[1.], [0.], [-1.]])

# Error: shape mismatch
M[1:4, 2:3] = array([1., 0., -1.])  # ValueError

```

Shapes must match exactly unless **broadcasting** applies (see Section 5.5).

### **4.5 Functions to Construct Arrays**

### **Special array constructors in NumPy:**

| Function | Shape | Description |
| --- | --- | --- |
| `zeros((n,m))` | (n, m) | Matrix filled with zeros |
| `ones((n,m))` | (n, m) | Matrix filled with ones |
| `full((n,m), q)` | (n, m) | Matrix filled with value `q` |
| `diag(v, k=0)` | (n, n) | Diagonal matrix from vector `v` (with optional offset `k`) |
| `random.rand(n,m)` | (n, m) | Matrix of random numbers in [0,1) |
| `arange(n)` | (n,) | Vector of first `n` integers |
| `linspace(a,b,n)` | (n,) | Vector with `n` points between `a` and `b` |
| `identity(n)` | (n, n) | Identity matrix |

### Variants:

- `zeros_like(A)`, `ones_like(A)` – use shape of A.
- Most support the `dtype` argument.

---

### **4.6 Accessing and Changing Shape**

### **4.6.1 `shape`**

- Returns the size of each dimension.
- `shape(M)` or `M.shape`

### **4.6.2 `ndim`**

- Returns the number of dimensions.
- `ndim(M)` or `M.ndim`

### **4.6.3 `reshape`**

- Changes shape **without copying**:

```python
python
CopierModifier
v = array([0,1,2,3,4,5])
v.reshape(2, 3)  # Shape (2,3)

```

- One dimension can be `1` to auto-calculate:

```python
python
CopierModifier
v.reshape(2, -1)  # Automatically deduces 4

```

### **Transpose**

- `A.T` gives the transpose of a matrix.
- Does not copy the data.
- Transposing a 1D vector does nothing – use `reshape`.

---

### **4.7 Stacking**

### **Concatenate and its variants:**

| Function | Action |
| --- | --- |
| `concatenate([a1, a2], axis=0)` | Stack arrays along axis 0 (vertically) |
| `hstack([a1, a2])` | Horizontal stack (axis=1) |
| `vstack([a1, a2])` | Vertical stack (axis=0) |
| `column_stack([v1, v2])` | Stack 1D vectors into columns |

### **Example – Symplectic Permutation**

```python
python
CopierModifier
def symp(v):
    n = len(v) // 2
    return hstack([v[-n:], -v[:n]])

```

---

### **4.8 Functions Acting on Arrays**

### **4.8.1 Universal Functions (UFuncs)**

- Apply elementwise:

```python
python
CopierModifier
cos(array([0, pi/2, pi]))  # => array([1, 0, -1])
array([1,2])**2  # => array([1, 4])

```

### **Making your own universal function**

- Use `vectorize`:

```python
python
CopierModifier
@vectorize
def heaviside(x):
    return 1. if x >= 0 else 0.

```

### **4.8.2 Array Functions**

- Act across axes:

```python
python
CopierModifier
sum(A)            # sum of all elements
sum(A, axis=0)    # sum along columns
sum(A, axis=1)    # sum along rows

```

## **4.9 Linear Algebra Methods in SciPy**

### **Overview**

SciPy’s `scipy.linalg` module provides advanced and efficient linear algebra tools, mostly as wrappers for LAPACK (a high-performance FORTRAN library). These methods are faster than those in `numpy.linalg`, which is more basic and array-focused.

---

### **4.9.1 Solving Several Linear Systems Using LU Factorization**

- **Goal**: Solve multiple systems Axi=biA x_i = b_iAxi=bi efficiently when matrix AAA is fixed and reused.
- **LU Factorization**:
    - Decomposes AAA into PA=LUP A = L UPA=LU where:
        - PPP: Permutation matrix
        - LLL: Lower triangular matrix
        - UUU: Upper triangular matrix
    - Use `lu_factor()` to precompute:
        
        ```python
        python
        CopierModifier
        [LU, piv] = scipy.linalg.lu_factor(A)
        
        ```
        
    - Use `lu_solve()` to solve:
        
        ```python
        python
        CopierModifier
        x_i = scipy.linalg.lu_solve((LU, piv), b_i)
        
        ```
        

---

### **4.9.2 Solving Least Squares Problems Using SVD**

- **Overdetermined System**: Ax=bA x = bAx=b, where A∈Rm×n,m>nA \in \mathbb{R}^{m \times n}, m > nA∈Rm×n,m>n
- **Goal**: Find xxx minimizing ∥Ax−b∥2\|Ax - b\|_2∥Ax−b∥2
- **SVD Factorization**: A=UΣVTA = U \Sigma V^TA=UΣVT
    - Use `svd()`:
        
        ```python
        python
        CopierModifier
        U, Sigma, VT = scipy.linalg.svd(A, full_matrices=False)
        x = VT.T @ ((U.T @ b) / Sigma)
        
        ```
        
    - Residual norm:
        
        ```python
        python
        CopierModifier
        r = A @ x - b
        nr = scipy.linalg.norm(r, 2)
        
        ```
        
- **Alternative**: `scipy.linalg.lstsq(A, b)` solves least squares directly via SVD.

---

### **4.9.3 Other Useful Methods in `scipy.linalg`**

| Function | Purpose |
| --- | --- |
| `sl.det(A)` | Determinant |
| `sl.eig(A)` | Eigenvalues and eigenvectors |
| `sl.inv(A)` | Matrix inverse |
| `sl.pinv(A)` | Pseudoinverse |
| `sl.norm(A, ord=2)` | Vector/matrix norm |
| `sl.svd(A)` | Singular Value Decomposition |
| `sl.lu(A)` | LU decomposition |
| `sl.qr(A)` | QR decomposition |
| `sl.cholesky(A)` | Cholesky decomposition |
| `sl.solve(A, b)` | Solve linear system |
| `sl.solve_banded(...)` | Solve banded systems |
| `sl.lstsq(A, b)` | Least squares solution |

---

## **4.10 Summary**

- Focused on arrays as representations of vectors and matrices.
- Solved key linear algebra problems using `scipy.linalg`, which offers fast and advanced tools via LAPACK.
- Set the stage for more advanced array operations in the next chapter.

# Chaptre 5

### **5.1 Array Views and Copies**

- **View**: A lightweight object referencing the same data as another array (no data copied).
    
    ```python
    python
    CopierModifier
    v = M[0, :]  # v is a view of M
    v[-1] = 0.   # modifies M too
    
    ```
    
    - Use `v.base` to find if `v` is a view.
    - Only **basic slicing** (`:`) returns a view; advanced indexing returns a **copy**.
    - Transposing and reshaping (e.g., `.T`, `.reshape()`) often return views.
- **Copy**: Use `array()` to explicitly copy:
    
    ```python
    python
    CopierModifier
    N = array(M.T)
    N.base is None  # True → it's a real copy
    
    ```
    

---

### **5.2 Comparing Arrays**

### **5.2.1 Boolean Arrays**

- Comparisons return **Boolean arrays**, not scalars:
    
    ```python
    python
    CopierModifier
    A == B          # returns an array
    (A == B).all()  # returns True/False
    
    ```
    

### **5.2.2 Checking Equality**

- Use `allclose(A, B)` to check if two float arrays are *approximately equal*:
    
    ```python
    python
    CopierModifier
    allclose(A, B, rtol=1e-5, atol=1e-8)
    # is equivalent to (abs(A-B) < atol + rtol*abs(B)).all()
    
    ```
    

### **5.2.3 Boolean Logic Operators**

- Avoid Python’s `and`, `or`, `not` with arrays. Use:
    
    
    | Logic | Use Instead |
    | --- | --- |
    | `and` | `&` |
    | `or` | ` |
    | `not` | `~` |
    
    ```python
    python
    CopierModifier
    (deviation < -0.5) | (deviation > 0.5)
    
    ```
    

---

### **5.3 Array Indexing**

### **5.3.1 Boolean Indexing**

- Boolean arrays can **mask** elements:
    
    ```python
    python
    CopierModifier
    M = array([[2, 3], [1, 4]])
    B = array([[True, False], [False, True]])
    M[B] = [10, 20]  # selectively updates M
    
    ```
    
- Can combine with conditions:
    
    ```python
    python
    CopierModifier
    M[M > 2] = 0
    
    ```
    

### **5.3.2 `where` Function**

- Syntax: `where(condition, a, b)`
    - Picks from `a` if condition is `True`, else from `b`.
    
    ```python
    python
    CopierModifier
    where(x < 0, 0, 1)
    
    ```
    
- If only condition is given, returns indices:
    
    ```python
    python
    CopierModifier
    where(b > 5)  # returns tuple of indices
    
    ```
    

---

### **5.4 Performance and Vectorization**

- **Python is interpreted**, so it's slower than compiled languages like C or FORTRAN.
- **NumPy** and **SciPy** use compiled code internally, making their functions much faster than pure Python implementations.
- **Example**: `numpy.dot(a, b)` is over 100x faster than a manually written scalar product using a loop.

### **5.4.1 Vectorization**

- **Vectorization** replaces slow `for` loops with fast NumPy operations.
- Example:
    - Slow: `for i in range(len(v)): w[i] = v[i] + 5`
    - Fast: `w = v + 5`
- **2D array example**:
    - Slow loop: averaging neighbors with nested loops.
    - Fast slicing:
        
        ```python
        python
        CopierModifier
        A[1:-1,1:-1] = (A[:-2,1:-1] + A[2:,1:-1] +
                        A[1:-1,:-2] + A[1:-1,2:]) / 4
        
        ```
        
- **`np.vectorize()`** can apply a custom function elementwise over an array more efficiently and more cleanly than list comprehensions.

---

### **5.5 Broadcasting**

- **Broadcasting** = automatic expansion of arrays with different shapes to make operations between them possible.
    - Example: `vector + 1` behaves as if `1` is turned into `[1, 1, 1, 1]`.

### **5.5.1 Mathematical View**

- **Concept**: Treat scalars or lower-dimensional arrays as constant functions that can be "reshaped" and "extended" to match higher dimensions.
- Example:
    - Adding a scalar to a vector: `v + 1` means the scalar is **broadcasted** over all vector elements.
    - Adding a row and column:
        
        ```python
        python
        CopierModifier
        C = np.arange(2).reshape(-1,1)  # shape (2,1)
        R = np.arange(2).reshape(1,-1)  # shape (1,2)
        C + R  # results in a (2,2) matrix via broadcasting
        
        ```
        

### **5.5.2 Broadcasting arrays**

- **Broadcasting** lets NumPy operate on arrays of different shapes by automatically reshaping and extending them to a common shape.
- If array shapes differ, NumPy:
    1. **Adds dimensions** (as 1s) to the **left** of the smaller shape.
    2. **Extends** dimensions of size 1 to match the other array.

### Examples:

- Adding a vector `(3,)` to a matrix `(4, 3)`:
    - Vector becomes `(1, 3)` → then extended to `(4, 3)` → **works**.
- Adding a vector `(3,)` to matrix `(3, 4)`:
    - Automatic broadcast **fails**.
    - Solution: reshape vector to `(3, 1)` → result shape becomes `(3, 4)` → **works**.

### **5.5.3 Typical Examples**

- **Rescale rows:**
    
    Multiply each row of matrix `M` by a coefficient vector `coeff` →
    
    Use `rescaled = M * coeff.reshape(-1, 1)`.
    
- **Rescale columns:**
    
    Multiply each column of `M` by vector `coeff` →
    
    Use `rescaled = M * coeff` or `M * coeff.reshape(1, -1)`.
    
- **Functions of two variables:**
    
    Given vectors `u` and `v`, generate matrix `W` with `W = u.reshape(-1, 1) + v`.
    
- **Using `ogrid`:**
    
    Generates shaped vectors ideal for broadcasting. Example:
    
    ```python
    python
    CopierModifier
    x, y = ogrid[0:1:3j, 0:1:3j]
    w = cos(x) + sin(2*y)
    
    ```
    

---

## **5.6 Sparse Matrices**

- Sparse matrices store only nonzero elements to save space and boost performance.
- Common in large-scale scientific computing.
- Use `scipy.sparse` for sparse matrix operations.

### **5.6.1 Formats:**

- **CSR (Compressed Sparse Row):**
    
    Uses `data`, `indices`, `indptr`. Efficient for row slicing and multiplication.
    
- **CSC (Compressed Sparse Column):**
    
    Same idea as CSR but column-based.
    
- **LIL (List of Lists):**
    
    Best for constructing and modifying matrices.
    

---

### **5.6.2 Generating Sparse Matrices:**

- Use functions like `sp.eye`, `sp.identity`, `sp.spdiags`, `sp.rand`.
- Use `sp.csr_matrix((m,n))` to create a sparse zero matrix.

---

### **5.6.3 Sparse Matrix Methods:**

- Convert between formats with `.tocsr()`, `.tocsc()`, `.tolil()`.
- Convert to full array with `.toarray()`.
- Use `dot()` from `scipy.sparse` for multiplication.
- Elementwise ops (`+`, , etc.) return CSR.
- Custom ops (e.g., `sin`) apply to `.data` of CSR/CSC matrices.

---

### **5.7 Summary :**

- Understand **views** and **boolean arrays** for clean and efficient code.
- **Sparse matrices** are key in large problems, with dedicated types and methods in `scipy.sparse`.