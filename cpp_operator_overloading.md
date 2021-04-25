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

## Built-in operator
Let's consider the following example:
```cpp
#include <iostream>

struct A {
  A(int i) : m_i(i) {}
  operator bool() const { return m_i > 0; }
  int m_i;
};

int main() {
  A a1(1), a2(2);
  std::cout << a1 + a2 << (a1 == a2);
}
```
This program outputs ```21``` because for both ```a1 + a2``` and ```a1 == a2```, the built-in operators **operator+ and operator== for int is used**. The two objects a1 and a2 are implicitly converted first to bool and then to int before calling the operator. In both cases, they convert first to true, then to 1.
1 + 1 equals 2, and 1 == 1 is true, so ```21``` is printed.

But, ```struct A``` does not overload operator+, so we need to use a built-in operator+. Can we somehow convert a1 and a2 to a type compatible with operator+? The C++ standard says in [[conv](https://timsong-cpp.github.io/cppwp/n4659/conv#2)]:
>expressions with a given type will be implicitly converted to other types in several contexts:
> — When used as operands of operators
> (...)

So we are allowed to use an implicit conversion sequence to convert a1 and a2 into something we can use with a built in operator+. The specific type of implicit conversion sequence we need in this case is a user-defined conversion sequence, which is defined in [[over.ics.user](https://timsong-cpp.github.io/cppwp/n4659/over.ics.user#1)]:
>A user-defined conversion sequence consists of an initial standard conversion sequence followed by a user-defined conversion (12.3) followed by a second standard conversion sequence.

As a first try, can we use the user-defined conversion to bool (our operator bool() const) and then use an operator+ for bool? [over.built](https://timsong-cpp.github.io/cppwp/n4659/over.built#2)]:
>For every promoted arithmetic type T , there exist candidate operator functions of the form
>```T operator+(T);```

And what's a promoted arithmetic type? [[over.built](https://timsong-cpp.github.io/cppwp/n4659/over.built#2)]:
>the term promoted integral type is used to refer to those integral types which are preserved by integral promotion [...]. Similarly, the term promoted arithmetic type refers to floating types plus promoted integral types.

So is bool a promoted arithmetic type? It's certainly not a floating type, but is it a promoted integral type? No, [[conv.prom](https://timsong-cpp.github.io/cppwp/n4659/conv.prom#6)]:
>A prvalue of type bool can be converted to a prvalue of type int, with false becoming zero and true becoming one.

So operator+ does not exist for bool, and we have to continue with our implicit conversion sequence. After the user-defined conversion from A to bool, we can use a standard conversion sequence, in particular the integral promotion we just discussed, converting true to 1. We then call operator+ for int, with the operands 1 and 1, resulting in 2.

For operator==, the story is exactly the same, as [[over.built](https://timsong-cpp.github.io/cppwp/n4659/over.built#13)] says:

>For every pair of promoted arithmetic types L and R , there exist candidate operator functions of the form (...)
> ```bool operator==(L, R);```

So we use the same conversion sequence from A to bool to int, ending up comparing 1 to 1, which results in true.
