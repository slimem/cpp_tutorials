# CPP Pointers

## Initializing a variable with its own address
Lets consider the following example:
```cpp
#include <iostream>

int main() {
  void * p = &p;
  std::cout << bool(p);
}
```
It is guaranteed to output ```1```, because as defined in [[basic.scope.pdecl](https://timsong-cpp.github.io/cppwp/n4659/basic.scope.pdecl#1)], the point of name declaration is after its
complete declarator and before its initialisation. This
means that line 4 is valid C++, because it's possible
to initialise the variable p with the address of an existing
variable, even if it is its own address.

The value of p is unknown, but can not be a null pointer value. The
cast must thus evaluate to ```1``` and initialise the temporary
```bool``` as true.

## Literal string to char* conversion
Let's consider the following example:
```cpp
#include <iostream>
     
int main() {
    char* str = "X";
    std::cout << str;
}
```
The compilation fails because the C++ standard defines literal strings as the following in [[lex.string](https://timsong-cpp.github.io/cppwp/n4659/lex.string#8)]:
>A narrow string literal has type “array of n const char”, where n is the size of the string as defined below, and has static storage duration.

An array of n const char converts to a **pointer to const char**. A note in [[conv.qual](https://timsong-cpp.github.io/cppwp/n4659/conv.qual#4)] extrapolates from the preceeding normative passages that "a prvalue of type “pointer to cv1 T” can be converted to a prvalue of type “pointer to cv2 T” if “cv2 T” is more cv-qualified than “cv1 T”." In this case however, char* is less cv-qualified than const char* , and the conversion is not allowed.

NOTE: While most compilers still allow ```char const[]``` to ```char*``` conversion with just a warning, this is **not a legal conversion since C++11**.

Check the following [link](https://dev.krzaq.cc/post/stop-assigning-string-literals-to-char-star-already/) for more details.
