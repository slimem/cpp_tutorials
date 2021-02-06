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
  
```
