# Program start and termination

## argv[argc]
Let's consider the following example:
```cpp
#include <iostream>

int main(int argc, char* argv[]) {
    std::cout << (argv[argc] == nullptr);
}
```
One would expect that this is an **out-of-bounds** array access but this is well defined in [[basic.start.main](https://timsong-cpp.github.io/cppwp/n4659/basic.start.main#2)];
>The value of argv[argc] shall be 0
