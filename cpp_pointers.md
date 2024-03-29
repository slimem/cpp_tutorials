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

## Typeid of pointers and int[]
Let's consider the following example:
```cpp
#include <iostream>
#include <typeinfo>

void takes_pointer(int* pointer) {
  if (typeid(pointer) == typeid(int[])) std::cout << 'a';
  if (typeid(pointer) == typeid(int*)) std::cout << 'p';
}

void takes_array(int array[]) {
  if (typeid(array) == typeid(int[])) std::cout << 'a';
  if (typeid(array) == typeid(int*)) std::cout << 'p';
}

int main() {
  int* pointer = nullptr;
  int array[1];

  takes_pointer(array);
  takes_array(pointer);

  std::cout << (typeid(int*) == typeid(int[]));
}
```
The program outputs ```pp0``` because functions taking pointers can also be called with arrays, and vice versa. But **arrays and pointers are not the same**.

First let's look at ```takes_pointer(array);```. What happens here is usually referred to as **the array "decaying" to a pointer**. To be a bit more precise, let's have a look at [[conv.array](https://timsong-cpp.github.io/cppwp/n4659/conv.array)] in the C++ standard:
>An lvalue or rvalue of type "array of N T" or "array of unknown bound of T" can be converted to a prvalue of type "pointer to T".

```array``` is of type "array of 1 int", which converts to a prvalue (temporary) of type "pointer to int".
So what happens with ```takes_array(pointer);```, does the pointer convert to an array? No, it's actually the other way around. Let's look at [[dcl.fct](https://timsong-cpp.github.io/cppwp/n4659/dcl.fct#5)] about function parameters:
>After determining the type of each parameter, any parameter of type "array of T" (...) is adjusted to be "pointer to T"

So in ```void takes_array(int array[])```, the type of array is **adjusted to be pointer to int**.

## Sizeof Pointer VS Array
Let's consider the following example:
```cpp
#include <iostream>
using namespace std;

size_t get_size_1(int* arr)
{
  return sizeof arr;
}

size_t get_size_2(int arr[])
{
  return sizeof arr;
}

size_t get_size_3(int (&arr)[10])
{
  return sizeof arr;
}

int main()
{
  int array[10];
  //Assume sizeof(int*) != sizeof(int[10])
  cout << (sizeof(array) == get_size_1(array));
  cout << (sizeof(array) == get_size_2(array));
  cout << (sizeof(array) == get_size_3(array));
}
```
The program outputs ```001```.
This question compares three ways for a function to take an array as parameter, while **two of them are actually the same**.

In main, the array is of array type, therefore the ```sizeof``` operator returns the size of the array in terms of bytes. [[expr.sizeof](https://timsong-cpp.github.io/cppwp/n4659/expr.sizeof#2)] in the standard says:
> When applied to an array, the result \[of the sizeof operator\] is the total number of bytes in the array. This implies that the size of an array of n elements is n times the size of an element.

In ```get_size_3```, the parameter is a reference to an array of **size 10**, therefore the sizeof operator returns the size of the array in terms of bytes. [[expr.sizeof](https://timsong-cpp.github.io/cppwp/n4659/expr.sizeof#2)] in the standard says:
> When applied to a reference or a reference type, the result is the size of the referenced type.

In ```get_size_1``` and ```get_size_2```, **the parameter is a pointer**, therefore the sizeof operator returns the size of the pointer. Although the parameter of ```get_size_2``` is an array, it is adjusted into a pointer: [[dcl.fct](https://timsong-cpp.github.io/cppwp/n4659/dcl.fct#5)] in the standard says:
> any parameter of type "array of T" (...) is adjusted to be "pointer to T"
