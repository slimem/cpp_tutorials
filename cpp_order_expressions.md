# CPP order of expressions

## Precedence, evaluation order and Short-circuiting
Let's consider the following example:
```cpp
#include <iostream>

int main() {
    int I = 1, J = 1, K = 1;
    std::cout << (++I || ++J && ++K);
    std::cout << I << J << K;
}
```
This program outputs ```1211```, but the question here, which will get incremented and which will not?\
This depends on **operator precedence**, evaluation order and short-circuting that are defined in C++ standard in [[expr.log.or](https://timsong-cpp.github.io/cppwp/n4659/expr.log.or)] 

First, precedence comes into play: Operator precedence is not actually
spelled out anywhere, but you can infer from the syntax below that && (logical-and-expression) has higher precedence than || (logical-or-expression):
>logical-or-expression:
> logical-and-expression
> logical-or-expression || logical-and-expression

So ```(++I || ++J && ++K)``` will be equal to ```(++I || (++J && ++K))```.

Next, evaluation order and short-circuiting:
>|| guarantees left-to-right evaluation; moreover, the second operand is not evaluated if the first operand evaluates to true.

So we first evaluate ```++I```, which is ```2```. This evaluates to true, so the second operand ```(++J && ++K)``` is never evaluated, and J and K do not get incremented.

The result of the entire expression is **true**, and 1 is printed. We then print I, J and K which now hold 2,1 and 1, respectively.
