# Taichi Lang

### Keywords:

- Differentiable Simulation

### `ti.init()`

- Always initialize Taichi with `ti.init()` before you do any Taichi operations.

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

Tensor in Taichi is just a multi-dimensional array.

