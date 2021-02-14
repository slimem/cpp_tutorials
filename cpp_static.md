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
