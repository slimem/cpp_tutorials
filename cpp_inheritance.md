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
