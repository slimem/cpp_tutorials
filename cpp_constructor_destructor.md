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

## Initialization of an aggregate class
Let's consider the following example:
```cpp
#include <iostream>

struct S {
    int one;
    int two;
    int three;
};

int main() {
    S s{1,2};
    std::cout << s.one;
}
```
The program is expected to display ```1``` because we are allowed to specify fewer initializers than a struct/class has members, as long as it is an **aggregate**. We will go into details about aggregates later. But let's focus on the initializer clause.\
In C++ standard [[dcl.init.aggr](https://timsong-cpp.github.io/cppwp/n4659/dcl.init.aggr#8)], initialization for aggregate states the following:
>If there are fewer initializer-clauses in the list than there are elements in a non-union aggregate, then each element not explicitly initialized is initialized as follows:
>* If the element has a default member initializer (12.2), the element is initialized from that initializer.
>* Otherwise, if the element is not a reference, the element is copy-initialized from an empty initializer list (11.6.4).
>* Otherwise, the program is ill-formed.

In our case, the second bullet point applies then the program is well-formed. Maybe it is better if the compiler shows a warning about this, for instance, *gcc* and *clang* require **-Wextra** to warn about this.

### What really is a non union aggregate?
Let's start with the most important part: what's an aggregate? The C++ standard says in [[adc.init.aggr](https://timsong-cpp.github.io/cppwp/n4659/dcl.init.aggr#1)] that an aggregate is basically any array or class with nothing "special" going in, such constructor, inheritance etc:
>An aggregate is an array or a class with
>* no user-provided, explicit, or inherited constructors,
>* no private or protected non-static data members,
>* no virtual functions, and
>* no virtual, private, or protected base classes

Since ```struct S``` is none of the above and not an **union**, it is an **aggregate**.

## Function prototype OR variable definition
Lets consider the following example:
```cpp
#include <iostream>
struct X {
  X() { std::cout << "X"; }
};

int main() { X x(); }
```
Some might expect that the program will output ```X``` but IT OUTPUTS NOTHING!
```X x();``` is a function prototype, not a variable definition. Remove the parentheses (or since C++11, replace them with {}), and the program will output ```X```.

## Initialization order of member variables 1
Let's consider the following example:
```cpp
#include <iostream>

struct A {
  A() { std::cout << "A"; }
};
struct B {
  B() { std::cout << "B"; }
};

class C {
public:
  C() : a(), b() {}

private:
  B b;
  A a;
};

int main()
{
    C();
}
```
This program actually outputs ```BA``` because the initialization order of member variables is determined by their order of declaration, not their order in the initialization list.

## Initialization order of member variables 2
Let's consider the following example:
```cpp
#include <iostream>

class A {
public:
  A() { std::cout << 'a'; }
  ~A() { std::cout << 'A'; }
};

class B {
public:
  B() { std::cout << 'b'; }
  ~B() { std::cout << 'B'; }
  A a;
};

int main() { B b; }
```
The following program outputs ```abBA``` because member variables are initialized before the constructor is called. The destructor is called before member variables are destroyed.

## Copy-initialization with explicit constructor
Let's consider the following example:
```cpp
#include <iostream>
using namespace std;

class C {
public:
  explicit C(int) {
    std::cout << "i";
  };
  C(double) {
    std::cout << "d";
  };
};

int main() {
  C c1(7);
  C c2 = 7;
}
```
The program outputs ```id```, because these are two examples of initialization. The first form, ```C c1(7)```, is called **direct-initialization**, the second, ```C c2 = 7```, is called **copy-initialization**. In most cases they are equivalent, but in this example they are not, since the **int constructor is explicit**.

The key is in [[over.match.copy](https://timsong-cpp.github.io/cppwp/n4659/over.match.copy#1)]:
>"[...] as part of a copy-initialization of an object of class type, a user-defined conversion can be invoked to convert an initializer expression to the type of the object >being initialized. [...] the candidate functions are selected as follows:
> - [...]
> - **When the type of the initializer expression is a class type "cv S", the non-explicit conversion functions of S and its base classes are considered. [...]**

And how is **direct-initialization** defined?

[[dcl.init](https://timsong-cpp.github.io/cppwp/n4659/dcl.init#16)]:
> The initialization that occurs in the forms
> - T x(a);
> - T x{a};
> - (...) is called direct-initialization."

So the int constructor is not even considered for 
initialization in the second case. Instead, a standard conversion sequence 
is used to convert the integer literal to a double, and the **double constructor (the only candidate) is used**.

## Default constructor for a constant object
Let's consider the following example:
```cpp
#include <iostream>

struct C {
    C() = default;
    int i;
};

int main() {
    const C c;
    std::cout << c.i;
}
```
This program **DOES NOT COMPILE** because we're trying to default-initialize ```c```. This is not allowed since it is **const** and **C** has a defaulted (not user-provided) constructor. The C++ standard says in [[dcl.init](https://timsong-cpp.github.io/cppwp/n4659/dcl.init#7)]:
> If a program calls for the default-initialization of an object of a const-qualified type T, T shall be a const-default-constructible class type or array thereof.

Now the question is, Is ```c``` **const-default-constructible?**
[[dcl.init](https://timsong-cpp.github.io/cppwp/n4659/dcl.init#7)] says again:
> A class type T is const-default-constructible if default-initialization of T would invoke a user-provided constructor of T (not inherited from a base class) or if
> - each direct non-variant non-static data member M of T has a default member initializer or, if M is of class type X (or array thereof), X is const-default-constructible,
> - if T is a union with at least one non-static data member, exactly one variant member has a default member initializer,
> - if T is not a union, for each anonymous union member with at least one non-static data member (if any), exactly one non-static data member has a default member initializer, and
> - each potentially constructed base class of T is const-default-constructible.

```C``` does not have a user-provided constructor, and the points below don't apply either.

There are several ways we could make C const-default-constructible (and make the program compile):
- Give int i a default member initializer: int i{0}.
- Remove = default from the constructor, and instead do C::C() = default; separately outside the class definition. This constructor now counts as user-provided.
- Manually provide a constructor: C() {}
