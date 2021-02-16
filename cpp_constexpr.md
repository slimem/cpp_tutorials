# CPP Constexpr

## Constexpr 14 binary literal
The following is a compile-type implementation of a binary literal in C++14. I know that C++14 already has literals such as ```0b11110000``` but it's a good example to gasp how code is evaluated at compile time.
```cpp
#include<iostream>

template <typename T = std::uint32_t>
constexpr T literal_bin(const char* t) {

    T value = 0;
    std::size_t size = 0;

    for (std::size_t i = 0; t[i] != '\0'; ++i) {

        if (size >= std::numeric_limits<T>::digits) { // returns the number of representation digits
            throw std::overflow_error("Too many bits\n");
        }

        switch (t[i]) {
        case ',':                               break; // do nothing
        case '0': value = value * 2; ++size;       break; // multiply actual value by two
        case '1': value = value * 2 + 1; ++size;   break; // multiply by two and add one
        default: throw std::domain_error("Only '0' '1' and ',' can be used");
        }
    }
    return value;
}

int main() {

    static_assert(literal_bin<uint8_t>("0000,1110") == 0xE, "WRONG EXPECTED VALUE");
    static_assert(literal_bin<uint32_t>("0000,0000,1111,1111") == 0xFF, "WRONG EXPECTED VALUE");
    
    constexpr auto x = literal_bin<uint16_t>("1111,1111"); // evaluated at compile time
    auto a = literal_bin<uint32_t>("0000,0000,1111,1111"); // evaluated at runtime

    std::cout << (int)literal_bin<uint8_t>("0000,1110") << std::endl;

    return 0;
}
```
If we consider the following from the previous example:
```cpp
constexpr auto x = literal_bin<uint16_t>("1111,1111"); // evaluated at compile time
auto a = literal_bin<uint32_t>("0000,0000,1111,1111"); // evaluated at runtime
```
We can say that a ```constexpr``` function can be evaluated both at compile time and runtime: It only depends on the return variable type:
* If the return variable type is ```constexpr```, the function is evaluated at compile time.
* For all other types (const and non const) the function is evaluated at runtime. Please note that a ```const``` variable can be initialized at runtime, this is why it forces the constexpr function to run also at runtime.

# Forcing a constexpr to only run at compile time
One of the hacks that forces a ```constexpr``` function to run at compile time is to cause a deliberate compile error at runtime: This way we can be sure that it can only be executed at compile time.
It can be done by **putting an unresolved symbol in a throw** like the following:
```cpp

extern const char* compile_invoked_at_runtime; // <- An unresolved extern variable 

template <typename T = std::uint32_t>
constexpr T literal_bin(const char* t) {

    T value = 0;
    std::size_t size = 0;

    for (std::size_t i = 0; t[i] != '\0'; ++i) {

        if (size >= std::numeric_limits<T>::digits) { // returns the number of representation digits
            throw std::overflow_error("Too many bits\n");
        }

        switch (t[i]) {
        case ',':                               break; // do nothing
        case '0': value = value * 2; ++size;       break; // multiply actual value by two
        case '1': value = value * 2 + 1; ++size;   break; // multiply by two and add one
        default: throw std::domain_error(compile_invoked_at_runtime); // <- only evaluated at runtime (linker)
        }
    }
    return value;
}
```
### How does it work?
The throw is only evaluated at runtime, so it is skipped at compile time. If we have a runtime implementation, it should evaluate everything!




