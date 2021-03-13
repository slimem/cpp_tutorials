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
