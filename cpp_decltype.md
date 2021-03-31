# CPP Decltype

## A simple decltype
Lets consider the following example:
```cpp
int main() {
  int a = 0;
  decltype((a)) b = a;
  b++;
  std::cout << a << b;
}
```
This program outputs ```11``` because the C++ standard says in [[dcl.type.simple](https://timsong-cpp.github.io/cppwp/n4659/dcl.type.simple#4)]:
>"The type denoted by decltype(e) is deﬁned as follows:
> - if e is an unparenthesized id-expression naming a structured binding ([[dcl.struct.bind](https://timsong-cpp.github.io/cppwp/n4659/dcl.struct.bind)]), decltype(e) is the referenced type as given in the specification of the structured binding declaration;
> - if e is an unparenthesized id-expression or an unparenthesized class member access ([[expr.ref](https://timsong-cpp.github.io/cppwp/n4659/expr.ref)]), decltype(e) is the type of the entity named by e. If there is no such entity, or if e names a set of overloaded functions, the program is ill-formed;
> - otherwise, if e is an xvalue, decltype(e) is T&&, where T is the type of e;
> - otherwise, if e is an lvalue, decltype(e) is T&, where T is the type of e;
> - otherwise, decltype(e) is the type of e."

Because ```a``` is encapsulated in parentheses, **it doesn't qualify for the first case, it is treated as an lvalue**, therefore **b's type is int&, not int**.
