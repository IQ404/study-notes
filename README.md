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

## `static_assert`

I may get this wrong but in my current understanding, when compiler at compile-time sees `static_assert(bool-constexpr , "message");`, at least for a C++20 compiler, it will evaluate `bool-constexpr`, and as soon as this `bool-constexpr` is evaluated to `false`, a compilation error is reported (and thus your program won't compile).

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

- In my current understanding, the typename(s) in `template<...>` specify the type parameters the compiler will be required to define the following construct.

  For instance, for a templated class containing a templated member function, the `template<...>` declared for the class does not need to contain the type parameters to be used in the templated member function. I think this is (at least partially) because the compiler does not need to define the templated member function immediately when defining the class (of course we can design to include the type parameter(s) to be used in the member function in the `template<...>` declared for the class, but then the member function is not a templated member function anymore, and then the compiler will define the templated member function immediately when defining the class). Currently I think of templated class and its templated member function as somewhat separated constructs as they are defined in separated compilation steps.

- In my current understanding, at compile-time, when the compiler sees a normal class with a templated member function, it is able to compile it into a class as if without the templated member function, but in somewhere it will record the identifier of that templated member function as a valid name within the class. Later (note that it is still in compile-time) when the compiler sees the usage of that templated member function, the compiler has the ability to compile that member function definition into the class according to the usage. 

### Template Specialization

Example usage:

```cpp
class Demo
{
public:
    template<typename T>
    static void fun()
    {
        std::cout << "class Demo has no implementation for this type on Demo::fun()" << std::endl;
    }

    template<>
    static void fun<int>()
    {
        std::cout << "Demo::fun()'s implementation for int" << std::endl;
    }

    template<>
    static void fun<float>()
    {
        std::cout << "Demo::fun()'s implementation for float" << std::endl;
    }

    template<>
    static void fun<double>()
    {
        std::cout << "Demo::fun()'s implementation for double" << std::endl;
    }
};

int main()
{
    //Demo::fun();        // error
    Demo::fun<char>();  // class Demo has no implementation for this type on Demo::fun()
    Demo::fun<int>();  // Demo::fun()'s implementation for int
    Demo::fun<float>();  // Demo::fun()'s implementation for float
    Demo::fun<double>();  // Demo::fun()'s implementation for double
}
```

- In my current mental picture, there is somewhere (e.g. a table, here for the class `Demo`) during compile-time which records all the identifiers of the specializations of the "mother" templated function (here it is `fun`). When the compiler first hit a usage of `fun`, it will first search within this table for the usage type. If the type is found in this table, the compiler will define a function (here it's a member function) according to the specialization. If not found, it will use the "mother" definition.

- Note that we cannot have a function declared with `template<>` without the existence of a templated function it is specialized for.

## `std::pair`

## `std::function`

# Part II: Styles

## Avoid Singletons

The initialization order of global variables <ins>in single translation unit</ins> is defined by the order of their declarations.

The initialization order of global variables across multiple tranlation units are undefined.

You can use Meyers singleton, but (❓ more on this are needed) it is still not thread safe.

The takeaway is: <ins>keep your objects dependencies in its own single tranlation unit</ins>.
