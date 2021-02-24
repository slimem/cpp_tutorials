# CPP Constructor Destructor

## Lifetime of automatic storage variables in case of a JUMP
[stmt.jump](https://timsong-cpp.github.io/cppwp/n4659/stmt.jump#2) Transfer back past an initialized variable with automatic storage duration involves 
the destruction of objects with automatic storage duration that are in scope at the point transferred from but not at the point transferred to,
this is why the following program displays ```aAaA```.
```cpp
#include <iostream>

using namespace std;

class A {
public:
  A() { cout << "a"; }
  ~A() { cout << "A"; }
};

int i = 1;

int main() {
label:
  A a;
  if (i--)
    goto label;
}
```
