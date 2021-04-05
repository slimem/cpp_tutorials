# CPP References

## LValue function return
If a function returns no reference, it can be used as an lvalue, this is why the following is true and displays ```30```:\
Please note that a static variable can also be initialized and returned by reference in the same function:
```cpp
#include<iostream>
using namespace std;
 
int &fun()
{
    static int x = 10;
    return x;
}
int main()
{
    fun() = 30; // <- this is allowed
    cout << fun();
    return 0;
}
```

## Dereferencing non-bound references
Since the function ```fun()``` returns a reference to a deleted object, ```fun() = 30``` will do nothing: this is undefined behaviour.\
The following displays ```10```:
```cpp
#include<iostream>
using namespace std;
 
int &fun()
{
    int x = 10;
    return x;
}
int main()
{
    fun() = 30;
    cout << fun();
    return 0;
}
```
## Universal references
The following program outputs ```112212```
```cpp
#include <iostream>
#include <utility>

int y(int &) { return 1; }
int y(int &&) { return 2; }

template <class T> int f(T &&x) { return y(x); }
template <class T> int g(T &&x) { return y(std::move(x)); }
template <class T> int h(T &&x) { return y(std::forward<T>(x)); }

int main() {
  int i = 10;
  std::cout << f(i) << f(20);
  std::cout << g(i) << g(20);
  std::cout << h(i) << h(20);
  return 0;
}
```

The T&& in the templated functions do not necessarily denote an rvalue reference, it depends on the type that is used to instantiate the template. If instantiated with an lvalue, it collapses to an lvalue reference, if instantiated with an rvalue, it collapses to an rvalue reference. See note [1].

Scott Meyers has written a very good article about this, where he introduces the concept of "universal references" (the official term is "forwarding reference") [forwarding_reference](http://isocpp.org/blog/2012/11/universal-references-in-c11-scott-meyers)

In this example, all three functions are called once with an lvalue and once with an rvalue. In all cases, calling with an lvalue (i) collapses T&& x to T& x (an lvalue reference), and calling with an rvalue (20) collapses T&& x to T&& x (an rvalue reference). Inside the functions, x itself is always an lvalue, no matter if its type is an rvalue reference or an lvalue reference.

-For the first example, y(int&) is called for both cases. Output: 11.
-For the second example, move(x) obtains an rvalue reference, and y(int&&)is called for both cases. Output: 22.
-For the third example, forward<T>(x) obtains an lvalue reference when x is an lvalue reference, and an rvalue reference when x is an rvalue reference, resulting in first a call to y(int&)and then a call to y(int&&). Output: 12.

Note [1]: [dcl.ref](https://timsong-cpp.github.io/cppwp/n4659/dcl.ref#6) in the standard: "If a typedef-name (§10.1.3, §17.1) or a decltype-specifier (§10.1.7.2) denotes a type TR that is a reference to a type T, an attempt to create the type “lvalue reference to cv TR” creates the type “lvalue reference to T”, while an attempt to create the type “rvalue reference to cv TR” creates the type TR." The example at the end of that paragraph is worth a look.

Note from the contributor: This demonstrates Scott Meyers's advice to use std::forward for forwarding references, and std::move for rvalue references.

## Reference to tenary expression with different types
Let's consider the following example:
```cpp
#include <iostream>

int main() {
    int i = 1;
    int const& a = i > 0 ? i : 1;
    i = 2;
    std::cout << i << a;
}
```
This program outputs ```21```.\
The question is, ```a``` is a reference to ```i``` or not. but this highly depends on the return types of the conditional expression for ```i``` and ```1```:
If both are **lvalue**, the result would be an **lvalue**. but here ```1``` is a **prvalue**.\
The standard says after ruling a bunch of options in [expr.cond](https://timsong-cpp.github.io/cppwp/n4659/expr.cond) (if none of them are ```void```, none of them are **class type**, they dont have the same value category, they both aren't **gvalue**, the C++ standard in [expr.cond](https://timsong-cpp.github.io/cppwp/n4659/expr.cond#6) says:\
"Otherwise, the result is a prvalue"

So the result of the teneray expression is a **prvalue** (a temporary). In other words, the reference ```a``` is bound to that temporary and does not end up modifying ```a```, this is why the program returns ```21```.

## Pointer is lvalue or lvalue?
Let's consider the following example:
```cpp
#include <iostream>

void f(char*&&) { std::cout << 1; }
void f(char*&) { std::cout << 2; }

int main() {
   char c = 'a';
   f(&c);
}
```
The program prints ```1``` because ```c``` is an **lvalue char**. **&c** returns a pointer to the **lvalue c**, but that pointer itself is an **rvalue**, since it's just a nameless temporary returned from ``operator&``.

The first overload of ```f``` takes an **rvalue reference to ```char*```**, the second takes an **lvalue reference to ```char*```**. Since the pointer is an **rvalue**, the **first overload is selected, and 1 is printed**.

## Reference-Related Objects
Lets consider the following example:
```cpp
#include <iostream>

using namespace std;

int main() {
    int a = '0';
    char const &b = a;
    cout << b;
    a++;
    cout << b;
}
```
This program outputs ```00```.
'0' is a **character literal, with type char**. (The value of '0' is actually implementation defined, but will typically be 48.) This value is then promoted to an **int**, and stored in ```a```.

We then take a reference ```b``` to ```a```. But ```b``` is a **char reference,** not an **int reference**, which means they are **not reference related**.
The C++ standard says about reference-related types in [[dcl.init.ref](https://timsong-cpp.github.io/cppwp/n4659/dcl.init.ref#4)]:
>Given types “cv1 T1” and “cv2 T2”, “cv1 T1” is reference-related to “cv2 T2” if T1 is the same type as T2, or T1 is a base class of T2.

"cv1 T1" here being char const, and "cv2 T2" being int , so **they're not related**. Since they're not, [[dcl.init.ref](https://timsong-cpp.github.io/cppwp/n4659/dcl.init.ref#5.2.2.2)] applies:
>Otherwise, the initializer expression is implicitly converted to a prvalue of type “cv1 T1”. The temporary materialization conversion is applied and the reference is bound to the result.

So the initializer expression ```a``` is converted to a **temporary char const**, which ```b``` refers to.
We then print ```b```, which **refers to our temporary char with the value '0'**.
We then increment the original ```a```, which **importantly does not modify the temporary that b refers to**.
We finally print ```b``` again, which still has the value '0'.
