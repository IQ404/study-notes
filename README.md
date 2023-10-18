# Taichi Lang

- Taichi is statically typed.

### Installation

Currently I will only show how I installed Taichi in my system.

I'm on Windows 10, with python 3.10 installed, type the following in PowerShell:

```
python3.10 -m pip install --upgrade taichi
```

We can now `import` taichi in our python 3.10 program.

Note that the version of Taichi I was using when writing this notes is <ins>1.6.0</ins>.

### Keywords:

- Differentiable Simulation
- Spatially sparse

### `ti.init()`

Always initialize Taichi with `ti.init()` before you do any Taichi operations.

```python
import taichi as ti

ti.init(arch=ti.gpu)  # if none of the gpu architecture is detected
                      # this will automatically falls back on cpu architectures.
```

This will specify the backend (i.e. the hardware architecture) for Taichi to run on.

### Common Data Types

- 32 bits integer: `ti.i32`
- 32 bits unsigned integer: `ti.u32`
- 32 bits floating point number: `ti.f32`

### Type Casts

```python
@ti.kernel
def f():
    a = 1
    b = ti.cast(a, ti.f32)
    print(b)    # 1.000000
```

### Compound Types

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

    M = ti.Matrix([[1,2],[3.0, 4]])    # [[1.000000, 2.000000], [3.000000, 4.000000]]
    print(M)

    r = ti.Struct(ori=a,dir=b,vision=10.0)
    print(r.vision)    # 10.000000
```

### Tensor

Tensor in Taichi is like a multi-dimensional array.

An element in a tensor can be either a scalar (`ti.var`), a vector (`ti.Vector`) or a matrix (`ti.Matrix`).

- Note that tensor and matrix in Taichi are two different constructs.

Always access element in a tensor via the `a[i,j,k]` syntax.

- There is no pointer in Taichi.

Examples of tensor:

```python
loss = ti.var(dt=ti.f32, shape=())
a = ti.var(dt=ti.f32, shape=(42, 63))
b = ti.Vector(3, dt=ti.f32, shape=4)
C = ti.Matrix(2, 2, dt=ti.f32, shape=(3, 5))

loss[None] = 3
print(loss[None])

a[3, 4] = 1
print('a[3, 4] = ', a[3, 4])

b[0] = [1, 2, 3]
print('b[0] = ', b[0][0], b[0][1], b[0][2])
```

Use `ti.Matrix` only for small matrices, use tensor for large mtrices.

`*` is for element-wise multiplication, `@` is for matrix multiplication.

### Kernel and func

Both `@ti.kernel` and `@ti.func` can have at most <ins>one</ins> `return` statement.

`@ti.func` cannot be called directly from python scope.

`@ti.func` can be called from `@ti.kernel` and other `@ti.func`.

`@ti.kernel` cannot be called from `@ti.func` or `@ti.kernel`.

Since, by the time of writing, `@ti.func` is force-inlined, it can not be recursive.

Recursion is also not allowed in `@ti.kernel`.

- Example:

```python
@ti.kernel
def fun(x: ti.f32) -> ti.i32:
    return x
```

---

Taichi supports chaining comparison.

E.g.

```python
5 > 4 >= 3 != 5    # True
```

---

### Range `for` loop

```python
@ti.kernel
def f():
    for i in range(3):    # parallelized
        print(i)
        for j in range(3):    # serialized
            print(i,j)

@ti.kernel
def g():
    for i,j,k in ti.ndrange((3,8),(1,6),9):    # parallelized, 3 <= i < 8, 1 <= j < 6, 0 <= k < 9
        print(i,j,k)

@ti.kernel
def h(p: ti.i32):
    if p > 0:
        for i in range(3):    # serialized
            print(i)
```

Note that for the last example, if the condition `if`-statement is known at the compile-time (and hence is optimized away by the compiler. E.g., `if p == p:`), then the following `for`-loop becomes the outermost loop, and thus it will be parallelized.

### Struct `for` loop

```python
import taichi as ti
ti.init(arch=ti.gpu)

n = 320
pixels = ti.var(dt=ti.i32, shape=(n*2, n))

@ti.kernel
def paint():
    for i,j in pixels:    # parallelized
        pixels[i,j] = i + j
```

### Atomic Operation

In Taichi, augmented assignment (e.g., `+=`) are made to be atomic operation.

```python
import taichi as ti
ti.init(arch=ti.gpu)

s = ti.var(dt=ti.i32, shape=())
t = ti.var(dt=ti.i32, shape=())

@ti.kernel
def sum():
    for i in range(5):    # parallelized
        s[None] += i    # sum correctly
        t[None] = t[None] + i    # data race
```



