# C++

One way to do dynamical allocation on the stack:

```cpp
int length = 10;

// instead of doing "char arr[length];" we can do:

char* arr = (char*)alloca(length * sizeof(char));
```

// TODO: stack overflow caused by large number of nested function calls vs by allocating large array.
