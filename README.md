# Taichi Lang

### Keywords:

- Differentiable Simulation

---

- Always initialize Taichi with `ti.init()` before you do any Taichi operations.

```python
ti.init(arch=ti.gpu)  # if none of the gpu architecture is detected
                      # this will automatically falls back on cpu architectures.
```

This will specify the backend hardware architecture for Taichi to run on.

- 
