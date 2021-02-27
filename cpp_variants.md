# CPP variants

## Variant default index
The following displays ```0```:
```cpp
#include <variant>
#include <iostream>
 
using namespace std;

int main() {
   variant<int, double, char> v;
   cout << v.index();
}
```

### Explanantion
std::variant can hold a value of any one of its alternative types, or no value. To refer to the alternative types, it uses an index i.
[variant.ctor](https://timsong-cpp.github.io/cppwp/n4659/variant.ctor#1) in the standard C++:

let i be in the range [0, sizeof...(Types)), and Ti be the ith type in Types....

In our case, T0 means int, T1 means double, and T2 means char.

Now what happens if you define a variant without initializing it with a certain type? The default constructor will pick the type T0, in our case int, and value-initialize it.
[variant.ctor](https://timsong-cpp.github.io/cppwp/n4659/variant.ctor#2)
```cpp
constexpr variant() noexcept
```

Constructs a variant holding a value-initialized value of type T0

Finally, we call ```index()``` and print the result. ```index()``` returns the index of the type of the contained value. The contained value is an int, aka T0, so 0 is returned.
[variant.status](https://timsong-cpp.github.io/cppwp/n4659/variant.status#3):
```cpp
constexpr size_t index() const noexcept;
```
_Effects+: If [it doesn't contain a value], returns ```variant_npos```. Otherwise, returns the zero- based index of the alternative of the contained value.
