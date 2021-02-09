# CPP References

## LValue function return
If a function returns bu reference, it can be used as an lvalue, this is why the following is true and displays ```30```:\
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
