# Contents

- [Features to Note](#Features-to-Note)
- [Installation](#Installation)
- [Keywords to Study](#Keywords-to-Study)
- [Miscellaneous Python-related Knowledge](#Miscellaneous-Python-related-Knowledge)
- [What is Data-Oriented](#What-is-Data-Oriented)
- [`ti.init()`](#tiinit)
- [Common Data Types](#Common-Data-Types)
- [Type Casts](#Type-Casts)
- [Compound Types](#Compound-Types)
- [Field](#Field)
- [GUI](#GUI)
- [GGUI](#GGUI)
- [Kernel and func](#Kernel-and-func)
- [Range `for` loop](#Range-for-loop)
- [Struct `for` loop](#Struct-for-loop)
- [Atomic Operation](#Atomic-Operation)
- [Debug](#Debug)
- [Metaprogramming](#Metaprogramming)
- [Object Oriented](#Object-Oriented)
- [Dense Data Layout](#Dense-Data-Layout)
  - [SOA in Taichi](#SOA-in-Taichi)
  - [AOS in Taichi](#AOS-in-Taichi)
- [Sparse Data Layout](#Sparse-Data-Layout)
- [Sparse Matrix](#)
- [Debugging](#)
- [Code Optimization](#)
- To be continued...

## Features to Note

- Taichi is statically typed.
- Index in Taichi starts from 0 (same as python and C++ etc.)
- There is no (naked) pointer in Taichi.
- Taichi is designed to have optimized performance with data-oriented programming.

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
- Interpreted language
- Bezier curve
- Marching squares/cubes

## Miscellaneous Python-related Knowledge

- [What does `if __name__ == "__main__":` do?](https://stackoverflow.com/a/419185/17172007)

  Note that both `import X` and `from X import ...` execute codes in file `X.py`.

- [The `pass` keyword](https://www.w3schools.com/python/ref_keyword_pass.asp)

- [We can actually call sub-class method in base-class](https://stackoverflow.com/a/9126240/17172007)

- [In python, no include guards is necessary](https://stackoverflow.com/a/2950584/17172007)

- [In Python, everything is an object](https://stackoverflow.com/a/44861529/17172007)

## What is Data-Oriented

```cpp
struct AOS
{
    int a;
    int b;
};

struct SOA
{
    int a[5];
    int b[5];
};

int main()
{
    // all the arrays on stack
    AOS aos[5];   // Array of Structures
    SOA soa;        // Structure of Arrays
}
```

SOA is more data-oriented than AOS.

Note that both `aos` and `soa` takes up equal amount of space on the memory. But they have different memory layouts for `a` and `b`:

```
ababababab    // aos
aaaaabbbbb    // soa
```

## `ti.init()`

Always initialize Taichi with `ti.init()` before you do any Taichi operations.

```python
import taichi as ti

ti.init(arch=ti.gpu)  # if none of the gpu architecture is detected
                      # this will automatically falls back on cpu architectures.
```

This will specify the backend (i.e. the hardware architecture) for Taichi to run on.

❓ What does it mean to a Taichi program if I do `ti.init` multple places?

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

---

In my current understanding:

- Once we have created a taichi gui object, running the taichi program will create a gui window.
- The reaction for the Window buttoms (on the top right corner) is dealed within `gui.show()`.
- What is currently shown stays (probably in some buffer `A`) until the next call of `gui.show()`.
- There is probably some other buffer `B` where we can draw the next frame. Calling `gui.show()` will take data from buffer `B` to buffer `A`, and then clear buffer `B`.

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

  ❓ I believe, in Python, if I write `a = 1` and then `a = 2`, the address referring by `a` changes. Will change in address result in new instantiation of taichi template function?

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

To decorate python class method as `ti.func` or `ti.kernel`, we need to first decorate the python class with `@ti.data_oriented`.

Note that a python class decorated with `@ti.data_oriented` is still in python scope. That is, the methods defined in the class are still in python scope unless decorated with `@ti.func` or `@ti.kernel`.

Hence, we can define taichi fields within any python-scope methods (including the `__init__` function).

- Note that, in my current understanding, as taichi's compiler uses JIT compilation, only when we instantiate a class will the taichi compiler starts to compile the machine code for allocating the taichi field data members of the class. If so, this explain why we don't need `ti.init` for a `.py` source file that only contains the definitions of classes.

Note also that, since data members are defined in python scope, taichi's JIT compiler will pass data members of python's types as constant to `ti.kernel` and `ti.func`.

## Dense Data Layout

Possible References: [Taichi Docs](https://docs.taichi-lang.org/docs/layout#organizing-an-efficient-data-layout)

- First of all, <ins>note</ins> that we can use `ti.get_addr(field, indices)` for checking memory adjacency (currently it can only be called within `@ti.kernel`).

- Since, normally, we gain much more computational power running under a parallel framework (e.g. GPU) than a serial framework (e.g. single thread CPU), we often want to focus on getting better memory access rather than reducing computations when dealing with parallel programming.

- The rule of thumb for better memory access is to <ins>align the order of memory access with the memory layout (i.e. the order for which the data is storage in main memory)</ins>

  This is because, each time when information is taken from main memory to the cache, a block of information is taken. That is, all the information local to the piece of information that is aimed to be taken specifically by that fetch will probably be sent into the cache. Hence, if the information wanted by the next fetch is local to the information taken by the last fetch, data access will have a chance to be done within the cache, which is much faster than accessing the main memory. This is why we want <ins>locality</ins>.

❓ Explain in details on how locality improves performance during multithreading on CPU, and on GPU.

In Taichi, fields are stored in memory as SNodeTree (Structural Node Tree).

`ti.root` defines the root of a SNodeTree. Following a `.dense()` defines the shape of a layer (if there is any) on the tree. Following `.place()` defines the type of the data element of the tree.

The last `.dense()` in a `ti.root` statement indicates the adjacent data elements in memory.

❓ How exactly does a SNodeTree lie in memory?

❓ Are all leaf layers for a field be adjacent in memory?

The default layout for a taichi field (i.e. define field directly, without using `ti.root`) is row-major.

We can use struct-`for` loop to traverse a taichi field (as usual) constructed from `ti.root`. The taichi compiler will automatically deduce the underlying data layout of a taichi field and apply the same order for data access.

Examples:

```python
x = ti.Vector.field(3, ti.i32, shape=())
# is equivalent to:
x = ti.Vector.field(3, ti.i32)
ti.root.place(x)    # this reveals that x is 0-d

x = ti.Vector.field(3, ti.i32, shape=8)
# is equivalent to:
x = ti.Vector.field(3, ti.i32)
ti.root.dense(ti.i, 8).place(x)
# ti.i means the layer added to the SNodeTree
# is a row (ti.j means the same for column)

x = ti.field(ti.i32, shape=(2,3))
# is equivalent to:
x = ti.field(ti.i32)
ti.root.dense(ti.ij, (2,3)).place(x)
# ti.ij means the layer added to the SNodeTree
# is a row-major 2-d array

x = ti.field(ti.i32)
ti.root.dense(ti.ij, (2,2)).dense(ti.ij, (3,3)).place(x)    # block-major
# store data in 4 blocks each has 9 cells
# order between each block is row-major
# order between each cell is row-major
# note that we still use two variables (i,j) to access this field in a struct-for loop
```

❓ Explain how Taichi executes the following code:

```python
import taichi as ti
ti.init(arch=ti.gpu)

a = ti.field(ti.i32)
ti.root.dense(ti.i, 2).dense(ti.i, 2).dense(ti.i, 2).place(a)

@ti.kernel
def f():
    for i in a:
        print(i, end=' ')
        j = 0
        while j < 1000:
            j += 1
        print(j, end=' ')
        print('a', end=' ')
f()

# my 1st execution: 6 7 4 5 2 3 0 1 1000 1000 1000 1000 1000 1000 1000 1000 a a a a a a a a 
# my 2nd execution: 4 5 6 7 0 1 2 3 1000 1000 1000 1000 1000 1000 1000 1000 a a a a a a a a 
```

- In my current understanding: (those are just conceptual guesses!!)
  - If the root of a field is connected directly to the leaf layer, then, taichi will use a single thread to traverse that leaf layer if the number of cells in that leaf layer is small, or use multiple threads partitioning the cells.
  - If the root of a field is connected to a layer which is not leaf layer, then taichi will try to allocate enough number of threads to (ideally) run all the paths that lead to a leaf layer in parallel, and if the number of cells in a leaf layer is large, taichi will use more threads (if available) to partition that leaf layer.
  - For each thread, it seems that the loop is further optimized. By my guess, taichi optimize the first print statement for all the iterations that are governed by each thread into one print statement for each thread (same is done to the third print statement later on).
  - It seems that each thread only starts dealing with the third print statement after the thread has done with the second print statement for all iterations the thread governs. This may be a strategy to "shrink" the `for` loop by keeping what are not necessary looping multiple times outside the loop.

### SOA in Taichi

In my current understanding, `ti.root` create new SNodeTree (and does destructions) in an automatic way.

```python
import taichi as ti
ti.init(arch=ti.gpu)

x = ti.field(ti.i32)
y = ti.field(ti.i32)

ti.root.dense(ti.i, 2).place(x)
ti.root.dense(ti.i, 2).place(y)
```

The above code creates one SNodeTree. The upper `ti.root` first find the root of this tree, the `.dense` following it attachs cells to the tail of whatever that has already been attached to the calling part (which is `ti.root` in this case). In this case, nothing has been attached to `to.root` yet, so the `.dense` following the `ti.root` is the "head" of its attachments. `.place(x)` makes the cells of its calling part (which is `ti.dense(ti.i, 2)`) become data containers for type `x`. Then, the lower `ti.root.dense()` does the same so now it attachs more cells to the tail of the cells created by the first `.dense`, and then makes the cells newly attached into data containers for type `y`. Hence, we get a memory layout as `xxyy`.

If we do something as follows:

```python
import taichi as ti
ti.init(arch=ti.gpu)

x = ti.field(ti.i32)
y = ti.field(ti.i32)

a = ti.root.dense(ti.i, 2)

a.place(x)
a.place(y)
```

what it does is find the root of the tree, do `.dense` as described above, then for each cell this `.dense` attached, make a data container for type `x`, then, for each cell this `.dense` attached, make a data container for type `y`. I think, in some way behind the scenes, `.place` is similar to `.dense` such that the data container for type `y` is created to follow the data container for type `x` that is already exists (i.e. they both behave as "attaching to the tails"). Hence, we get a memory layout as `xyxy`.

If we do something as follows:

```python
import taichi as ti
ti.init(arch=ti.gpu)

x = ti.field(ti.i32)
y = ti.field(ti.i32)

ti.root.dense(ti.i, 2).dense(ti.i, 2).place(x)
ti.root.dense(ti.i, 2).dense(ti.i, 2).place(y)
```

it finds the root of the tree, do the first `.dense` as described above. I think, in such cases, the memory block regarding this first `.dense` contains the memory block regarding the second `.dense`. The same executions applies for the remaining code. Hence, we get a memory layout as `xxxxyyyy`.

Hence, to get a memory layout as `xxyyxxyy`, we want to do:

```python
import taichi as ti
ti.init(arch=ti.gpu)

x = ti.field(ti.i32)
y = ti.field(ti.i32)

a = ti.root.dense(ti.i, 2)  # make a row, let's call it A

a.dense(ti.i, 2).place(x)  # for each cell in row A, attach (to the tail) a row of x
a.dense(ti.i, 2).place(y)  # for each cell in row A, attach (to the tail) a row of y
```

We can manually allocate/destruct field as follows:

```python
import taichi as ti
ti.init(arch=ti.gpu)

@ti.kernel
def func(v: ti.template()):
    for I in ti.grouped(v):
        print(I)

fb = ti.FieldsBuilder()
y = ti.field(dtype=ti.f32)
fb.dense(ti.i, 5).place(y)
x = ti.field(dtype=ti.i32)
fb.dense(ti.i, 10).place(x)
fb_snode_tree = fb.finalize()  # Finalizes the FieldsBuilder and returns a SNodeTree
func(y)
func(x)
fb_snode_tree.destroy()  # Destruction
```

In my current understanding, `ti.root` encapsulates the same process as above to automate the creation/destruction of SNodeTree. More specifically, field constructed by `ti.root` also needs to be finalized, and it is done when we first access the field. E.g.

```python
import taichi as ti
ti.init(arch=ti.gpu)

x = ti.field(ti.i32)
a = ti.root.dense(ti.i, 2)
a.dense(ti.i, 2).place(x)

x[0] = 1 # finalizing x
         # same if this was called in a @ti.kernel

# Doing the following will cause error, since x has been finalized:
y = ti.field(ti.i32)
a.dense(ti.i, 2).place(y)
```

### AOS in Taichi

```python
x = ti.field(ti.i32)
y = ti.Vector.field(3, ti.f32)
ti.root.dense(ti.i, 5).place(x, y)
# In memory: xyxyxyxyxy
```

## Sparse Data Layout

Apart from `.dense` block, we can also attach pointers block to a SNodeTree by using `.pointer()`. E.g.

```python
import taichi as ti
ti.init(arch=ti.gpu)

x = ti.field(ti.i32)

a = ti.root.pointer(ti.i, 2)
b = a.dense(ti.j, 2)
b.place(x)

x[0,0] = 1

@ti.kernel
def f():
    for i,j in x:
        print(i, j, ':', x[i,j])

f()  # 0 0 : 1
     # 0 1 : 1
```

In the above code, when the field `x` is first created (before the assignment `x[0,0] = 1`), the two pointer nodes in the SNodeTree are defaultly set to be inactive (in my current understanding, it means that they are pointing to `nullptr`, i.e. `000...`, though I am not 100% sure on this).

Assigning an element of a field will (only) activate all the parent nodes (which are activatable. E.g. pointer/hash/bitmasked nodes) containing the element.

Assigning an element will allocate the whole contiguous memory in which the element is sit, up to where an activatable node in the SNodeTree is found (because the pointer behind this activatable node solely controls the activity of this whole dense block of contiguous memory). Nevertheless, note that it is totally legal for an active node to have all of its activatable sub-nodes being inactive.

We can also attach `.bitmasked()` node to a SNodeTree. Behind the scenes `.bitmasked()` block is same as `.dense()` block. But the SNodeTree uses 1 extra bit for each bitmasked cell to flag it as active/inactive, so that each bitmasked cell can be activated/deactivated independently.

A struct-`for` statement looping through a field will escape all nodes that are inactive.



### Deactivation

At the time of writing, the taichi's implementation of deactivation is quite messy to me (or maybe it is my brain that is messy at this point). In my current understandings:

- `.bitmasked` node does not do any memory recycling with respect to its activity (no matter if you are using `ti.deactivate()` or `.deactivate_all()` on it). So, whenever we want to recycle memory by deactivation, we may want to do it on `.pointer` node.

```python

```

- You seems to be able to access the data held by a `.bitmasked` node if you explicitly do so, even after you deactivate it. This may be a feature but I tend to think of it as a bug, because if a node is inactive, why bother reading it.

```python

```

- Deactivate `.pointer` node seems to recycle the memory for all of its downstream structure. But I am not 100% percent sure of this (because the [Docs](https://docs.taichi-lang.org/docs/sparse#3-deactivation) says "`ti.deactivate` does not recursively deactivate all the descendants of a cell."), so, prefer to use `.deactivate_all()` on `.pointer` node when it is feasible.

```python

```

- Even when you do deactivation on `.pointer` node, it seems under some circumstances you can still access it. So, prototype your sparse data structure and test it before you do any serious project. One advice that I think may help is to divide what you want to do into separate `ti.kernel`s (do one thing a time).

```python
import taichi as ti
ti.init(arch=ti.gpu)

a = ti.field(dtype=ti.i32)
block = ti.root.pointer(ti.i,2)
block.place(a)


@ti.kernel
def test():
    a[0] = 5
    ti.deactivate(block,[0])
    print(a[0] , ti.is_active(block,[0])) # 5 0




@ti.kernel
def pr():
    print(a[0], ti.is_active(block,[0])) # 0 0

@ti.kernel
def loo():
    for i in a:
        print(i)

test()
pr()
loo()
```

## Debugging

- To force Taichi be single threaded, you can write `ti.init(arch=ti.cpu, cpu_max_num_threads=1)`.
