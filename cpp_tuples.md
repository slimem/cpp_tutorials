# CPP Tuples

## Getting multiple T types from a tuple
Lets consider the following example:
```cpp
#include <iostream>
#include <tuple>

int main()
{
    const auto t = std::make_tuple(42, 3.14, 1337);
    std::cout << std::get<int>(t);
}
```
The compilation fails because ```std::get<T>(tuple)``` can only be used on a tuple which **has exactly one T in it**. The C++ standard says in 
[[tuple.elem](https://timsong-cpp.github.io/cppwp/n4659/tuple.elem#5)]:
>template <class T, class... Types>\
>constexpr const T& get(const tuple<Types...>& t) noexcept;\
> - Requires: The type T occurs exactly once in Types.... Otherwise, the program is ill-formed.
