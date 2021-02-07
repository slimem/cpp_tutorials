# CPP const keyword

## Pointer to const value
```const char* p = "hello"``` means that ```p``` points to a constant value in memory. The following example prints ```c```:
```cpp
#include <iostream>
using namespace std;
int main()
{
    const char* p = "12345"; // <= a pointer to a const array
    const char **q = &p; // <= a double pointer to the previous const array
    *q = "abcde"; // <= this is allowed since we are not changing the data
                  // but pointing to new data
    const char *s = ++p; // a pointer to the pre-incremented caracter b
    p = "XYZWVU"; // p points to data "XYZWVU" 
    cout << *++s; // prints c (preincrement)
    return 0;
}
```
## Const keyword after or before the variable type?
In the following example, ```const``` keyword is placed after ```int``` and it compiles with no error. The placement of ```const``` keyword is not important but it is preferred to put it before the variable type:
```cpp
#include <iostream>
int const s=9;
int main()
{
    std::cout << s;
    return 0;
}
```
