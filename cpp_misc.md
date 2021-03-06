# CPP misc

## Scope resolution for global variables
In the following example, the scope resolution operator ```::x``` refers to the global variable so ```1``` is displayed:
```cpp
#include<iostream>
using namespace std;
int x = 1; // <- access with scope resolution operator ::
void fun()
{
    int x = 2;
    {
        int x = 3;
        cout << ::x << endl;
    }
}
int main()
{
    fun();
    return 0;
}
```

## Union values
In CPP, **unions** act like structs (members are public by default) but **all data members share the same memory address** hence the result ```10``` in the following example:
```cpp
#include<iostream>
using namespace std;
 
union A {
  int a;
  unsigned int b;
  A() { a = 10; }
  unsigned int getb() {return b;}
};
 
int main()
{
    A obj;
    cout << obj.getb();
    return 0;
}
```

## Object that can only be created using the operator new
To do this, the destructor must be declared private, this way, the object can only be constructed using a pointer, and freed using a ```friend``` or a member function to the class that calls the destructor.\
In the following example, the **friend** method ```void destructTest(Test*)``` calls the private destructor of the class Test.
```cpp
// Objects of test can only be created using new
class Test
{
private:
    ~Test() {}
friend void destructTest(Test* );
};
 
// Only this function can destruct objects of Test
void destructTest(Test* ptr)
{
    delete ptr;
}
 
int main()
{
    // create an object
    Test *ptr = new Test;
 
    // destruct the object
    destructTest(ptr); // <- Calls the private destructor
 
    return 0;
}
```

## Default value for uninitialized global variables
In Cpp, all the uninitialized global variables are set to ```0```. This is why the following program displays ```3```.
```cpp
#include<iostream>
using namespace std;
 
int x[100];
int main()
{
    cout << x[99] + 3 << endl;
}
```

