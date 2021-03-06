.. _man-arrays:

**************************
 Multi-dimensional Arrays
**************************

Julia, like most technical computing languages, provides a first-class
array implementation. Most technical computing languages pay a lot of
attention to their array implementation at the expense of other
containers. Julia does not treat arrays in any special way. The array
library is implemented almost completely in Julia itself, and derives
its performance from the compiler, just like any other code written in
Julia.

An array is a collection of objects stored in a multi-dimensional
grid.  In the most general case, an array may contain objects of type
``Any``.  For most computational purposes, arrays should contain
objects of a more specific type, such as ``Float64`` or ``Int32``.

In general, unlike many other technical computing languages, Julia does
not expect programs to be written in a vectorized style for performance.
Julia's compiler uses type inference and generates optimized code for
scalar array indexing, allowing programs to be written in a style that
is convenient and readable, without sacrificing performance, and using
less memory at times.

In Julia, all arguments to functions are passed by reference. Some
technical computing languages pass arrays by value, and this is
convenient in many cases. In Julia, modifications made to input arrays
within a function will be visible in the parent function. The entire
Julia array library ensures that inputs are not modified by library
functions. User code, if it needs to exhibit similar behaviour, should
take care to create a copy of inputs that it may modify.

Arrays
======

Basic Functions
---------------

=============== ==============================================================================
Function        Description
=============== ==============================================================================
``eltype(A)``   the type of the elements contained in A
``length(A)``   the number of elements in A
``ndims(A)``    the number of dimensions of A
``size(A)``     a tuple containing the dimensions of A
``size(A,n)``   the size of A in a particular dimension
``stride(A,k)`` the stride (linear index distance between adjacent elements) along dimension k
``strides(A)``  a tuple of the strides in each dimension
=============== ==============================================================================

Construction and Initialization
-------------------------------

Many functions for constructing and initializing arrays are provided. In
the following list of such functions, calls with a ``dims...`` argument
can either take a single tuple of dimension sizes or a series of
dimension sizes passed as a variable number of arguments.

===================================== =====================================================================
Function                              Description
===================================== =====================================================================
``Array(type, dims...)``              an uninitialized dense array
``cell(dims...)``                     an uninitialized cell array (heterogeneous array)
``zeros(type, dims...)``              an array of all zeros of specified type
``ones(type, dims...)``               an array of all ones of specified type
``trues(dims...)``                    a ``Bool`` array with all values ``true``
``falses(dims...)``                   a ``Bool`` array with all values ``false``
``reshape(A, dims...)``               an array with the same data as the given array, but with
                                      different dimensions.
``copy(A)``                           copy ``A``
``deepcopy(A)``                       copy ``A``, recursively copying its elements
``similar(A, element_type, dims...)`` an uninitialized array of the same type as the given array
                                      (dense, sparse, etc.), but with the specified element type and
                                      dimensions. The second and third arguments are both optional,
                                      defaulting to the element type and dimensions of ``A`` if omitted.
``reinterpret(type, A)``              an array with the same binary data as the given array, but with the
                                      specified element type
