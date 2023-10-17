# Taichi Lang

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

### Kernel and func



Taichi function (decorated with `@ti.func`) cannot be called directly from python scope.

It can be called from `@ti.kernel` and other `@ti.func`.

Since, by the time of writing, it is force-inlined, it can not be recursive.

`@ti.func` can at most have <ins>one</ins> `return` statement.
