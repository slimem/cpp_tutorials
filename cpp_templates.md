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

## Template recursion type deduction
Lets consider the following example:
```cpp
#include <iostream>

template<typename T>
T sum(T arg) { // case 1
    return arg;   
}

template<typename T, typename ...Args>
T sum(T arg, Args... args) {    // case 2
    return arg + sum<T>(args...);
}

int main() {
    auto n1 = sum(0.5, 1, 0.5, 1); // displays 3
    auto n2 = sum(1, 0.5, 1, 0.5); // displays 2
    std::cout << n1 << n2;
}
```
In the example above, when calling sum, the function template calls itself **recursively** until it hits the base case ```T sum(T arg)```, so we can say that
```sum(a,b,c,d)``` is implemented as ```sum(a) + sum(b,c,d)``` and so on and so forth.\
BUT since ```T``` was not specified, it is deducted from the first argument ```T``` which is **double** in ```auto n1 = sum(0.5, 1, 0.5, 1);``` and **int** in ```auto n2 = sum(1, 0.5, 1, 0.5);```.\

However, when sum calls itself, it explicitly specifies T (it does ```sum<T>(args...)```). So ```T``` is only deduced on the initial call to sum, and this ```T``` is used for all the subsequent recursive calls, no matter the type of the arguments along the way, this is why n1 is considered ```(0.5 + (1 + (0.5 + (1)))) == 3``` and n2 is ```(0 + (1 + (0 + (1)))) == 2```.

## Template class definition lookup
Lets consider the following example:
```cpp
#include <iostream>

void f() {
    std::cout << "1";
}

template<typename T>
struct B {
    void f() {
        std::cout << "2";
    }
};

template<typename T>
struct D : B<T> {
    void g() {
        f();
    }
};

int main()
{
    D<int> d;
    d.g();
}
```
The following program displays ```1``` because according to the standard, [temp.dep](https://timsong-cpp.github.io/cppwp/n4659/temp.dep#3), "in the definition of a class or class template, if a base class depends on a template-parameter, the base class scope is not examined during unqualified name lookup either at the point of definition of the class template or member or during an instantiation of the class template or member". So when the compiler sees a call to a function f() inside g() it should choose the one from the global scope, and the program should output ```1```.

## Template metaprogramming
### Computing 2^N
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
### Computing factorial
The following computes the factorial of an int and should return ```120```.
```cpp
#include <iostream>

template<int N>
struct factorial {
    enum { value = N * factorial<N-1>::value };
};

template<>
struct factorial<0> {
    enum { value = 1 };
};

int main() {
    std::cout << factorial<5>::value << std::endl;
    return 0;
}
```

## Template Matching with specialization (example 1, easy)
Let's consider the following example:
```cpp
#include <iostream>

template<class T>
void f(T) { std::cout << 1; }

template<>
void f<>(int*) { std::cout << 2; }

template<class T>
void f(T*) { std::cout << 3; }

int main() {
    int *p = nullptr; 
    f( p );
}
```
This program displays ```3``` because ```f()``` is overloaded by two function templates ```void f(T)``` and ```void f(T*)```. Note that overload resolution only considers the function templates, not the explicit specialisation ```void f<>(int*)```! For overload resolution, first we deduce the template arguments for each function template, and get ```T = int *``` for the first one, and ```T = int``` for the second.

Both function templates are viable, but which one is best? According to [over.match.best](https://timsong-cpp.github.io/cppwp/n4659/over.match.best#1), a function template is a better match than another function template if it's more specialised. So ```T = int *``` is more specialized.

But since the specialization ```void f<>(int*)``` comes after ```void f(T)``` and according to [temp.expl.spec](https://timsong-cpp.github.io/cppwp/n4659/temp.expl.spec#3),  the overloaded function selects ```void f(T*)``` and displays ```3```.

## Template Matching with specialization (example 2, hard)
Let's consider the following example:
```cpp
#include <iostream>

using namespace std;

template<typename T>
void f(T) {
    cout << 1;
}

template<>
void f(int) {
    cout << 2;
}

void f(int) {
    cout << 3;
}

int main() {
    f(0.0);
    f(0);
    f<>(0);
}
```
The program outputs ```132```.

Here we have three calls to ```f```. Which function is selected for each call? [[temp.over](https://timsong-cpp.github.io/cppwp/n4659/temp.over#1)] is surprisingly readable:
>A function template can be overloaded either by (non-template) functions of its name or by (other) function templates of the same name.

The function template ```f``` is in our case overloaded by the non-template ```void f(int)```. It goes on:

>When a call to that name is written [...], template argument deduction [is] performed for each function template to find the template argument values (if any) that can be used with that function template to instantiate a function template specialization that can be invoked with the call arguments

So in the tree cases above, we first do template argument deduction, giving ```double```, ```int``` and ```int```, respectively.

Then:
>For each function template, if the argument deduction and checking succeeds, the template-arguments [...] are used to synthesize the declaration of a single function template specialization which is added to the candidate functions set to be used in overload resolution

So in all the three cases above, a specialization of the function template f is added to the candidates to overload resolution, with T equal to double, int and int, respectively.

Now look at each call:
### f(0.0)
For ```f(0.0)```, T is deduced as double, and a specialization void f(double) is added as a candidate for overload resolution. The other candidate is the non-template function void f(int). The one taking a double is a better match, and ```1``` is printed.

### f(0)
For f(0), T is deduced as int, and a specialization void f(int) is added as a candidate for overload resolution. The other candidate is the non-template function void f(int). Both are equally good matches, so is the program ill-formed?\
In the C++ standard, [[over.match.best](https://timsong-cpp.github.io/cppwp/n4659/over.match.best#1)] says:
>F1 is defined to be a better function than another viable function F2 if [...]
> - F1 is not a function template specialization and F2 is a function template specialization

void ```f(int)``` is not a function template specialization, and ```template<> void f(int)``` is, so the former is selected, and ```3``` is printed.

### f<>(0)
Finally we get to f<>(0). <> is an (empty) template argument list, which can be specified for template functions, but not for non-template functions. So in this case, the non-template function is not an option, we explicitly
ask for the function template. This is also described in a note in [[temp.arg.explicit](https://timsong-cpp.github.io/cppwp/n4659/temp.arg.explicit#4)]:

>An empty template argument list can be used to indicate that a given use refers to a specialization of a function template even when a non-template function is visible that would otherwise be used.

T is deduced to ```int```, the specialization for int is the only candidate, and ```2``` is printed.
