# CPP Constructor Destructor

## Lifetime of automatic storage variables in case of a JUMP
[stmt.jump](https://timsong-cpp.github.io/cppwp/n4659/stmt.jump#2) Transfer back past an initialized variable with automatic storage duration involves 
the destruction of objects with automatic storage duration that are in scope at the point transferred from but not at the point transferred to,
this is why the following program displays ```aAaA```.
```cpp
#include <iostream>

using namespace std;

class A {
public:
  A() { cout << "a"; }
  ~A() { cout << "A"; }
};

int i = 1;

int main() {
label:
  A a;
  if (i--)
    goto label;
}
```
## Explicit copy constructor for derived class
Lets consider the following example:
```cpp
#include <iostream>
using namespace std;

class A {
public:
    A() { cout << "A"; }
    A(const A &) { cout << "a"; }
};

class B: public virtual A {
public:
    B() { cout << "B"; }
    B(const B &) { cout<< "b"; }
};

class C: public virtual A {
public:
    C() { cout<< "C"; }
    C(const C &) { cout << "c"; }
};

class D:B,C {
public:
    D() { cout<< "D"; }
    D(const D &) { cout << "d"; }
};

int main() {
    D d1;
    cout << "\n";
    D d2(d1);
}
```
The first line displays ```ABCD``` because constructors are called from the base class then left to right ([class.base.init](https://timsong-cpp.github.io/cppwp/n4659/class.base.init#13) in the C++ standard).\
But at the second line, someone would expect that it would display ***abcd*** but it's ```ABCd```, because when a copy constructor is user defined, calling the copy constructors of its base classes must be done explicitly ([class.copy.ctor](https://timsong-cpp.github.io/cppwp/n4659/class.copy.ctor#14) in the C++ standard).

## Initializer list
The following program outputs ```1133```:
```cpp
#include <initializer_list>
#include <iostream>

struct A {
  A() { std::cout << "1"; }

  A(int) { std::cout << "2"; }

  A(std::initializer_list<int>) { std::cout << "3"; }
};

int main(int argc, char *argv[]) {
  A a1;
  A a2{};
  A a3{ 1 };
  A a4{ 1, 2 };
}
```
A1 is default initialized, as describted in [dcl.init](https://timsong-cpp.github.io/cppwp/n4659/dcl.init#12)

A2 does not use the ```initializer_list``` constructor with a list of zero elemets, but the default constructor in the C++ standard [dcl.init.list](https://timsong-cpp.github.io/cppwp/n4659/dcl.init.list#3):

```cpp
struct S {
  S(std::initializer_list<double>); // #1
  S(std::initializer_list<int>);    // #2
  S();                              // #3
  // ...
};
S s1 = { 1.0, 2.0, 3.0 };           // invoke #1
S s2 = { 1, 2, 3 };                 // invoke #2
S s3 = { };                         // invoke #3
```

in A3 and A4, the ```initializer_list``` constructor is chosed with the overload resolution, as described in [over.match.list](https://timsong-cpp.github.io/cppwp/n4659/over.match.list)
This means that when a constructor with one element is provided, the standard prioritizes ```intializer_list<int>``` first, then ```A(int)```.

