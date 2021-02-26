# CPP static and dynamic linking

## Defining variables with C language linkage
### Definition
Let's consider the following example:
```cpp
#include <iostream>

namespace A {
  extern "C" { int x; } // Definition!!!
};

namespace B {
  extern "C" { int x; } // Definition!!! (defined for the second time)
};

int A::x = 0;

int main() {
  std::cout << B::x;
  A::x=1;
  std::cout << B::x;
}
```
The program has a compilation error because ```A::x``` and ```B::x``` refer to the same variable, and ```int x``` is a **definition** not a **declaration**.\
In the C++ standard [dcl.link](https://timsong-cpp.github.io/cppwp/n4659/dcl.link#6):
"Two declarations for a variable with C language linkage with the same name (ignoring the namespace names that qualify it)
 that appear in different namespace scopes refer to the same variable."
 
 Now, why is int x; a definition, and not an extern declaration? The C++ standard declares in [dcl.link](https://timsong-cpp.github.io/cppwp/n4659/dcl.link#7) that "*a declaration directly contained in a linkage-specification is 
 treated as if it contains the extern specifier*". x is not directly contained in the linkage specification, 
 and thus doesn't have the implicit extern. Therefore it's a definition and repeated definition of x causes a compilation error.
 
 The following example shows the difference between declaration and definition in linkage-specification:
 ```cpp
 (...)
extern "C" int i;                   // declaration
extern "C" {
  int i;                            // definition
}
(...)
 ```
 ### Declaration
 In the following example, A::x and B::x refer to the same variable so the output is ```01```.
 ```cpp
 #include<iostream>

namespace A{
  extern "C" int x;
};

namespace B{
  extern "C" int x;
};

int A::x = 0;

int main(){
  std::cout << B::x; // same variable
  A::x=1; // same variable
  std::cout << B::x;
}
 ```
 
 Due to the extern "C" specifications, A::x and B::x actually refer to the same variable.

[dcl.link](https://timsong-cpp.github.io/cppwp/n4659/dcl.link#6) in the C++ standard:
"Two declarations for a variable with C language linkage with the same name (ignoring the namespace names that qualify it) that appear in different namespace scopes refer to the same variable."
