# C++

## Contents

# Part I: Features and Tools

## Preprocessing

- A backslash (`\`) followed by a newline (hitting Enter), often refer to as line continuation, will remove the newline character (and the `\`) in preprocessing phase.

  The process of handling a backslash (`\`) followed by a newline (hitting Enter), even outside of preprocessor directives, is indeed the work of the preprocessor in C++.

- The `#` operator in preprocessing, often known as the stringizing operator, converts its following argument into a string literal.

  E.g.

  ```cpp
  #define STRINGIZE(x) #x
  ```

  The stringizing operator (`#`) in C++ can only be used within a macro definition (`#define`). `#` has no special meaning in regular C++ code.

- `__LINE__` returns the line it is placed.

- `__FILE__` return the file path it is placed.

- `__debugbreak()` is a <ins>MSVC-specific</ins> function that will break at the place where it is called if the visual studio project is run with the debugger, or it will exit the program if the project is run without the debugger.

- In my current understanding: (❓ NEED DEEPER UNDERSTANDING)

  - When preprocessor sees the syntax `#include "..."`, it will first search, within the files included in the C++ project from the Solution Explorer of Visual Studio, for the file specified by `"..."` under the directory where the file containing this line of `#include "..."`. If not found, it will search under the folders recorded in `Properties -> Configuration Properties -> C/C++ -> General -> Additional Include Directories` of the C++ project in Visual Studio. If still not found, it will give an error.
  - When preprocessor sees the syntax `#include <...>`, it will search under the folders recorded in `Properties -> Configuration Properties -> C/C++ -> General -> Additional Include Directories` of the C++ project in Visual Studio. If not found, it will give an error.

## `static` function

Functions declared with `static` keyword at namespace scope will have internal linkage, which means it is only visible within its own translation unit (which is the "thing" merged by the preprocessor from all the associated header files and `.cpp` files).

On the other hand, external linkage (e.g. functions and variables that are defined at namespace scope) means it is visible across translation units (i.e. visible to the linker), so that, e.g., function definition in a `.cpp` file can be linked to the function declaration in another `.cpp` file.

## Dynamical Allocation on Stack

One way to do dynamical allocation on the stack:

```cpp
int length = 10;

// instead of doing "char arr[length];" we can do:

char* arr = (char*)alloca(length * sizeof(char));
```

// TODO: stack overflow caused by large number of nested function calls vs by allocating large array.

❓ How is dynamical allocation on the stack affecting the performance compared to static allocation?

## `mutable` data members

In a class, a member function marked as `const` is allowed to modify `mutable` data members.

E.g.

```cpp
class C
{
    mutable int var = 0;

public:

    void fun() const
    {
        var++;  // OK
    }
};
```

## Pointer

```cpp
int*& r;  // r is a reference to a pointer to an int

const int* x;  // x is a pointer to a constant int
int const* y;  // y is a pointer to a constant int, same as above
int* const z;  // z is a constant pointer to a (mutable) int
int const i;  // this is equivalent to const int i;
```

## Lambda Expression

## Template

## `std::pair`

## `std::function`

# Part II: Styles

## Avoid Singletons

