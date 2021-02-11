# CPP Exceptions

## Exception order of Base classes and derived classes
The following code will display ```Caught Base Exception``` because base classes are catched first. This is why derived class should be added first.\
In MSVC, the following warning is shown: ```warning C4286: 'Derived': is caught by base class ('Base')```.

```cpp
#include<iostream>
using namespace std;
 
class Base {};
class Derived: public Base {};
int main()
{
   Derived d;
   try {
       throw d;
   }
   catch(Base b) {
        cout<<"Caught Base Exception";
   }
   catch(Derived d) {
        cout<<"Caught Derived Exception";
   }
   return 0;
}
```

## Implicit type conversion
In the following example, ```default exceptionn``` is called because implicit conversion from char ```'a'``` to ```int``` does not occurr.
```cpp
#include <iostream>
using namespace std;
 
int main()
{
    try
    {
       throw 'a';
    }
    catch (int param)
    {
        cout << "int exceptionn";
    }
    catch (...)
    {
        cout << "default exceptionn";
    }
    cout << "After Exception";
    return 0;
}
```
## Order of ALL catch block
The compilation fails for the following code because the ```catch( ... )``` must be the last catch block.
```cpp
#include <iostream>
using namespace std;
 
int main()
{
    try
    {
       throw 10;
    }
    catch (...)
    {
        cout << "default exceptionn";
    }
    catch (int param)
    {
        cout << "int exceptionn";
    }
 
    return 0;
}
```
## Rethrowing an exception inside a nested try catch
Throwing an exception inside a nested is allowed and can be useful when the nested catch block handles some part of the exception and then rethrows it.\
The following code produces the following output:
```
Inner Catch
Outer Catch
```
```cpp
#include <iostream>
using namespace std;
 
int main()
{
    try
    {
        try
        {
            throw 20;
        }
        catch (int n)
        {
            cout << "Inner Catch\n";
            throw;
        }
    }
    catch (int x)
    {
        cout << "Outer Catch\n";
    }
}  
```
## Order of Constructor and Destructor call
When an object is created inside a try block, the destructor for the object is called before the control is transferred to catch block. This is why the following code produces this output:
```
Constructing an object of Test
Destructing an object of Test
Caught 10
```
```cpp
#include <iostream>
using namespace std;
 
class Test {
public:
   Test() { cout << "Constructing an object of Test " << endl; }
  ~Test() { cout << "Destructing an object of Test "  << endl; }
};
 
int main() {
  try {
    Test t1;
    throw 10;
  } catch(int i) {
    cout << "Caught " << i << endl;
  }
}
```
## Exception thrown inside constructor
When an exception is thrown inside a constructor, it makes the object considered as "incomplete" (not constructed) so its destructor won't be called. Also, destructors are called in reverse order, thus giving the following output:
```
Constructing object number 1
Constructing object number 2
Constructing object number 3
Constructing object number 4
Destructing object number 3
Destructing object number 2
Destructing object number 1
Caught 4
```
```cpp
#include <iostream>
using namespace std;
 
class Test {
  static int count;
  int id;
public:
  Test() {
    count++;
    id = count;
    cout << "Constructing object number " << id << endl;
    if(id == 4)
       throw 4;
  }
  ~Test() { cout << "Destructing object number " << id << endl; }
};
 
int Test::count = 0;
 
int main() {
  try {
    Test array[5];
  } catch(int i) {
    cout << "Caught " << i << endl;
  }
}
```
## Abnormal termination of uncaught exception
When an exception is thrown and is not caught, the program terminates abnormally (Unhandled exception).
```cpp
#include <iostream>
using namespace std;
 
int fun() throw (int)
{
    throw 10;
} // <= Program terminates here
 
int main() {
 
  fun();
 
  return 0;
}
```
## Exception list
An exception is thrown even if the exception list is not set. But as civilized programmers, we should set an exception list.
```cpp
#include <iostream>
using namespace std;
 
// Ideally it should have been "int fun() throw (int)"
int fun()
{
    throw 10;
}
 
int main()
{
    try
    {
        fun();
    }
    catch (int )
    {
        cout << "Caught";
    }
    return 0;
}
```