## Operator& on bit-fields
The following program fails because ```operator&``` cannot be applied to bit-fields ([class.bit](https://timsong-cpp.github.io/cppwp/n4659/class.bit#3)) in the C++ standard:
* The address-of operator & shall not be applied to a bit-field, so there are no pointers to bit-fields.
```cpp
#include <iostream>

struct X {
    int var1 : 3;
    int var2;
};

int main() {
    X x;
    std::cout << (&x.var1 < &x.var2); // Compile error
}
```
## Comma operator evaluation
In the following example, the comma operator is evaluated from left to right, so the result is ```20```.
```cpp
#include <iostream>

int main() {
  int a = 10;
  int b = 20;
  int x;
  x = (a, b);
  std::cout << x; // 20
}
```
According to [expr.comma](https://timsong-cpp.github.io/cppwp/n4659/expr.comma#1) in the C++ standard: "A pair of expressions separated by a comma is evaluated left-to-right; the left expression is a discarded-value expression (...) The type and value of the result are the type and value of the right operand".
The same example can be applied to the following:
```cpp
f(a, (t=3, t+2), c);
```
where the second argument of ```f```  has the value 5.

## Continue statement in a loop
Lets consider the following program:
```cpp
#include <iostream>

int main() {
    int i=1;
    do {
        std::cout << i;
        i++;
        if(i < 3) continue;
    } while(false);
    return 0;
}
```
The program outputs ```1``` because the **continue** statements causes the control to pass to **the end of the loop**, not the beginning. (C++ standard [stmt.cont](https://timsong-cpp.github.io/cppwp/n4659/stmt.cont#1))

## Ambiguity resolution of type conversion and declaration
Lets consider the following example:
```cpp
#include <iostream>

struct X {
    X() { std::cout << "1"; }
    X(const X &) { std::cout << "3"; }
    ~X() { std::cout << "2"; }

    void f() { std::cout << "4"; }

} object; // <- A

int main() {
    X(object); // <- B
    object.f();
}
```
The program outputs ```11422``` because at the line ```X(object)```, someone can easily interpret as:
* Creating a temprary unnamed variable ```object``` OR
* Creating a new variable of type ```X``` named ```object```.
Regarding this subject, the C++ standard says that an expression-statement with a function-style explicit type conversion is considered a declaration ([stmt.ambig](https://timsong-cpp.github.io/cppwp/n4659/stmt.ambig#1))

This way we can explain the output ```11422``` with the following:
* An instance ```object``` of ```X``` is created (A) --> 1
* At ```X(object)``` (B), another instance named ```object``` is created which shadows the previously created ```object``` --> 1
* A normal call to ```f()``` --> 4
* Destruction of the recently-created ```object``` at (B) --> 2
* Destruction of the global ```object``` at (A) --> 2

## Unsequenced access to volatile variable
Lets consider the following example:
```cpp
#include <iostream>

volatile int a;

int main() {
  std::cout << (a + a);
}
```
The program results in an **undefined behaviour**, because the issue here is not the missing initializer of the variable ```a```: it will implicitly be initialized to ```0``` here. But the issue is the access to a twice without sequencing between the accesses. According to the C++ standard in [[intro.execution](https://timsong-cpp.github.io/cppwp/n4659/intro.execution#14)], access of volatile glvalues are side-effects and according to [[intro.execution](https://timsong-cpp.github.io/cppwp/n4659/intro.execution#17)], these two unsequenced side-effects on the same memory location results in **undefined behaviour**.

## Parsing a+++++b token
Let's consider the following example:
```cpp
#include <iostream>

int main() {
 int a = 5,b = 2;
 std::cout << a+++++b;
}
```
The program has a **compilation error**. Some might expect the lexer (including me) to parse the serie of characters ```a+++++b``` as follows: ```a++ + ++b```,
but according to the [maximal munch principle](https://en.wikipedia.org/wiki/Maximal_munch), the lexer will take the longest sequence of characters to produce the next token(with a few exceptions).

The C++ standard says in [[lex.pptoken](https://timsong-cpp.github.io/cppwp/n4659/lex.pptoken#3)]:
> - Otherwise, the next preprocessing token is the longest sequence of characters that could constitute a preprocessing token, even if that would cause further lexical analysis to fail (...).

So, after parsing ```a++```, it will parse ```++``` not just ```+```. The sequence is thus parsed as: ```a ++ ++ + b``` which is ill-formed since post-increment requires a modifiable lvalue but the first post-increment will produce a prvalue, as per the C++ standard in [[expr.post.incr](https://timsong-cpp.github.io/cppwp/n4659/expr.post.incr#1)]:
> - The value of a postfix ++ expression is the value of its operand. (...) The result is a prvalue.

## Integer overflow
Let's consider the following example:
```cpp
#include <iostream>
#include <limits>

int main() {
  int i = std::numeric_limits<int>::max();
  std::cout << ++i;
}
```
The program has **undefined behaviour**: Signed integer overflow is undefined behaviour according to the standard [[expr](https://timsong-cpp.github.io/cppwp/n4659/expr#4)]:
>"If during the evaluation of an expression, the result is not mathematically defined or not in the range of representable values for its type, the behavior is undefined."

Most implementations will just wrap around, so if you try it out on your machine, you will probably see the same as if you had done:
```cpp
std::cout << std::numeric_limits<int>::min();
```
Relying on such undefined behaviour is however not safe. For an interesting example, see this [link](http://stackoverflow.com/questions/7682477/why-does-integer-overflow-on-x86-with-gcc-cause-an-infinite-loop).

## Sizeof an empty string
Let's consider the following example:
```cpp
#include <iostream>

int main() {
    std::cout << sizeof("");
}
```
The program outputs ```1``` because ```""``` is not completely emtpy, but has a terminating ```\0```.
The C++ standard says in [[lex.string](https://timsong-cpp.github.io/cppwp/n4659/lex.string#14)]:
> \0 is appended to every string literal so that programs that scan a string can find its end.

The type of ```""``` is mentionned in [[lex.string](https://timsong-cpp.github.io/cppwp/n4659/lex.string#6)]:
> a string-literal that does not begin with an encoding-prefix is an ordinary string literal, and is initialized with the given characters.
> Ordinary string literals (...) are also referred to as narrow string literals. A narrow string literal has type "array of n const char", where n is the size of the string as defined below

So the type of ```""``` is **array of n const char**. Its size is defined in [[lex.string](https://timsong-cpp.github.io/cppwp/n4659/lex.string#15)]:
>The size of a narrow string literal is the total number of escape sequences and other characters, plus at least one for the multibyte encoding of each universal-character-name, plus one for the terminating \0.

There are no escape sequences, no other characters and no universal-character-names (unicode U... constructs), so the size of the array is just 1 for the implicit terminating \0.

Finally, what does ```sizeof``` return when applied to an array? In the C++ standard, [[expr.sizeof](https://timsong-cpp.github.io/cppwp/n4659/expr.sizeof#2)] says:
>When applied to an array, the result is the total number of bytes in the array. This implies that the size of an array of n elements is n times the size of an element.

Our array is 1 elements long, so it returns 1 times the size of the element. How big is the element? [[expr.sizeof](https://timsong-cpp.github.io/cppwp/n4659/expr.sizeof#2)]:
>izeof(char), sizeof(signed char) and sizeof(unsigned char) are 1

So we have one element with size 1, and the program prints 1.
