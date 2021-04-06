# CPP Strings

## Modifying a string literal
Let's consider the following example:
```cpp
#include <iostream>

int main() {
  char* a = const_cast<char*>("Hello");
  a[4] = '';
  std::cout << a;
}
```
The program's behaviour is undefined because modifying a string literals is undefined behavior. 
In practice, the implementation can for instance store the string literal in read-only memory, 
such as the code segment. Two string literals might even be stored in overlapping memory. 
So allowing them to be modified is clearly not a good idea.

According to the C++ standard in [[lex.literal](https://timsong-cpp.github.io/cppwp/n4659/lex.string#1)], ```"Hello"``` is **a string literal**:
>A string-literal is a sequence of characters (as defined in [lex.ccon]) surrounded by double quotes, optionally prefixed by R, u8, u8R, u, uR, U, UR, 
>L, or LR, as in "...", R"(...)", u8"...", u8R"\*\*(...)\*\*", u"...", uR"*~(...)*~", U"...", UR"zzz(...)zzz", L"...", or LR"(...)", respectively.

The particularity of storing of string literals is unspecified. 
Thus the result of modification of a string literal is undefined, Also consider that the 
type of the string literal is array of const char, and thus not modifiable.

In [[lex.string](https://timsong-cpp.github.io/cppwp/n4659/lex.string#16)]:
>"Whether all string literals are distinct (that is, are stored 
>in nonoverlapping objects) and (...) is unspecified. \[Note: The effect of attempting to modify a string literal is undefined. - end note\]"
