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
## Template with non-type argument
A template can have a non-type argument (a value). But they must be constants and known at compile type like the following:
```cpp
#include <iostream>
using namespace std;

template < class T, int N = 50 > // default value
T fun(T arr[], int size)
{
    if (size > N)
        cout << "Not possible";
    T max = arr[0];
    for (int i = 1; i < size; i++)
        if (max < arr[i])
            max = arr[i];
    return max;
}

int main()
{
    int arr[] = { 12, 3, 14 };
    cout << fun<int>(arr, 3); // N is default
}
```
Also, non-type arguments are ```const``` and shouldn't be modified. As an example, the following code compilation fails:
```cpp
#include <iostream>
using namespace std;
 
template <int i>
void fun()
{
   i = 20; // <= Fails here, since i is const
   cout << i;
}
 
int main()
{
   fun<10>();
   return 0;
}
```
## Template specialization
Sometimes we want a function template to behave differently with a certain type, this is why the follwing code outputs ```Called specialization: 20```:
```cpp
#include <iostream>
using namespace std;
 
template <class T>
T max (T &a, T &b)
{
    return (a > b)? a : b;
}
 
template <>
int max <int> (int &a, int &b)
{
    cout << "Called Specialization: ";
    return (a > b)? a : b;
}
 
int main ()
{
    int a = 10, b = 20;
    cout << max <int> (a, b);
}
```
## Template metaprogramming
Template metaprogramming is done at compile time. For example, the following computes 2^N. Please note that class members should be static or enums:
```cpp
#include <iostream>

template<int N>
struct function {
    enum { val = 2 * function<N - 1>::val };
    // static const int val = 2 * function<N - 1>::val;
};
// specialization for N = 0
template<>
struct function<0> {
    enum { val = 1 };
    // static const int val = 1;
};

int main() {
    std::cout << function<10>::val << std::endl; // outputs 1024 (2^10)
    return 0;
}
```