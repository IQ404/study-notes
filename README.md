# Native Python

# NumPy



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
```

# JAX
