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

## Operator& on bit-fields
The following program fails because ```operator&``` cannot be applied to bit-fields ([class.bit](https://timsong-cpp.github.io/cppwp/n4659/class.bit#3)) in the C++ standard:
* The address-of operator & shall not be applied to a bit-field, so there are no pointers to bit-fields.
```cpp
#include <iostream>

struct X {
    int var1 : 3;
    int var2;
};

int main() {
    X x;
    std::cout << (&x.var1 < &x.var2); // Compile error
}
```
## Comma operator evaluation
In the following example, the comma operator is evaluated from left to right, so the result is ```20```.
```cpp
#include <iostream>

int main() {
  int a = 10;
  int b = 20;
  int x;
  x = (a, b);
  std::cout << x; // 20
}
```
According to [expr.comma](https://timsong-cpp.github.io/cppwp/n4659/expr.comma#1) in the C++ standard: "A pair of expressions separated by a comma is evaluated left-to-right; the left expression is a discarded-value expression (...) The type and value of the result are the type and value of the right operand".
The same example can be applied to the following:
```cpp
f(a, (t=3, t+2), c);
```
where the second argument of ```f```  has the value 5.
