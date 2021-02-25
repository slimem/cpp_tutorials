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
