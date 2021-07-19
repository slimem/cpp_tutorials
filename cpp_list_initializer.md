# CPP List Initializer

## Construction of an initializer list
Let's consider the following example:
```cpp
#include <initializer_list>
#include <iostream>

class C {
public:
    C() = default;
    C(const C&) { std::cout << 1; }
};

void f(std::initializer_list<C> i) {}

int main() {
    C c;
    std::initializer_list<C> i{c};
    f(i);
    f(i);
}
```
The program displays ```1```, because of the following:
### Construction of an initializer list

In the C++ standard, [[dcl.init.list](https://timsong-cpp.github.io/cppwp/n4659/dcl.init.list#5)] explains construction of initializer_list:
> An object of type std::initializer_list\<E\> is constructed from an initializer list as if the implementation generated and materialized (7.4) a prvalue of type **array of N const E**, where N is the number of elements in the initializer list.
> Each element of that array is copy-initialized with the corresponding element of the initializer list, and the std::initializer_list\<E\> object is constructed to refer to that array.

So when we construct the initializer list ```i```, we can imagine a temporary array of one 
```C``` being created, where the element is copy-initialized. Since **c is an lvalue**, a copy is made (not for instance a move), and **1 is printed**.

Then, ```f``` is called twice, taking the freshly created initializer_list by value. So a copy of the initializer_list is made for each call. 
Does that make a copy of the elements in the initializer list, as it would when taking for instance a vector or a list by value? 
**No**. As we saw at the end of the quote above:
>the std::initializer_list\<E\> object is constructed to refer to that array.

So the initializer_list only refers to that array, it doesn't have a copy of it. 
A note in [[initializer_list.syn](https://timsong-cpp.github.io/cppwp/n4659/initializer_list.syn#1)] spells it out verbatim:
>Copying an initializer list does not copy the underlying elements.

The key takeaway is: If you write an initializer list constructor for your class, never move the elements out of the initializer list. 
Even if you took it by value, you were not passed an exclusive copy of the elements, which might still be used by others.

## Constructor with list initializer
Let's consider the following example:
```cpp
#include <initializer_list>
#include <iostream>

struct A {
  A() { std::cout << "1"; }

  A(int) { std::cout << "2"; }

  A(std::initializer_list<int>) { std::cout << "3"; }
};

int main(int argc, char *argv[]) {
  A a1;
  A a2{};
  A a3{ 1 };
  A a4{ 1, 2 };
}
```
The program outputs ```1133```. Let's explain the outputs one by one:
### First line a1 (output = 1)
In the first line in ```main()```, ```a1``` is default initialized, as described in [[dcl.init](https://timsong-cpp.github.io/cppwp/n4659/dcl.init#12)].
### Second line a2{} (output = 1)
In the second line, ```a2``` doesn't actually use the ```initializer_list``` constructor with a list of zero elements, but the default constructor. In [dcl.init.list](https://timsong-cpp.github.io/cppwp/n4659/dcl.init.list#3)]:
> List-initialization of an object or reference of type T is defined as follows:
> * (...)
> * Otherwise, if the initializer list has no elements and T is a class type with a default constructor, the object is value-initialized.
> * Otherwise, if T is a specialization of std::initializer_list, the object is constructed as described below.
### Third and fourth lines a3{1} and a4{1,2} (output = 3):
In the third and fourth lines, ```a3``` and ```a4``` constructors are chosen in overload resolution, as described in [[over.match.list](https://timsong-cpp.github.io/cppwp/n4659/over.match.list)]:
>When objects of non-aggregate class type T are list-initialized (...), overload resolution selects the constructor in two phases:
> * Initially, the candidate functions are the initializer-list constructors ([[dcl.init.list](https://timsong-cpp.github.io/cppwp/n4659/dcl.init.list)]) of the class T and the argument list consists of the initializer list as a single argument.
> * If no viable initializer-list constructor is found, overload resolution is performed again, where the candidate functions are all the constructors of the class T and the argument list consists of the elements of the initializer list.

Initializer list constructors **are greedy**, so even though ```A(int)``` constructor is available, the standard mandates that initializer_list\<int\> is prioritized, and if - and only if - it's not available, the compiler is allowed to look for other constructors. (This is why it is not recommended to provide a constructor that ambiguously overloads with an initializer_list constructor. See the answer to #4 in http://herbsutter.com/2013/05/09/gotw-1-solution/ )
