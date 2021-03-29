# CPP Operator Overloading

## Hiding new and new[] operators
By making an empty private ```new``` and ```new[]``` operators, we can restrict dynamic allocation of objects:
```cpp
// will be updated
```

## Auto type conversion operator
Let's consider the following example:
```cpp
#include <iostream>

struct Foo {
  operator auto() {
    std::cout << "A";
    return 1;
  }
};

int main() {
  int a = Foo();
  std::cout << a;
}
```
The program is guaranteed to output ```A1``` because a normal conversion function can have a deduced return type (only conversion function templates are not allowed so due to [[class.conv.fct](https://timsong-cpp.github.io/cppwp/n4659/class.conv.fct#6)]:
>A conversion function template shall not have a deduced return type (§10.1.7.4).

And even if conversion functions don't have a return type specified in the same way as normal functions, they do have a return type: [[class.conv.fct](https://timsong-cpp.github.io/cppwp/n4659/class.conv.fct#1)]:
>The type of the conversion function (§11.3.5) is “function taking no parameter returning conversion-type-id”.

Where conversion-type-id is the ```T``` in operator ```T()``` {...}. So we're allowed to deduce this type just like we deduce return types from normal functions.
