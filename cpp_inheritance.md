# CPP Inheritance

## Order of constructors and destructors call
From inherited classes, inherited constructors are called from left to right (in the following example ```class Derived : public Base1, public Base2```,
```Base1``` constructor then ```Base2``` then ```Derived```, from left to right). Destructors are called in a way that the last constructed object is the first to be destoryed.
The following examples gives this result:
```
Base1's constructor called
Base2's constructor called
Derived's constructor called
Derived's destructor called
Base2's destructor called
Base1's destructor called
```
```cpp
using namespace std;
class Base1 {
public:
    Base1()
    {
        cout << " Base1's constructor called" << endl;
    }
    ~Base1() {
        cout << " Base1's destructor called" << endl;
    }
};

class Base2 {
public:
    Base2()
    {
        cout << "Base2's constructor called" << endl;
    }
    ~Base2() {
        cout << " Base2's destructor called" << endl;
    }
};

class Derived : public Base1, public Base2 {
public:
    Derived()
    {
        cout << "Derived's constructor called" << endl;
    }
    ~Derived()
    {
        cout << "Derived's destructor called" << endl;
    }
};

int main()
{
    Derived d;
    return 0;
}
```
## Size of multiple inheritance classes with/without virtual
When a class inherits from multiple classes, it also creates two copies of the inherited classes and this is a waste of space:\
The following program outputs ```80```, since ```int arr[10]``` is counted twice:
```cpp
#include<iostream>
using namespace std;
 
class base {
    int arr[10];
};
 
class b1: public base { }; // 40
 
class b2: public base { }; // 40
 
class derived: public b1, public b2 {}; // 40 + 40
 
int main(void)
{
  cout << sizeof(derived);
  return 0;
}
```
On the other hand, when using ```virtual```, the following program outputs ```48``` since ```int arr[10]``` is counted only once, plus 2 vtable pointers:
```cpp
#include<iostream>
using namespace std;
 
class base {
  int arr[10];
};
 
class b1: virtual public base { }; // 40 + vtable = 44
 
class b2: virtual public base { }; // 40 + vtable = 44
 
class derived: public b1, public b2 {}; // 40 + 2 vtables = 48
 
int main(void)
{ 
  cout << sizeof(derived);
  return 0;
} 
```
## Method lookup for inherited classes
The print function is not present in class R. So it is looked up in the inheritance hierarchy upward and calls the first matching function. The following outputs:
```
Inside Q
```
```cpp
#include<iostream>
  
using namespace std;
class P {
public:
   void print()  { cout <<" Inside P"; }
};
  
class Q : public P {
public:
   void print() { cout <<" Inside Q"; }
};
  
class R: public Q { };
  
int main(void)
{
  R r; 
  r.print();
  return 0;
}
```
