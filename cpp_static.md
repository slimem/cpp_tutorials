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
