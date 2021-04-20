# CPP Types

## Char types
Let's consider the following example:
```cpp
#include <iostream>
#include <type_traits>
 
int main() {
    if(std::is_signed<char>::value){
        std::cout << std::is_same<char, signed char>::value;
    }else{
        std::cout << std::is_same<char, unsigned char>::value;
    }
}
```
The program outputs ```0``` because ```char```, ```signed char``` and ```unsigned char``` are
three distinct types. The C++ standard says in [[basic.fundamental](https://timsong-cpp.github.io/cppwp/n4659/basic.fundamental#1)]:
>Plain char, signed char, and unsigned char are three distinct types (...).
