# Metaknowledge

- A simple place to get familiar with Python is [here](https://www.learnpython.org/).

# Module and Package

- Any Python file with a .py extension is called a module.
- The name of the module is the same as the file name (without the .py extension).
- You can use the import statement `import <module_name>` in one module to import another module.
  - When the Python interpreter sees `import <module_name>`, it will look for the `<module_name>.py` file in the directory holding the module that is executed as the main program.
  - PYTHONPATH is an environment variable that you can set to add additional directories. If not found, Python interpreter will look for `<module_name>.py` in those additional directories.
  - If not found, 

# NumPy

```python
import numpy as np

def f(x):
    return x**2

x_array = np.linspace(-5, 5, 100)
dfdx_numerical = np.gradient(f(x_array_2), x_array_2)
```

- The `np.vectorize` function in NumPy is used to transform a Python function that accepts and returns scalars into a "vectorized" function. This vectorized function can then operate element-wise on arrays, making it possible to apply the function to each element of an array without explicitly writing a loop. This is particularly useful for integrating non-NumPy functions into NumPy operations.

- [Broadcasting](https://numpy.org/doc/stable/user/basics.broadcasting.html#:~:text=The%20term%20broadcasting%20describes%20how,that%20they%20have%20compatible%20shapes.)

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

expr_np = lambdify(x, expr, 'numpy')

import numpy as np
x_array = np.array([1, 2, 3])

print("x: \n", x_array)
print("f(x) = x**2: \n", expr_np(x_array))

# Symbolic Differentiation:
dfdx_composed = diff(exp(-2*x) + 3*sin(3*x), x)
```

# JAX

```python
import numpy as np
narr = np.array([1, 2, 3])

import jax.numpy as jnp  # Package jax.numpy is a wrapped NumPy, which pretty much replaces NumPy when JAX is used

arr = jnp.array([1., 2., 3.])
print("Type of JAX NumPy array:", type(arr))

# or it can be created by converting existing NumPy array:
arr = jnp.array(narr.astype('float32'))

# JAX arrays are immutable:

try:
    arr[2] = 4.0
except TypeError as err:
    print(err)

arr2 = arr.at[2].set(4.0)
print(arr)
print(arr2)

# For automatic differentiation (autodiff):

from jax import grad, vmap

def f(x):
    return jnp.abs(x)

print("Function value at x = 3:", f(3.0))
print("Derivative value at x = 3:",grad(f)(3.0))
print("Function value at x = -3:", f(-3.0))
print("Derivative value at x = -3:",grad(f)(-3.0))
# NOTE that we cannot feed in the gradient function with integer

# We also cannot directly feed in the gradient function with array:
try:
    grad(f)(arr)
except TypeError as err:
    print(err)
# Instead we can do:
dfdx_jax_vmap = vmap(grad(f))(arr)
print(dfdx_jax_vmap)
```
- ❓ Automatic differentiation (autodiff) method breaks down the function into common functions (sin, cos, log, power functions, etc.), and constructs the computational graph consisting of the basic functions. Then the chain rule is used to compute the derivative at any node of the graph. It is the most commonly used approach in machine learning applications and neural networks, as the computational graph for the function and its derivatives can be built during the construction of the neural network, saving in future computations.
