# Taichi Lang

### Keywords:

- Differentiable Simulation

### `ti.init()`

Always initialize Taichi with `ti.init()` before you do any Taichi operations.

```python
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

An element in a tensor can be either a scalar (`var`), a vector (`ti.Vector`) or a matrix (`ti.Matrix`).

- Note that tensor and matrix in Taichi are two different constructs.

Always access element in a tensor via the `a[i,j,k]` syntax.

There is no pointer in Taichi.

