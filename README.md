# C++

One way to do dynamical allocation on the stack:

```cpp
int length = 10;

// instead of doing "char arr[length];" we can do:

char* arr = (char*)alloca(length * sizeof(char));
```

Just note that if such allocation causes stack overflow, program behaviour is undefined.
