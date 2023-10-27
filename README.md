# Taichi Lang

- Taichi is statically typed.
- Index in Taichi starts from 0 (same as python and C++ etc.)
- There is no pointer in Taichi.

## Installation

Currently I will only show how I installed Taichi in my system.

I'm on Windows 10, with python 3.10 installed, type the following in PowerShell:

```
python3.10 -m pip install --upgrade taichi
```

We can now `import` taichi in our python 3.10 program.

Note that the version of Taichi I was using when writing this notes is <ins>1.6.0</ins>.

## Keywords to Study:

- Differentiable Simulation
- Spatially sparse
- Memory footprint
- Load balance
- JIT compilation
- Cellular automaton
- Diffusion-limited aggregation

## Miscellaneous Python-related Questions

- [What does `if __name__ == "__main__":` do?](https://stackoverflow.com/a/419185/17172007)

  Note that both `import X` and `from X import ...` execute codes in file `X.py`.

- [The `pass` keyword](https://www.w3schools.com/python/ref_keyword_pass.asp)

- [We can actually call sub-class method in base-class](https://stackoverflow.com/a/9126240/17172007)

- [In python, no include guards is necessary](https://stackoverflow.com/a/2950584/17172007)

## `ti.init()`

Always initialize Taichi with `ti.init()` before you do any Taichi operations.

```python
import taichi as ti

ti.init(arch=ti.gpu)  # if none of the gpu architecture is detected
                      # this will automatically falls back on cpu architectures.
```

This will specify the backend (i.e. the hardware architecture) for Taichi to run on.

## Common Data Types

- 32 bits integer: `ti.i32`
- 32 bits unsigned integer: `ti.u32`
- 32 bits floating point number: `ti.f32`

## Type Casts

```python
@ti.kernel
def f():
    a = 1
    b = ti.cast(a, ti.f32)
    print(b)    # 1.000000
```

## Compound Types

```python
import taichi as ti
ti.init(arch=ti.gpu)

vec3f = ti.types.vector(3, ti.f32)
mat2f = ti.types.matrix(2,2, ti.f32)
ray = ti.types.struct(ori=vec3f, dir=vec3f, vision=ti.f32)

@ti.kernel
def f():
    a = vec3f(0.0)
    print(a)    # [0.000000, 0.000000, 0.000000]

    b = vec3f(0,1,0)
    print(b)    # [0.000000, 1.000000, 0.000000]

    M = mat2f([[1,2],[3,4]])    # [[1.000000, 2.000000], [3.000000, 4.000000]]
    print(M)

    r = ray(a,b,10)
    print(r.dir)    # [0.000000, 1.000000, 0.000000]

# Similarly:

@ti.kernel
def f():
    a = ti.Vector([0.0,0,0])
    print(a)    # [0.000000, 0.000000, 0.000000]

    b = ti.Vector([0,1,0])
    print(b)    # [0, 1, 0]
    print(b[1])    # 1

    M = ti.Matrix([[1,2],[3.0, 4]])    # [[1.000000, 2.000000], [3.000000, 4.000000]]
    print(M)
    print(M[1,0])    # 3.000000

    r = ti.Struct(ori=a,dir=b,vision=10.0)
    print(r.vision)    # 10.000000
```

Note:

- Use `ti.Matrix` only for small matrices, use tensor for large matrices.

  E.g. Use `ti.Matrix.field(3, 2, dtype=ti.f32, shape=(64, 32))` instead of `ti.Matrix.field(64, 32, dtype=ti.f32, shape=(3, 2))`

- `*` is for element-wise multiplication, `@` is for matrix multiplication.

## Field

Note that we cannot allocate field in taichi scope.

Field in Taichi is like a multi-dimensional array.

Elements in a taichi field can be scalar, vector, matrix or struct.

Those elements can be accessed via `[i,j,..]` syntax.

Fields are global: they can be read/written from python scope and taichi scope.

- Scalar field:

```python
f1 = ti.field(dtype=ti.f32, shape=())   # 0 dimensional scalar field (i.e. a ti.f32 scalar)
f2 = ti.field(dtype=ti.f32, shape=4)   # 1 dimensional scalar field of 4 ti.f32 scalars
f3 = ti.field(dtype=ti.f32, shape=(10,20))   # 2 dimensional scalar field of 10*20 ti.f32 scalars

vf = ti.Vector.field(3, dtype=ti.f32, shape=(2,3))    # 2-d vector field of 2*3 3-d vectors of type ti.f32

mf = ti.Matrix.field(3,2, dtype=ti.f32, shape=(4,5))    # 2-d field of 4*5 matrices of type ti.f32 with 3 rows and 2 columns

f1[None] = 1.0    # accessing the scalar in the 0-d scalar field
vf[1,2][1] = 2.0    # acessing the 2nd element of the 3-d vector located at row 2, column 3 of the 2-d vector field
vf[0,1] = [6,8,10]    # direct vector assignment
mf[1,2][1,0] = 3.0    # acessing the element at row 2, column 1 of the 3*2 matrix located at row 2, column 3 of the 2-d matrix (tensor) field
```

## GUI

As mentioned, the `shape=(x,y)` given to a taichi field means `x` rows and `y` column.

We can set the field as the image output to the taichi gui of the same dimension. In such cases, the field is map to the screen as if the screen has a conventional cartesian coordinates with origin at bottom left corner (i.e. `x` rows map to the horizontal `x`-axis increasing from left to right; `y` columns map to the vertical `y`-axis increasing from bottom to top).

The following example code shows this:

```python
import taichi as ti
ti.init(arch=ti.gpu)

x, y = 640,180
image = ti.field(dtype=ti.f32, shape=(x, y))

@ti.kernel
def fill_image():
    for i,j in image:
        if i < 200 and j < 80:
            image[i,j] = 0
        else:
            image[i,j] = 1

fill_image()
gui = ti.GUI('canvas', (x, y))
while gui.running:
    gui.set_image(image)
    gui.show()
```

## GGUI



## Kernel and func

Both `@ti.kernel` and `@ti.func` can have at most <ins>one</ins> `return` statement.

`@ti.func` cannot be called directly from python scope.

`@ti.func` can be called from `@ti.kernel` and other `@ti.func`.

`@ti.kernel` cannot be called from `@ti.func` or `@ti.kernel`, it will be called from python scope.

Since, by the time of writing, `@ti.func` is force-inlined, it can not be recursive.

Recursion is also not allowed in `@ti.kernel`.

- Example:

```python
@ti.kernel
def fun(x: ti.f32) -> ti.i32:
    return x
```

---

❓ In the following code:

```python
import taichi as ti
ti.init(arch=ti.gpu)

a = 42

@ti.kernel
def f():
    a = 1
    print(a)
```

Is the `a` in `f()` a separately defined variable which overwrites the identifier of the `a` defined in python scope, or is `a = 1` just overwritting the content of the variable created when pass `a` to the taichi scope by value? Hence, if `a = 1` is commented out, is `a` passed by (const) reference to taichi scope or are we having an overhead of copying the value?

---

The following shows why we want global taichi 0-d scalar field:

```python
import taichi as ti
ti.init(arch=ti.gpu)

a = 42

@ti.kernel
def f():
    print(a)

f()    # 42
a = 43
f()    # 42

b = ti.field(dtype=ti.i32, shape=())

@ti.kernel
def g():
    print(b[None])

b[None] = 42
g()    # 42
b[None] = 43
g()    # 43
```

❓ Explain how does the taichi JIT compiler deal with this internally.

---

Taichi supports chaining comparison.

E.g.

```python
5 > 4 >= 3 != 5    # True
```

---

## Range `for` loop

```python
@ti.kernel
def f():
    for i in range(3):    # parallelized
        print(i)
        for j in range(3):    # serialized
            print(i,j)
    # serialized between for loops
    for i,j,k in ti.ndrange((3,8),(1,6),9):    # parallelized, 3 <= i < 8, 1 <= j < 6, 0 <= k < 9
        print(i,j,k)

@ti.kernel
def g(p: ti.i32):
    if p > 0:
        for i in range(3):    # serialized
            print(i)
```

Note that for the last example, if the condition `if`-statement is known at the compile-time (and hence is optimized away by the compiler. E.g., `if p == p:`), then the following `for`-loop becomes the outermost loop (recall that `ti.func` is force-inlined), and thus it will be parallelized.

Note that `break` cannot be used for the parallelized for loop.

## Struct `for` loop

```python
import taichi as ti
ti.init(arch=ti.gpu)

n = 320
pixels = ti.field(ti.i32, shape=(n*2, n))

@ti.kernel
def paint():
    for i,j in pixels:    # parallelized
        pixels[i,j] = i + j
```

## Atomic Operation

In Taichi, augmented assignment (e.g., `+=`) are made to be atomic operation.

```python
import taichi as ti
ti.init(arch=ti.gpu)

s = ti.field(ti.i32, shape=())
t = ti.field(ti.i32, shape=())

@ti.kernel
def sum():
    for i in range(5):    # parallelized
        s[None] += i    # sum correctly
        t[None] = t[None] + i    # data race
```

---

Case Study:

```python
# case 1:
for i in range(5):
        p = pos[i]
        for j in range(i):
            diff = p - pos[j]                     # 1
            r = diff.norm(1e-5)                   # 0.5
            f = -G * m * m * (1.0 / r)**3 * diff  # 0.5
            force[i] += f                         # 5
            force[j] += -f                        # 5

# case 2:
for i in range(5):
        p = pos[i]
        for j in range(5):
            if i != j:
                diff = p - pos[j]                     # 1
                r = diff.norm(1e-5)                   # 0.5
                f = -G * m * m * (1.0 / r)**3 * diff  # 0.5
                force[i] += f                         # 5
```

Shown in the comments are the amount of computation taken for each line.

The total computation to be done for `case 1` and `case 2` are 120 and 140 respectively.

Nevertheless, if the outermost `for` loops are executed in parallel, the computation for each thread (from thread 1 to 5) are 0, 12, 24, 36, 48 respectively for `case 1`, and are 28, 28, 28, 28, 28 respectively for `case 2`.

Hence, `case 1` is better for single thread CPU execution, while `case 2` is better for GPU execution.

❓ How about in case where the number of total iterations is much larger than the number of total threads available?

## Debug

❗ The version of Taichi that I am currently using (1.6.0) seems to have bugs with the taichi scope `assert` statement working on GPU.

E.g. The only output of the following code, currently revealing on my machine, is `0`

```python
import taichi as ti
ti.init(arch=ti.cuda, debug=False)

@ti.kernel
def f(a: ti.i32, b: ti.i32) -> ti.i32:
    assert a != b, f"a and b are same"
    print('No debugging')
    c = 42
    return c

d = f(1,1)

print(d)
```

## Metaprogramming

### To write dimensionality-independent code

- `ti.template()` marks the type of the parameter so that we can pass any Taichi-things to the parameter.
- The parameter of type `ti.template()` will be <ins>passed by reference</ins>.

  - We can not modify data of python scope in taichi scope if it is not in a taichi field, because it will be passed into the taichi scope as a constant.
  - We can modify data of taichi scope in taichi scope (e.g., data defined in `ti.kernel` passed as `ti.template()` to `ti.func` is passed by ~~const~~ reference).

- Once the type of a parameter is marked by `ti.template()`, the function will be defined as a Taichi template function.
- A Taichi template function will be instantiated if and only if a call to the function with a new parameter (even it is same typed as the previous one) is made.

- `ti.grouped(f)` takes a taichi field `f` and returns a data structure that holds all the `ti.Vector`s representing the indices of `f` that traverse `f`.

  We can extract those taichi vectors using the `for i in ti.grouped(f):` syntax.

  Note that we can do indexing directly using taichi vector (like `f[v]` where `v` is a `ti.Vector`).

  Note that if `f` is defined as `shape=()`, the `i` will be `ti.Vector([])`.

  Using `ti.Vector([])` to do indexing is equivalent to do `[None]`.

Example:

```python
import taichi as ti
ti.init(arch=ti.gpu)

@ti.kernel
def copy(src: ti.template(), dst: ti.template()):
    for i in ti.grouped(src):
        dst[i] = src[i]

a = ti.field(ti.f32, 4)
b = ti.field(ti.f32, 4)
c = ti.Vector.field(3, ti.f32, shape=(3,5))
d = ti.Vector.field(3, ti.f32, shape=(3,5))

copy(a,b)
copy(c,d)
```

- Accessing metadata:

```python
import taichi as ti
ti.init(arch=ti.gpu)

@ti.kernel
def f(src: ti.template(), dst: ti.template()):
    if src.dtype == dst.dtype:
        print("Same type")
    else:
        print("Different types")
    
    if src.shape == dst.shape:
        print("Same shape")
    else:
        print("Different shapes")

a = ti.field(ti.i32, 4)
b = ti.field(ti.f32, 4)
c = ti.Vector.field(3, ti.f32, shape=(3,5))

f(a,b)    # Different types
          # Same shape
f(b,c)    # Same type
          # Different shapes
f(a,c)    # Different types
          # Different shapes

d = ti.Vector([1,2,3,4])
e = ti.Matrix([[1,2],[3,4],[5,6]])

print(d.n)    # 4 (rows)
print(d.m)    # 1 (column)
print(e.n)    # 3 (rows)
print(e.m)    # 2 (columns)
```

### To improve runtime performance

`ti.static()` in Taichi is to some extent similar to `constexpr` in C++.

The `if`-statement `if ti.static(x):`, when it is legal, will eliminate the overhead of branching.

The `for`-loop `for i in ti.static(range(x)):`, when it is legal, will unroll the loop and then execute the contents serially.

## Object Oriented

