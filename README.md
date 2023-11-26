# C++

## Contents

## Preprocessing



## Currently Unclassified Notes

One way to do dynamical allocation on the stack:

```cpp
int length = 10;

// instead of doing "char arr[length];" we can do:

char* arr = (char*)alloca(length * sizeof(char));
```

// TODO: stack overflow caused by large number of nested function calls vs by allocating large array.

---

Functions declared with `static` keyword at namespace scope will have internal linkage, which means it is only visible within its own translation unit (which is the "thing" merged by the preprocessor from all the associated header files and `.cpp` files).

On the other hand, external linkage (e.g. functions and variables that are defined at namespace scope) means it is visible across translation units (i.e. visible to the linker), so that, e.g., function definition in a `.cpp` file can be linked to the function declaration in another `.cpp` file.
