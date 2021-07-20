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

## Different std::string constructors for string_literals and C string
Let's consider the following example:
```cpp
#include <string>
#include <iostream>

int main() {
  using namespace std::string_literals;
  std::string s1("hello world",5); 
  std::string s2("hello world"s,5); 

  std::cout << s1 << s2;
}
```
The program outputs ```hello world```. Even though these constructor invocations look similar for s1 and s2, the one that takes a **C-style string** and the one that takes a **basic_string** behave differently with respect to the second argument. For the former, it means **"until this position"**, but for the latter it means **"from this position"**.

In the constructor taking a **C-style string** (the s1 case) the second argument is treated as how many character to take from the start of the C-style string. In [[string.cons](https://timsong-cpp.github.io/cppwp/n4659/string.cons#11)]:
> basic_string(const charT* s, size_type n, const Allocator& a = Allocator());
> Effects: Constructs an object of class basic_string and determines its initial string value from the array of charT of length n whose first element is designated by s

On the other hand, in the constructor that takes **basic_string** (the s2 case) the second argument is treated as the position to start copying the string from. [[string.cons](https://timsong-cpp.github.io/cppwp/n4659/string.cons#4)]:
> basic_string(const basic_string& str, size_type pos, const Allocator& a = Allocator());
> Effects: Constructs an object of class basic_string and determines the effective length rlen of the initial string value as str.size() - pos, as indicated in Table 57.

Table 57 then goes on to explain that the data of the string starts at pos:
> [the string data] points at the first element of an allocated copy of rlen consecutive elements of the string controlled by str beginning at position pos
