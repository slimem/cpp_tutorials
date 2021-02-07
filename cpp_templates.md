# CPP Templates

## Static variables inside a template function
For each new type (int, double), a new function is created, and with it a new static variable like the following:
```cpp
count<int>
count<float>
```
Please note that they are both independant, this is why the code below gives the following output:
```
x = 1 count = 0
x = 1 count = 1
x = 1.1 count = 0
```
```cpp
#include <iostream>
using namespace std;
 
template <typename T>
void fun(const T&x)
{
    static int count = 0;
    cout << "x = " << x << " count = " << count << endl;
    ++count;
    return;
}
 
int main()
{
    fun<int> (1); 
    cout << endl;
    fun<int>(1); 
    cout << endl;
    fun<double>(1.1);
    cout << endl;
    return 0;
}
```
