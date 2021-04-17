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

## Ambiguous call
Let's consider the following example:
```cpp
#include <iostream>

void print(char const *str) { std::cout << str; }
void print(short num) { std::cout << num; }

int main() {
  print("abc");
  print(0);
  print('A');
}
```
The compilation fails because of an ambiguous call at ```print(0)```.
The statement ```print(0);``` is ambiguous due to overload resolution rules. Both print functions are viable, but for the compiler to pick one, one of them has to have a better conversion sequence than the other. [[over.match.best](https://timsong-cpp.github.io/cppwp/n4659/over.match.best#2)]: "If there is exactly one viable function that is a better function than all other viable functions, then it is the one selected by overload resolution; otherwise the call is ill-formed".

(a) Because 0 is a null pointer constant[1], it can be converted implicitly into any pointer type with a single conversion.

(b) Because 0 is of type int, it can be converted implicitly to a short with a single conversion too.
In our case, both are standard conversion sequences with a single conversion of "conversion rank". Since no function is better than the other, the call is ill-formed.

\[1\] [[conv.ptr](https://timsong-cpp.github.io/cppwp/n4659/conv.ptr#1)] A null pointer constant is an integer literal (§ 5.13.2) with value zero or a prvalue of type ```std::nullptr_t```. A null pointer constant can be converted to a pointer type.

## Sizeof and unary expressions
Let's consider the following example:
```cpp
#include <iostream>

int main() {
   int n = sizeof(0)["abcdefghij"]; 
   std::cout << n;   
}
```
The program outputs ```1``` because the C++ standard details the grammar for ```sizeof``` in [[expr.unary](https://timsong-cpp.github.io/cppwp/n4659/expr.unary#1)]:
>unary-expression:
> - ...
> - sizeof unary-expression
> - sizeof ( type-id )
> - sizeof ... ( identifier )
> - ...

We have three cases and the one that applies here is sizeof unary-expression. The unary expression is ```(0)["abcdefghij"]```, which looks odd but is just array indexing of string literal which is a **const char array**.
We can see that ```(0)["abcdefghij"]``` is identical to ```("abcdefghij")[0]``` from [[expr.sub](https://timsong-cpp.github.io/cppwp/n4659/expr.sub#1)] which says:
>... The expression E1[E2] is identical (by definition) to \*((E1)+(E2)) ...

So we end up with 0th element of "abcdefghij", which is a, which is a ```char```. And the result of sizeof('a') will be ```1``` since [[expr.sizeof](https://timsong-cpp.github.io/cppwp/n4659/expr.sizeof#1)] says:
>... sizeof(char), sizeof(signed char) and sizeof(unsigned char) are 1 ...
