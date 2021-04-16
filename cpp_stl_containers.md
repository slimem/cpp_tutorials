# CPP STL Containers

## Vector construction with one size_t parameter
Lets consider the following example:
```cpp
#include <iostream>
#include <vector>

struct Foo
{
    Foo() { std::cout<<"a"; }
    Foo(const Foo&) { std::cout<<"b"; }
};

int main()
{
    std::vector<Foo> bar(5);
}
```
The program outputs ```aaaaa``` because since C++11, ```std::vector``` has a one parameter constructor (+allocator) 
[[vector.cons](https://timsong-cpp.github.io/cppwp/n4659/vector.cons#3)] in the standard:
```cpp
explicit vector(size_type n, const Allocator& = Allocator())
```
Which constructs a vector with ```n``` value-initialized elements. Each value-initialization calls the default ```Foo``` constructor, resulting in the output ```aaaaa```.

The "trick" is, that before C++11, std::vector had a 2 parameter constructor
( + allocator ), which constructed the container with n copies of the second parameter, 
which is defaulted to T().

So this code before C++11 would output ```abbbbb```, because the call would be equivalent to ```std::vector<Foo> bar(5,T())```.

## Map of <bool.int>
Let's consider the following example:
```cpp
#include <iostream>
#include <map>
using namespace std;

int main()
{
  map<bool,int> mb = {{1,2},{3,4},{5,0}};
  cout << mb.size(); 
  map<int,int> mi = {{1,2},{3,4},{5,0}};
  cout << mi.size();
}
```
This program outputs ```13``` because ```std::map``` stores values based on a unique key. The keys for mb are boolean, and 1, 3 and 5 all **evaluate to the same key, true**.
The C++ standard says in [[map.overview](https://timsong-cpp.github.io/cppwp/n4659/map.overview#1)]:
>"A map is an associative container that supports unique keys (contains at most one of each key value)."

The type of mb is ```map<bool,int>```. The key is bool, so the integers 1, 3 and 5 used for initialization are first converted to bool, and **they all evaluate to true**.
The C++ standard also says in [[conv.bool](https://timsong-cpp.github.io/cppwp/n4659/conv.bool#1)]:
>"A prvalue of arithmetic, unscoped enumeration, pointer, or pointer to member type can be converted to a prvalue of type bool. A zero value, null pointer value, or null member >pointer value is converted to false; any other value is converted to true."
