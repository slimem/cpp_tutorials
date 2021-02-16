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

    std::cout << (int)literal_bin<uint8_t>("0000,1110") << std::endl;

    return 0;
}
```
