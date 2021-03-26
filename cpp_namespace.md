# CPP Namespace

## Koenig lookup
Lets consider the following example:
```cpp
#include <iostream>

namespace x {
  class C {};
  void f(const C& i) {
    std::cout << "1";
  }
}

namespace y {
  void f(const x::C& i) {
    std::cout << "2";
  }
}

int main() {
  f(x::C());
}
```
One would suspect that this example will fail because the functions ```f``` are declared inside namespaces, and the call ```f(x::C())``` is **unqualified**
 (not preceded by x:: or y::).
 
However, due to [argument-dependent name loockup](https://en.wikipedia.org/wiki/Argument-dependent_name_lookup) aka "Koenig lookup", the behavior is well defined.

With argument-dependent name lookup, the namespaces of the arguments to a function is added to the set of namespaces to be searched for that function.
Since we're passing an ```x::C``` to ```f```, the namespace ```x``` is also searched, and the function ```x::f``` is found.

The C++ standard says in [[basic.lookup.argdep](https://timsong-cpp.github.io/cppwp/n4659/basic.lookup.argdep#1)]:
>When the postfix-expression in a function call (§8.2.2) is an unqualified-id, other namespaces not considered during 
>the usual unqualified lookup (§6.4.1) may be searched(...). These modifications to the search depend on the types of the arguments (...).

```f``` is an **unqualified-id**, so "other namespaces" may be searched.

Also, [[basic.lookup.argdep](https://timsong-cpp.github.io/cppwp/n4659/basic.lookup.argdep#2)] says:
>For each argument type T in the function call, there is a set of zero or more associated namespaces and a set of zero or more associated
> classes to be considered. The sets of namespaces and classes are determined entirely by the types of the function arguments (...).
> The sets of namespaces and classes are determined in the following way:
> (...)
>  - If T is a class type (including unions), its associated classes are: the class itself; the class of which it is a member, 
>  if any; and its direct and indirect base classes. Its associated namespaces are the namespaces of which its associated classes are members.

```T``` in our case is ```C```, so it's associated classes is just ```C```. It's associated namespaces is then just ```x```, which is searched to find a function ```f```.