``rand(dims)``                        ``Array`` of ``Float64``\ s with random, iid[#]_ and uniformly
                                      distributed values in [0,1)
``randn(dims)``                       ``Array`` of ``Float64``\ s with random, iid and standard normally
                                      distributed random values
``eye(n)``                            ``n``-by-``n`` identity matrix
``eye(m, n)``                         ``m``-by-``n`` identity matrix
``linspace(start, stop, n)``          vector of ``n`` linearly-spaced elements from ``start`` to ``stop``
``fill!(A, x)``                       fill the array ``A`` with value ``x``
===================================== =====================================================================

.. [#] *iid*, independently and identically distributed.

Comprehensions
--------------

Comprehensions provide a general and powerful way to construct arrays.
Comprehension syntax is similar to set construction notation in
mathematics::

    A = [ F(x,y,...) for x=rx, y=ry, ... ]

The meaning of this form is that ``F(x,y,...)`` is evaluated with the
variables ``x``, ``y``, etc. taking on each value in their given list of
values. Values can be specified as any iterable object, but will
commonly be ranges like ``1:n`` or ``2:(n-1)``, or explicit arrays of
values like ``[1.2, 3.4, 5.7]``. The result is an N-d dense array with
dimensions that are the concatenation of the dimensions of the variable
ranges ``rx``, ``ry``, etc. and each ``F(x,y,...)`` evaluation returns a
scalar.

The following example computes a weighted average of the current element
and its left and right neighbor along a 1-d grid. :

.. testsetup:: *

    srand(314)

.. doctest:: array-rand

    julia> const x = rand(8)
    8-element Array{Float64,1}:
     0.843025
     0.869052
     0.365105
     0.699456
     0.977653
     0.994953
     0.41084 
     0.809411

    julia> [ 0.25*x[i-1] + 0.5*x[i] + 0.25*x[i+1] for i=2:length(x)-1 ]
    6-element Array{Float64,1}:
     0.736559
     0.57468
     0.685417
     0.912429
     0.8446  
     0.656511

.. note:: In the above example, ``x`` is declared as constant because type
  inference in Julia does not work as well on non-constant global
  variables.

The resulting array type is inferred from the expression; in order to control
the type explicitly, the type can be prepended to the comprehension. For example,
in the above example we could have avoided declaring ``x`` as constant, and ensured
that the result is of type ``Float64`` by writing::

    Float64[ 0.25*x[i-1] + 0.5*x[i] + 0.25*x[i+1] for i=2:length(x)-1 ]

Using curly brackets instead of square brackets is a shorthand notation for an
array of type ``Any``:

.. doctest::

    julia> { i/2 for i = 1:3 }
    3-element Array{Any,1}:
     0.5
     1.0
     1.5

.. _man-array-indexing:

Indexing
--------

The general syntax for indexing into an n-dimensional array A is::

    X = A[I_1, I_2, ..., I_n]

where each I\_k may be:

1. A scalar value
2. A ``Range`` of the form ``:``, ``a:b``, or ``a:b:c``
3. An arbitrary integer vector, including the empty vector ``[]``
4. A boolean vector

The result X generally has dimensions
``(length(I_1), length(I_2), ..., length(I_n))``, with location
``(i_1, i_2, ..., i_n)`` of X containing the value
``A[I_1[i_1], I_2[i_2], ..., I_n[i_n]]``. Trailing dimensions indexed with
scalars are dropped. For example, the dimensions of ``A[I, 1]`` will be
``(length(I),)``. Boolean vectors are first transformed with ``find``; the size of
a dimension indexed by a boolean vector will be the number of true values in the vector.

Indexing syntax is equivalent to a call to ``getindex``::

    X = getindex(A, I_1, I_2, ..., I_n)

Example:

.. doctest::

    julia> x = reshape(1:16, 4, 4)
    4x4 Array{Int64,2}:
     1  5   9  13
     2  6  10  14
     3  7  11  15
     4  8  12  16

    julia> x[2:3, 2:end-1]
    2x2 Array{Int64,2}:
     6  10
     7  11

Assignment
----------

The general syntax for assigning values in an n-dimensional array A is::

    A[I_1, I_2, ..., I_n] = X

where each I\_k may be:

1. A scalar value
2. A ``Range`` of the form ``:``, ``a:b``, or ``a:b:c``
3. An arbitrary integer vector, including the empty vector ``[]``
4. A boolean vector

If ``X`` is an array, its size must be ``(length(I_1), length(I_2), ..., length(I_n))``,
and the value in location ``i_1, i_2, ..., i_n`` of ``A`` is overwritten with
the value ``X[I_1[i_1], I_2[i_2], ..., I_n[i_n]]``. If ``X`` is not an array, its
value is written to all referenced locations of ``A``.

A boolean vector used as an index behaves as in ``getindex`` (it is first transformed
with ``find``).

Index assignment syntax is equivalent to a call to ``setindex!``::

      setindex!(A, X, I_1, I_2, ..., I_n)

Example:

.. doctest::

    julia> x = reshape(1:9, 3, 3)
    3x3 Array{Int64,2}:
     1  4  7
     2  5  8
     3  6  9

    julia> x[1:2, 2:3] = -1
    -1

    julia> x
    3x3 Array{Int64,2}:
     1  -1  -1
     2  -1  -1
     3   6   9

Concatenation
-------------

Arrays can be concatenated along any dimension using the following
functions:

================ ======================================================
Function         Description
================ ======================================================
``cat(k, A...)`` concatenate input n-d arrays along the dimension ``k``
``vcat(A...)``   shorthand for ``cat(1, A...)``
``hcat(A...)``   shorthand for ``cat(2, A...)``
``hvcat(A...)``
================ ======================================================

Concatenation operators may also be used for concatenating arrays:

=================== =========
Expression          Calls
=================== =========
``[A B C ...]``     ``hcat``
``[A, B, C, ...]``  ``vcat``
``[A B; C D; ...]`` ``hvcat``
=================== =========

Vectorized Operators and Functions
----------------------------------

The following operators are supported for arrays. In case of binary operators,
the dot (element-wise) version of the operator should be used when both inputs
are non-scalar, and any version of the operator may be used if one of the
inputs is a scalar.

1.  Unary arithmetic — ``-``, ``+``, ``!``
2.  Binary arithmetic — ``+``, ``-``, ``*``, ``.*``, ``/``, ``./``,
    ``\``, ``.\``, ``^``, ``.^``, ``div``, ``mod``
3.  Comparison — ``==``, ``!=``, ``<``, ``<=``, ``>``, ``>=``
4.  Unary Boolean or bitwise — ``~``
5.  Binary Boolean or bitwise — ``&``, ``|``, ``$``

The following built-in functions are also vectorized, whereby the functions act
element-wise::

    abs abs2 angle cbrt
    airy airyai airyaiprime airybi airybiprime airyprime
    acos acosh asin asinh atan atan2 atanh
    acsc acsch asec asech acot acoth
    cos  cosh  sin  sinh  tan  tanh  sinc  cosc
    csc  csch  sec  sech  cot  coth
    acosd asind atand asecd acscd acotd
    cosd  sind  tand  secd  cscd  cotd
    besselh besseli besselj besselj0 besselj1 besselk bessely bessely0 bessely1
    exp  erf  erfc  erfinv erfcinv exp2  expm1
    beta dawson digamma erfcx erfi
    exponent eta zeta gamma
    hankelh1 hankelh2
     ceil  floor  round  trunc
    iceil ifloor iround itrunc
    isfinite isinf isnan
    lbeta lfact lgamma
    log log10 log1p log2
    copysign max min significand
    sqrt hypot

Furthermore, Julia provides the ``@vectorize_1arg`` and ``@vectorize_2arg``
macros to automatically vectorize any function of one or two arguments
respectively.  Each of these takes two arguments, namely the ``Type`` of
argument (which is usually chosen to be to be the most general possible) and
the name of the function to vectorize. Here is a simple example:

.. doctest::

    julia> square(x) = x^2
    square (generic function with 1 method)

    julia> @vectorize_1arg Number square
    square (generic function with 4 methods)

    julia> methods(square)
    # 4 methods for generic function "square":
    square{T<:Number}(x::AbstractArray{T<:Number,1}) at operators.jl:248
    square{T<:Number}(x::AbstractArray{T<:Number,2}) at operators.jl:249
    square{T<:Number}(x::AbstractArray{T<:Number,N}) at operators.jl:251
    square(x) at none:1

    julia> square([1 2 4; 5 6 7])
    2x3 Array{Int64,2}:
      1   4  16
     25  36  49

Broadcasting
------------

It is sometimes useful to perform element-by-element binary operations
on arrays of different sizes, such as adding a vector to each column
of a matrix.  An inefficient way to do this would be to replicate the
vector to the size of the matrix:

.. doctest::

    julia> a = rand(2,1); A = rand(2,3);

    julia> repmat(a,1,3)+A
    2x3 Float64 Array:
     0.848333  1.66714  1.3262
     1.26743   1.77988  1.13859

This is wasteful when dimensions get large, so Julia offers
``broadcast``, which expands singleton dimensions in
array arguments to match the corresponding dimension in the other
array without using extra memory, and applies the given
function elementwise:

.. doctest::

    julia> broadcast(+, a, A)
    2x3 Float64 Array:
     0.848333  1.66714  1.3262
     1.26743   1.77988  1.13859

    julia> b = rand(1,2)
    1x2 Float64 Array:
     0.629799  0.754948

    julia> broadcast(+, a, b)
    2x2 Float64 Array:
     1.31849  1.44364
     1.56107  1.68622

Elementwise operators such as ``.+`` and ``.*`` perform broadcasting if necessary. There is also a ``broadcast!`` function to specify an explicit destination, and ``broadcast_getindex`` and ``broadcast_setindex!`` that broadcast the indices before indexing.

Implementation
--------------

The base array type in Julia is the abstract type
``AbstractArray{T,n}``. It is parametrized by the number of dimensions
``n`` and the element type ``T``. ``AbstractVector`` and
``AbstractMatrix`` are aliases for the 1-d and 2-d cases. Operations on
``AbstractArray`` objects are defined using higher level operators and
functions, in a way that is independent of the underlying storage class.
These operations are guaranteed to work correctly as a fallback for any
specific array implementation.

The ``Array{T,n}`` type is a specific instance of ``AbstractArray``
where elements are stored in column-major order (see additional notes in
:ref:`man-performance-tips`). ``Vector`` and ``Matrix`` are aliases for
the 1-d and 2-d cases. Specific operations such as scalar indexing,
assignment, and a few other basic storage-specific operations are all
that have to be implemented for ``Array``, so that the rest of the array
library can be implemented in a generic manner for ``AbstractArray``.

``SubArray`` is a specialization of ``AbstractArray`` that performs
indexing by reference rather than by copying. A ``SubArray`` is created
with the ``sub`` function, which is called the same way as ``getindex`` (with
an array and a series of index arguments). The result of ``sub`` looks
the same as the result of ``getindex``, except the data is left in place.
``sub`` stores the input index vectors in a ``SubArray`` object, which
can later be used to index the original array indirectly.

``StridedVector`` and ``StridedMatrix`` are convenient aliases defined
to make it possible for Julia to call a wider range of BLAS and LAPACK
functions by passing them either ``Array`` or ``SubArray`` objects, and
thus saving inefficiencies from indexing and memory allocation.

The following example computes the QR decomposition of a small section
of a larger array, without creating any temporaries, and by calling the
appropriate LAPACK function with the right leading dimension size and
stride parameters.

.. doctest::

    julia> a = rand(10,10)
    10x10 Float64 Array:
     0.763921  0.884854   0.818783   0.519682   …  0.860332  0.882295   0.420202
     0.190079  0.235315   0.0669517  0.020172      0.902405  0.0024219  0.24984
     0.823817  0.0285394  0.390379   0.202234      0.516727  0.247442   0.308572
     0.566851  0.622764   0.0683611  0.372167      0.280587  0.227102   0.145647
     0.151173  0.179177   0.0510514  0.615746      0.322073  0.245435   0.976068
     0.534307  0.493124   0.796481   0.0314695  …  0.843201  0.53461    0.910584
     0.885078  0.891022   0.691548   0.547         0.727538  0.0218296  0.174351
     0.123628  0.833214   0.0224507  0.806369      0.80163   0.457005   0.226993
     0.362621  0.389317   0.702764   0.385856      0.155392  0.497805   0.430512
     0.504046  0.532631   0.477461   0.225632      0.919701  0.0453513  0.505329

    julia> b = sub(a, 2:2:8,2:2:4)
    4x2 SubArray of 10x10 Float64 Array:
     0.235315  0.020172
     0.622764  0.372167
     0.493124  0.0314695
     0.833214  0.806369

    julia> (q,r) = qr(b);

    julia> q
    4x2 Float64 Array:
     -0.200268   0.331205
     -0.530012   0.107555
     -0.41968    0.720129
     -0.709119  -0.600124

    julia> r
    2x2 Float64 Array:
     -1.175  -0.786311
      0.0    -0.414549

Sparse Matrices
===============

`Sparse matrices <http://en.wikipedia.org/wiki/Sparse_matrix>`_ are
matrices that contain enough zeros that storing them in a special data
structure leads to savings in space and execution time. Sparse
matrices may be used when operations on the sparse representation of a
matrix lead to considerable gains in either time or space when
compared to performing the same operations on a dense matrix.

Compressed Sparse Column (CSC) Storage
--------------------------------------

In Julia, sparse matrices are stored in the `Compressed Sparse Column
(CSC) format
<http://en.wikipedia.org/wiki/Sparse_matrix#Compressed_sparse_column_.28CSC_or_CCS.29>`_.
Julia sparse matrices have the type ``SparseMatrixCSC{Tv,Ti}``, where ``Tv``
is the type of the nonzero values, and ``Ti`` is the integer type for
storing column pointers and row indices.::

    type SparseMatrixCSC{Tv,Ti<:Integer} <: AbstractSparseMatrix{Tv,Ti}
        m::Int                  # Number of rows
        n::Int                  # Number of columns
        colptr::Vector{Ti}      # Column i is in colptr[i]:(colptr[i+1]-1)
        rowval::Vector{Ti}      # Row values of nonzeros
        nzval::Vector{Tv}       # Nonzero values
    end

The compressed sparse column storage makes it easy and quick to access
the elements in the column of a sparse matrix, whereas accessing the
sparse matrix by rows is considerably slower. Operations such as
insertion of nonzero values one at a time in the CSC structure tend to
be slow. This is because all elements of the sparse matrix that are
beyond the point of insertion have to be moved one place over.

All operations on sparse matrices are carefully implemented to exploit
the CSC data structure for performance, and to avoid expensive operations.

If you have data in CSC format from a different application or library, 
and wish to import it in Julia, make sure that you use 1-based indexing.
The row indices in every column need to be sorted. If your `SparseMatrixCSC` 
object contains unsorted row indices, one quick way to sort them is by
doing a double transpose.

In some applications, it is convenient to store explicit zero values in 
a `SparseMatrixCSC`. These *are* accepted by functions in ``Base`` (but
there is no guarantee that they will be preserved in mutating operations).
Because of this, ``countnz`` is not a constant-time operation; instead,
``nfilled`` should be used to obtain the number of elements in a sparse
matrix.

Sparse matrix constructors
--------------------------

The simplest way to create sparse matrices are using functions
equivalent to the ``zeros`` and ``eye`` functions that Julia provides
for working with dense matrices. To produce sparse matrices instead,
you can use the same names with an ``sp`` prefix:

.. doctest::

    julia> spzeros(3,5)
    3x5 sparse matrix with 0 Float64 entries:

    julia> speye(3,5)
    3x5 sparse matrix with 3 Float64 entries:
            [1, 1]  =  1.0
            [2, 2]  =  1.0
            [3, 3]  =  1.0

The ``sparse`` function is often a handy way to construct sparse
matrices. It takes as its input a vector ``I`` of row indices, a
vector ``J`` of column indices, and a vector ``V`` of nonzero
values. ``sparse(I,J,V)`` constructs a sparse matrix such that
``S[I[k], J[k]] = V[k]``.

.. doctest::

    julia> I = [1, 4, 3, 5]; J = [4, 7, 18, 9]; V = [1, 2, -5, 3];

    julia> S = sparse(I,J,V)
    5x18 sparse matrix with 4 Int64 entries:
            [1 ,  4]  =  1
            [4 ,  7]  =  2
            [5 ,  9]  =  3
            [3 , 18]  =  -5

The inverse of the ``sparse`` function is ``findn``, which
retrieves the inputs used to create the sparse matrix.

.. doctest::

    julia> findn(S)
    ([1,4,5,3],[4,7,9,18])

    julia> findnz(S)
    ([1,4,5,3],[4,7,9,18],[1,2,3,-5])

Another way to create sparse matrices is to convert a dense matrix
into a sparse matrix using the ``sparse`` function:

.. doctest::

    julia> sparse(eye(5))
    5x5 sparse matrix with 5 Float64 entries:
            [1, 1]  =  1.0
            [2, 2]  =  1.0
            [3, 3]  =  1.0
            [4, 4]  =  1.0
            [5, 5]  =  1.0

You can go in the other direction using the ``dense`` or the ``full``
function. The ``issparse`` function can be used to query if a matrix
is sparse.

.. doctest::

    julia> issparse(speye(5))
    true

Sparse matrix operations
------------------------

Arithmetic operations on sparse matrices also work as they do on dense
matrices. Indexing of, assignment into, and concatenation of sparse
matrices work in the same way as dense matrices. Indexing operations,
especially assignment, are expensive, when carried out one element at
a time. In many cases it may be better to convert the sparse matrix
into ``(I,J,V)`` format using ``findnz``, manipulate the non-zeroes or
the structure in the dense vectors ``(I,J,V)``, and then reconstruct
the sparse matrix.

Correspondence of dense and sparse methods
------------------------------------------
The following table gives a correspondence between built-in methods on sparse
matrices and their corresponding methods on dense matrix types. In general,
methods that generate sparse matrices differ from their dense counterparts in
that the resulting matrix follows the same sparsity pattern as a given sparse
matrix ``S``, or that the resulting sparse matrix has density ``d``, i.e. each
matrix element has a probability ``d`` of being non-zero.

Details can be found in the :ref:`stdlib-sparse` section of the standard library
reference.

.. tabularcolumns:: |l|l|L|

+-----------------------+-------------------+----------------------------------------+
| Sparse                | Dense             | Description                            |
+-----------------------+-------------------+----------------------------------------+
| ``spzeros(m,n)``      | ``zeros(m,n)``    | Creates a *m*-by-*n* matrix of zeros.  |
|                       |                   | (``spzeros(m,n)`` is empty.)           |
+-----------------------+-------------------+----------------------------------------+
| ``spones(S)``         | ``ones(m,n)``     | Creates a matrix filled with ones.     |
|                       |                   | Unlike the dense version, ``spones``   |
|                       |                   | has the same sparsity pattern as *S*.  |
+-----------------------+-------------------+----------------------------------------+
| ``speye(n)``          | ``eye(n)``        | Creates a *n*-by-*n* identity matrix.  |
+-----------------------+-------------------+----------------------------------------+
| ``full(S)``           | ``sparse(A)``     | Interconverts between dense            |
|                       |                   | and sparse formats.                    |
+-----------------------+-------------------+----------------------------------------+
| ``sprand(m,n,d)``     | ``rand(m,n)``     | Creates a *m*-by-*n* random matrix (of |
|                       |                   | density *d*) with iid non-zero elements|
|                       |                   | distributed uniformly on the           |
|                       |                   | interval [0, 1].                       |
+-----------------------+-------------------+----------------------------------------+
| ``sprandn(m,n,d)``    | ``randn(m,n)``    | Creates a *m*-by-*n* random matrix (of |
|                       |                   | density *d*) with iid non-zero elements|
|                       |                   | distributed according to the standard  |
|                       |                   | normal (Gaussian) distribution.        |
+-----------------------+-------------------+----------------------------------------+
| ``sprandn(m,n,d,X)``  | ``randn(m,n,X)``  | Creates a *m*-by-*n* random matrix (of |
|                       |                   | density *d*) with iid non-zero elements|
|                       |                   | distributed according to the *X*       |
|                       |                   | distribution. (Requires the            |
|                       |                   | ``Distributions`` package.)            |
+-----------------------+-------------------+----------------------------------------+
| ``sprandbool(m,n,d)`` | ``randbool(m,n)`` | Creates a *m*-by-*n* random matrix (of |
|                       |                   | density *d*) with non-zero ``Bool``    |
|                       |                   | elements with probability *d* (*d* =0.5|
|                       |                   | for ``randbool``.)                     |
+-----------------------+-------------------+----------------------------------------+


