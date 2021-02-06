# CPP Exceptions

## Exception order of Base classes and derived classes
The following code will display ```Caught Base Exception``` because base classes are catched first. This is why derived class should be added first.
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
