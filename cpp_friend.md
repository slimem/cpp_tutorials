# CPP Friend

## Friend method of a forward defined class
Let's consider the following example:
```cpp
#include <iostream>

class A;

class B {
public:
  B() { std::cout << "B"; }
  friend B A::createB();
};

class A {
public:
  A() { std::cout << "A"; }

  B createB() { return B(); }
};

int main() {
  A a;
  B b = a.createB();
}
```
The compilation fails at ```friend B A::createB();```. To declare ```A::createB()``` a friend of B, the compiler needs to know that that function exists. 
Since it has only seen the declaration of ```A``` so far, not the full definition, it cannot know this.
