# CPP Static

## Defining a static variable
The following code compilation will fail because ```a``` is not defined inside class ```B```, so linking will fail with the following error:
```sh
Undefined reference B::a
```
```cpp
#include <iostream>
using namespace std;
 
class A
{
private:
    int x;
public:
    A(int _x)  {  x = _x; }
    int get()  { return x; }
};
 
class B
{
    static A a;
public:
   static int get()
   {  return a.get(); }
};

//A B::a(0); <- The following must be added
 
int main(void)
{
    B b;
    cout << b.get();
   
```

## static members in methods
The following will output ```1 2 3 4``` because the static variable ```count``` is incremented and called 4 times in chained methods ```t.fun().fun().fun().fun()```.
```cpp
#include<iostream>
using namespace std;
 
class Test
{
private:
    static int count;
public:
    Test& fun(); 
};
 
int Test::count = 0;
 
Test& Test::fun()
{
    Test::count++;
    cout << Test::count << " ";
    return *this;
}
 
int main()
{
    Test t;
    t.fun().fun().fun().fun();
    return 0;
}
```
## Default value of static variable
Let's consider the following example:
```cpp
#include <iostream>

int main() {
  static int a;
  std::cout << a;
}
```
This program is guaranteed to output ```0``` since ```a``` is a ```static``` local variable, it is automatically zero-initialized. This would not have happened if we removed the keyword static, making it a non-static local variable.

The C++ standard says in [[basic.start.static](https://timsong-cpp.github.io/cppwp/n4659/basic.start.static#2)]:
>If constant initialization is not performed, a variable with static storage duration (6.7.1) or thread storage duration (6.7.2) is zero-initialized (11.6).

```a``` has static storage duration and is not constant initialized , so it gets zero-initialized.

The C++ standard also says in [[dcl.init](https://timsong-cpp.github.io/cppwp/n4659/dcl.init#6)]:
>To zero-initialize an object or reference of type T means:
> - if T is a scalar type (6.9), the object is initialized to the value obtained by converting the integer literal 0 (zero) to T;

So ```a``` gets initialized to ```0```.
