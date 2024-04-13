# Native Python

# NumPy

```python
import numpy as np

def f(x):
    return x**2

x_array = np.linspace(-5, 5, 100)
dfdx_numerical = np.gradient(f(x_array_2), x_array_2)
```

- The `np.vectorize` function in NumPy is used to transform a Python function that accepts and returns scalars into a "vectorized" function. This vectorized function can then operate element-wise on arrays, making it possible to apply the function to each element of an array without explicitly writing a loop. This is particularly useful for integrating non-NumPy functions into NumPy operations.

# SymPy

```python
# This format of module import allows to use the sympy functions without sympy. prefix.
from sympy import *

# This is actually sympy.sqrt function, but sympy. prefix is omitted.
sqrt(18)

# Numerical evaluation of the result is available, and you can set number of the digits to show in the approximated output
N(sqrt(18),8)  # sympy.N()

# Define list of symbols:
x, y = symbols('x y')  # sympy.symbols()
# Definition of the expression:
expr = 2 * x**2 - x * y

expr_manip = x * (expr + x * y + x**3)

expand(expr_manip)

factor(expr_manip)

# substitute particular values for the variables in the expression:
expr.evalf(subs={x:-1, y:2})

from sympy.utilities.lambdify import lambdify

f_symb_np = lambdify(x, f_symb, 'numpy')

import numpy as np
x_array = np.array([1, 2, 3])

print("x: \n", x_array)
print("f(x) = x**2: \n", f_symb_np(x_array))

# Symbolic Differentiation:
dfdx_composed = diff(exp(-2*x) + 3*sin(3*x), x)
```

# JAX
