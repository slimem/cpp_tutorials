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

## Default parameter for overrided method
Lets consider the following example:
```cpp
#include <iostream>

struct A {
    virtual void foo (int a = 1) {
        std::cout << "A" << a;
    }
};

struct B : A {
    virtual void foo (int a = 2) {
        std::cout << "B" << a;
    }
};

int main () {
    A *b = new B;
    b->foo();
}
```
The program outputs ```B1```:
In the first line of main, we create a new B object, with an A pointer a pointing to it.

On the next line, we call ```b->foo()```, where b has the static type A, and the dynamic type B. Since foo() is virtual, the dynamic type of b is used to ensure B::foo() gets called rather than A::foo().

However, which default argument is used for the int a parameter to foo()? [[dcl.fct.default](https://timsong-cpp.github.io/cppwp/n4659/dcl.fct.default#10)] in the standard:
>A virtual function call (§13.3) uses the default arguments in the declaration of the virtual function determined by the static type of the pointer or reference denoting the object. An overriding function in a derived class does not acquire default arguments from the function it overrides.

So ```B::foo()``` is called, but with the default argument from ```A::foo()```, and the output is ```B1```.

## Scope hiding for derived class
Let's consider the following example:
```cpp
#include <iostream>

struct Base {
  void f(int) { std::cout << "i"; }
};

struct Derived : Base {
  void f(double) { std::cout << "d"; }
};

int main() {
  Derived d;
  int i = 0;
  d.f(i);
}
```
The program outputs ```d``` because ```void f(int)``` isn't even in scope, and is not considered for overload resolution at all. When the name ```f``` is introduced in Derived, it hides the name ```f``` that was introduced in Base.
[[basic.scope](https://timsong-cpp.github.io/cppwp/n4659/basic.scope)] in the C++ standard is dedicated to scope. [[basic.scope.hiding](https://timsong-cpp.github.io/cppwp/n4659/basic.scope.hiding#1)] ¶1 and ¶3 has some hints about what's going on in our case:

> 1: "A name can be hidden by an explicit declaration of that same name in a nested declarative region or derived class (§13.2)"
> 3: "The declaration of a member in a derived class (Clause 13) hides the declaration of a member of a base class of the same name; see §13.2"

Both of these refer to [[class.member.lookup](https://timsong-cpp.github.io/cppwp/n4659/class.member.lookup)], so let's explore the details about how the name f is looked up in a class scope C:

[[class.member.lookup](https://timsong-cpp.github.io/cppwp/n4659/class.member.lookup#4)]:
>"If C contains a declaration of the name f, the declaration set contains every declaration of f declared in C that satisfies the requirements of the language construct in which >the lookup occurs.(...) If the resulting declaration set is not empty, the subobject set contains C itself, and calculation is complete."

So when looking for a declaration of f and finding Derived::f, the calculation is complete, and Base is not examined at all. If no f was found in Derived, we would continue with [[class.member.lookup](https://timsong-cpp.github.io/cppwp/n4659/class.member.lookup#5)]:

>"Otherwise (i.e., C does not contain a declaration of f or the resulting declaration set is empty), S(f,C) is initially empty. If C has base classes, calculate the lookup set >for f in each direct base class subobject Bi, and merge each such lookup set S(f, Bi) in turn into S(f, C).

But since Derived does indeed contain a declaration of f, we never get around to looking at Base.

## Call order of non-implicit constructer/copy constructor of an inherited class
In the following example:
```cpp
#include <iostream>
using namespace std;

class A
{
public:
    A() { cout << "A"; }
    A(const A &) { cout << "a"; }
};

class B: public virtual A
{
public:
    B() { cout << "B"; }
    B(const B &) { cout<< "b"; }
};

class C: public virtual A
{
public:
    C() { cout<< "C"; }
    C(const C &) { cout << "c"; }
};

class D:B,C
{
public:
    D() { cout<< "D"; }
    D(const D &) { cout << "d"; }
};

int main()
{
    D d1;
    D d2(d1);
}
```
We get the following output: ```ABCDABCd```.
### Order of constructor call
Let's consider why first ```ABCD``` were displayed. At first, ```d1``` is initialized in the order A B C D. That order is defined by [[class.base.init](https://timsong-cpp.github.io/cppwp/n4659/class.base.init#13)]:

> * First, and only for the constructor of the most derived class (§ 4.5), virtual base classes are initialized in the order they appear on a depth-first left-to-right traversal of the directed acyclic graph of base classes, where “left-to-right” is the order of appearance of the base classes in the derived class base-specifier-list.
> * Then, direct base classes are initialized in declaration order as they appear in the base-specifier-list
> (...)
> * Finally, the compound-statement of the constructor body is executed.

So the output is ABCD.
### Order of copy-constructor call
On the second line, ```d2``` is initialized. But it displays ABCd because an **implicitly-defined copy constructor would have called the copy constructor of its bases**  in [[class.copy.ctor](https://timsong-cpp.github.io/cppwp/n4659/class.copy.ctor#14)]:
> The implicitly-defined copy/move constructor for a non-union class X performs a memberwise copy/move of its bases and members.

But since we provide a user-defined copy constructor, the constructor of its bases were called instead, hence the output ABCd.
