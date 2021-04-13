## CPP new and delete

# Difference between new and malloc
* ```new``` is an operator, ```malloc``` is a function.
* ```new``` calls constructor, ```malloc``` doesn't.
* ```new``` returns appropriate pointer, ```malloc``` returns ```void *``` and pointer needs to typecast to appropriate type.

# A dynamic array of object
The following allocates dynamically an array[5] of ints. Please note that the returned type is a pointer to the allocated type (after ```new``` operator)
```cpp
int* foo = new int [5];
delete[] foo
```
Following the same rule, we can allocate an array of pointers to int: The returned type will be a pointer to an array of pointers:
```cpp
int** foo = new int* [5];
delete[] foo;
```

# Delete effects
Deleting a ```NULL``` pointers has no effect, so it is unnecessairy to check for ```NULL``` before ```delete```:
```cpp
int* p = NULL;
delete p;
```
On the other hand, calling ```delete``` twice to an allocated pointer is undefined behaviour:
```cpp
int* p = new int;
delete p;
delete p; // <= Access violation
```

## Sizeof of operator new
Let's consider the following example:
```cpp
#include <iostream>

struct A {
    A() { std::cout << "A"; }
    ~A() { std::cout << "a"; }
};

int main() {
    std::cout << "main";
    return sizeof new A;
}
```
The program only outputs ```main```.

[[expr.sizeof](https://timsong-cpp.github.io/cppwp/n4659/expr.sizeof#1)] in the standard:
>The sizeof operator yields the number of bytes in the object representation
> of its operand. The operand is either an expression, which is an unevaluated operand (Clause 8) or a parenthesized type-id.

The most common form of ```sizeof``` is probably the latter, ```sizeof(type-id)``` (e.g. sizeof(int)). 
But there's also the ```sizeof expression``` form (e.g. sizeof new A) which is used in this question.

The operand here is the expression ```new A```. The type of that expression is **"pointer to A"**. The size of a pointer varies between platforms, but that doesn't really matter in this question since we never print it. The question is whether **new A constructs a new A, and if so, whether it gets destructed**.

The quote above says that the expression is **"an unevaluated operand"**, but what does that mean?

[[expr](https://timsong-cpp.github.io/cppwp/n4659/expr#8)]:
>In some contexts, unevaluated operands appear (8.2.8, 8.3.3, 8.3.7, 10.1.7.2). An unevaluated operand is not evaluated.

So the expression ```new A``` is **not evaluated**, and A is **never constructed**. The expression is only used for sizeof. And since there is no ```delete```, A is never destructed either.
