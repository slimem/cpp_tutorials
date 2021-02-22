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

## Object that can only be created using the operator new
To do this, the destructor must be declared private, this way, the object can only be constructed using a pointer, and freed using a ```friend``` or a member function to the class that calls the destructor.\
In the following example, the **friend** method ```void destructTest(Test*)``` calls the private destructor of the class Test.
```cpp
// Objects of test can only be created using new
class Test
{
private:
    ~Test() {}
friend void destructTest(Test* );
};
 
// Only this function can destruct objects of Test
void destructTest(Test* ptr)
{
    delete ptr;
}
 
int main()
{
    // create an object
    Test *ptr = new Test;
 
    // destruct the object
    destructTest(ptr); // <- Calls the private destructor
 
    return 0;
}
```

## Default value for uninitialized global variables
In Cpp, all the uninitialized global variables are set to ```0```. This is why the following program displays ```3```.
```cpp
#include<iostream>
using namespace std;
 
int x[100];
int main()
{
    cout << x[99] + 3 << endl;
}
```
