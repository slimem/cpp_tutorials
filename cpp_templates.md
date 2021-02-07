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
The same can also happen with template classes like the following:
```cpp
#include <iostream>
using namespace std;
 
template <class T>
class Test
{
private:
    T val;
public:
    static int count;
    Test()  {   count++;   }
};
 
template<class T>
int Test<T>::count = 0;
 
int main()
{
    Test<int> a;
    Test<int> b;
    Test<double> c;
    cout << Test<int>::count   << endl;
    cout << Test<double>::count << endl;
    return 0;
}
```
Which produces the following output (because each Test<T> has its own ```int count```):
```
2
1
```
## Template with ambiguous call
In the following code, ```T max(T x, T y)``` requires that both x and y are the same type. This is why the compilation fails at ```cout << max(3, 7.0) << std::endl;```.
```cpp
#include <iostream>
using namespace std;
 
template <typename T>
T max(T x, T y)
{
    return (x > y)? x : y;
}
int main()
{
    cout << max(3, 7) << std::endl;
    cout << max(3.0, 7.0) << std::endl;
    cout << max(3, 7.0) << std::endl;
    return 0;
}
```
## Template size with static member
Since static variables are not counted in the ```sizeof()```, the code below outputs the following: ```(sizeof(int) = 4, sizeof(char) = 1)```
```
2
8
```
```cpp
#include<iostream>
#include<stdlib.h>
using namespace std;
 
template<class T, class U>
class A  {
    T x;
    U y;
    static int count;
};
 
int main()  {
   A<char, char> a;
   A<int, int> b;
   cout << sizeof(a) << endl;
   cout << sizeof(b) << endl;
   return 0;
}
```
## Template with default value
A template can have a default value, but it needs to be put at the rightmost side (same as functions):
```cpp
#include<iostream>
#include<stdlib.h>
using namespace std;
 
template<class T, class U, class V=double>
class A  {
    T x;
    U y;
    V z;
    static int count;
};
 
int main()
{
   A<int, int> a;
   A<double, double> b;
   cout << sizeof(a) << endl; // 16 (static members are not included)
   cout << sizeof(b) << endl; // 24
   return 0;
}
```
