# CPP Move/Copy Constructor

## Copy/Move construction on a mutable object
Let's consider the following example:
```cpp
#include <iostream>
#include <utility>

struct X {
    X() { std::cout << "1"; }
    X(X &) { std::cout << "2"; }
    X(const X &) { std::cout << "3"; }
    X(X &&) { std::cout << "4"; }
    ~X() { std::cout << "5"; }
};

struct Y {
    mutable X x;
    Y() = default;
    Y(const Y &) = default;
};

int main() {
    Y y1;
    Y y2 = std::move(y1);
}
```
The program outputs ```1255``` but many parameters come into play, so let's consider them one by one:
### 1 - A simple ordinary construction of y1
Like the title says, the ```1``` is displayed after the construction of y1 since it calls ```X()```.

### 2 - Move operation on an object with no move constructor
In the line ```Y y2 = std::move(y1);```, the ```std::move``` turns y1 into an **rvalue**, but since **y has no move constructor, its copy constructor is called**.
 The implicitly-defined copy constructor performs a copy of ```x```, as specified in the
 C++ standard [[class.copy.cotr](https://timsong-cpp.github.io/cppwp/n4659/class.copy.ctor#14)]:
>The implicitly-defined copy/move constructor for a non-union class X performs a memberwise copy/move of its bases and members.

### 3 - Copy constructor of a mutable object
Now the question is, which copy constructor to call for x? ```X(X&)``` or ```X(const X&)```?
Since x is **mutable**, the C++ standard is clear about copy/move construction of mutable
objects in [[dcl.stc](https://timsong-cpp.github.io/cppwp/n4659/dcl.stc#9)]:
>The mutable specifier on a class data member nullifies a const specifier
> applied to the containing class object and permits modification
> of the mutable class member even though the rest of the object is const

So x is considered non-const, and ```X(X&)``` is a better match to overload resolution than ```X(const X&)```, because in the second case,
 a **const conversion has to happen**, so ```X(X&)``` is called, printing ```2```.
 
### 4 - Destruction
Finally, both y1 and y2 are destroyed at the end of ```main()```, displaying ```55```.

##
Let's consider the following example:
```cpp
#include <iostream>
#include <utility>

struct X {
    X() { std::cout << "1"; }
    X(X &) { std::cout << "2"; }
    X(const X &) { std::cout << "3"; }
    X(X &&) { std::cout << "4"; }
    ~X() { std::cout << "5"; }
};

struct Y {
    mutable X x;
    Y() = default;
    Y(const Y &) = default;
};

int main() {
    Y y1;
    Y y2 = std::move(y1);
}
```
The program outputs ```1255```: First, the line ```Y y1;``` creates an instance of ```Y``` which has an ```X``` data member, which is default constructed, printing ```1```.
Then, ```Y y2 = std::move(y1);``` copy-initializes another ```Y```. The ```std::move``` **turns y1 into an rvalue**, but since Y has no move constructor, its **copy constructor is called**. The implicitly-defined copy-constructor performs a copy of x, as specified in [[class.copy.ctor](https://timsong-cpp.github.io/cppwp/n4659/class.copy.ctor#14)]:
>The implicitly-defined copy/move constructor for a non-union class X performs a memberwise copy/move of its bases and members.

Now the question is which copy constructor is used to initialize the copy of x. Will it pick ```X(X &)``` or ```X(const X &)```? The Y inside the Y(const Y &) copy constructor is const, but x is marked **mutable**, and [[dcl.stc](https://timsong-cpp.github.io/cppwp/n4659/dcl.stc#9)] says:
>The mutable specifier on a class data member nullifies a const specifier applied to the containing class object and permits modification of the mutable class member even though
>the rest of the object is const

So ```x``` is considered **non-const**, and ```X(X &)``` is a better match to overload resolution than ```X(const X &)```, because in the latter case, a const conversion has to happen. So ```X(X &)``` is called, printing ```2```.

Finally, both y1 and y2 are destroyed at the end of main, printing ```55```.

## Difference between copy construction and assignment operator
Let's consider the following example:
```cpp
#include <iostream>

struct C {
  C() { std::cout << "1"; }
  C(const C& other) { std::cout << "2"; }
  C& operator=(const C& other) { std::cout << "3"; return *this;}
};

int main() {
  C c1;
  C c2 = c1;
}
```
The program outputs ```12``` because on the first line, ```C c1;```, the object c1 is default initialized, so the **default constructor** is called, printing ```1```.

On the second line, ```C c2 = c1;```, the object c2 is **copy initialized**, so the **copy constructor is called**, printing ```2```. (Note that no assignment is taking place, even though there's a = involved.) The C++ standard says in [[dcl.init](https://timsong-cpp.github.io/cppwp/n4659/dcl.init#15)]:
>"The initialization that occurs in the form of a brace-or-equal-initializer (...) is called copy-initialization."

The initialization of ```C c2 = c1;``` is the **equal** part of a braced-or-equal-initializer, so we have **copy-initialization**.
[[dcl.init](https://timsong-cpp.github.io/cppwp/n4659/dcl.init#16)] also says:
>If the destination type is a (possibly cv-qualified) class type (...) If the initialization is direct-initialization, or if it 
>is copy-initialization where the cv-unqualified version of the source type is the same class as, or a derived class of, the class of the destination, constructors are >considered. The applicable constructors are enumerated (§16.3.1.3), and the best one is chosen through overload 
>resolution (§16.3). The constructor so selected is called to initialize the object, 
>with the initializer expression or expression-list as its argument(s).

The destination type here is ```C```, and the initialization is **copy-initialization**. The source type is the same class as the class of the destination, so constructors are considered. The only constructor taking a C as an argument is our copy constructor, which prints ```2```.
