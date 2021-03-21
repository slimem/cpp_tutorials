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
