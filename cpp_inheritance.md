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

## Data members of inherited classes
The following code fails because ```x``` is defined in the child class but called from the parent.
```cpp
#include<iostream>
using namespace std;
 
class Base
{
public:
    void show()
    {
        cout<<" In Base ";
    }
};
 
class Derived: public Base
{
public:
    int x;
    void show()
    {
        cout<<"In Derived ";
    }
    Derived()
    {
        x = 10;
    }
};
 
int main(void)
{
    Base *bp, b;
    Derived d;
    bp = &d;
    bp->show();
    cout << bp->x; // <= fails here
    return 0;
}
```
## Shadowing methods with the same name from parent class
If a derived class writes its own method, then all functions of base class with same name become shadowed, even if signaures of base class functions are different.
In the following, ```int fun()``` shadows ```int fun()``` and ```int fun(int i)```. This is why the compilation fails.
```cpp
#include<iostream>
using namespace std;
 
class Base
{
public:
    int fun()  { cout << "Base::fun() called"; }
    int fun(int i)  { cout << "Base::fun(int i) called"; }
};
 
class Derived: public Base
{
public:
    int fun() {  cout << "Derived::fun() called"; }
};
 
int main()
{
    Derived d;
    d.fun(5); // <= compilation fails here
    return 0;
}
```
On the other hand, we still can use the scope resolution operator ```d.Base::fun(5)``` to call the method in the parent class:
```cpp
int main()  {
    Derived d;
    d.Base::fun(5); // <= compilation no longer fails
    return 0;
}
```

## Object slicing
The following program outputs ```Base class``` and ```Base class``` because the derived class object is sliced off and all the data members inherited will be copied to the base class. This should always be avoided because it gives counter-intuitive results like the following.
```cpp
#include <iostream>
#include<string>
using namespace std;
 
class Base
{
public:
    virtual string print() const
    {
        return "This is Base class";
    }
};
 
class Derived : public Base
{
public:
    virtual string print() const
    {
        return "This is Derived class";
    }
};
 
void describe(Base p)
{
    cout << p.print() << endl;
}
 
int main()
{
    Base b;
    Derived d;
    describe(b);
    describe(d);
    return 0;
}
```

The following is another example of object slicing: it displays ```YX```:
```cpp
#include <iostream>
struct X {
  virtual void f() const { std::cout << "X"; }
};

struct Y : public X {
  void f() const { std::cout << "Y"; }
};

void print(const X &x) { x.f(); }

int main() {
  X arr[1];
  Y y1;
  arr[0] = y1;
  print(y1);
  print(arr[0]);
}
```

Because arr is an array of X, not of pointers to X. When an object of type Y is stored in it, it is converted to X, and we lose the "Y part" of the object.


## Diamond Inheritance
There are two copies of 'a' in DerivedDerived which makes the statement ```cout << a;``` ambiguous. Replacing ```class Derived2 : public Base``` by ```class Derived2 : virtual public Base``` solves the issue.
```cpp
#include<iostream>
using namespace std;
 
class Base
{
protected:
    int a;
public:
    Base() {a = 0;}
};
 
class Derived1:  public Base
{
public:
    int c;
};
 
 
class Derived2:  public Base
{
public:
    int c;
};
 
class DerivedDerived: public Derived1, public Derived2
{
public:
    void show()  {   cout << a;  } // <= Fails here, a is ambiguous
};
 
int main(void)
{
    DerivedDerived d;
    d.show();
    return 0;
}
```

## Virtual access of private members
Lets consider the following:
```cpp
#include <iostream>

class A {
public:
  virtual void f() { std::cout << "A"; }
};

class B : public A {
private:
  void f() { std::cout << "B"; }
};

void g(A &a) { a.f(); }

int main() {
  B b;
  g(b);
}
```
Here, ```B::f()``` is called even though it is private, because access is checked at the call point which is ```A::f()``` (C++ standard [class.access.virt](https://timsong-cpp.github.io/cppwp/n4659/class.access.virt#2))
