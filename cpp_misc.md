# CPP misc

## Scope resolution for global variables
In the following example, the scope resolution operator ```::x``` refers to the global variable so ```1``` is displayed:
```cpp
#include<iostream>
using namespace std;
int x = 1; // <- access with scope resolution operator ::
void fun()
{
    int x = 2;
    {
        int x = 3;
        cout << ::x << endl;
    }
}
int main()
{
    fun();
    return 0;
}
```

## Union values
In CPP, **unions** act like structs (members are public by default) but **all data members share the same memory address** hence the result ```10``` in the following example:
```cpp
#include<iostream>
using namespace std;
 
union A {
  int a;
  unsigned int b;
  A() { a = 10; }
  unsigned int getb() {return b;}
};
 
int main()
{
    A obj;
    cout << obj.getb();
    return 0;
}
```
