# CPP Lambdas

## lambda with static local variable
In the following example:
```cpp
#include <iostream>

void f(int a = []() { static int b = 1; return b++; }())
{
   std::cout << a;
}

int main()
{
   f();
   f();
}
```
The above example outputs ```12``` but many parameters come into play, so let's consider them one by one.
### 1 - Is the lambda evaluated once or twice?
```f()``` has a parameter with a default argument which then calls 
a **lambda**. According to the C++ standard in [[dct.fct.default](https://timsong-cpp.github.io/cppwp/n4659/dcl.fct.default#9)],
 the lambda is evaluated at each ```f()``` call 
:
>A default argument is evaluated each time the function is called with no argument for the corresponding parameter.

### 2 - Is the same static variable used in both calls?
The static variable ```b``` is post-incremented and returns its value that will be assigned to ```a``` so here we have two cases:
* If ```b``` from both calls are the same, we will first get ```1``` then ```2```.
* If ```b``` from both calls are different, we will get ```1``` then ```1```.

According to the C++ standard in [[expr.prim.lambda.closure](https://timsong-cpp.github.io/cppwp/n4659/expr.prim.lambda.closure#1)]:
>The type of a lambda-expression (which is also the type of the closure object) is a unique, unnamed non-union class type, called the closure type.

So there's only ONE unique closure type for this expression, meaning that the ```static int b``` is the same for both calls, since it is a **static 
local variable in the function call operator of the closure type**, hence the output ```12```.

## Implicit capture
Let's consider the following example:
```cpp
#include <iostream>

int a = 1;

int main() {
    auto f = [](int b) { return a + b; };

    std::cout << f(4);
}
```
The program outputs ```5```.
It is clear that ```a``` is not captured explicitly, so the question is whether a should be captured implicitly (and if yes, then a default capture has to be specified). [[expr.prim.lambda.capture](https://timsong-cpp.github.io/cppwp/n4659/expr.prim.lambda.capture#7)] says:
>A lambda-expression with an associated capture-default that does not explicitly capture \*this or a variable with automatic storage duration \[...\], is said to implicitly >capture the entity (i.e., \*this or a variable) if the compound-statement:
> - odr-uses the entity (in the case of a variable),
> - \[...\]

The use of ```a``` constitutes an **odr-use**. But since ```a``` has static storage duration rather than automatic storage duration, it is not implicitly captured.
Since ```a``` is **neither explicitly nor implicitly captured**, ```a``` in the lambda expression simply **refers to the global variable a**.

(Note: It is also disallowed to capture a explicitly, because of [[expr.prim.lambda.capture](https://timsong-cpp.github.io/cppwp/n4659/expr.prim.lambda.capture#4)]):
>The identifier in a simple-capture is looked up using the usual rules for unqualified name lookup; each such lookup shall find an entity. An entity that is designated by a >simple-capture is said to be explicitly captured, and shall be \*this (when the simple-capture is “this” or “* this”) or a variable with automatic storage duration declared in >the reaching scope of the local lambda expression.

