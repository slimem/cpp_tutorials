# CPP Parsing no regex

## A fast approach with sscanf
The following allows simple string parsing with sscanf instead of regex. For some cases, this can be faster.
```cpp
#include <iostream>

int main() {
    // create 3 empty strings on the stack to use as parsing result
    char str1[32] {""};
    char str2[32] {""};
    char str3[32] {""};
    // string to be parsed
    std::string source{"STR1_STR2_STR3"};
    
    // make sure they are null terminated
    if (str1[0] == '\0') {
        std::cout << "It is already null terminated\n";
    }
    
    // more on this below
    sscanf(source.c_str(), "%31[^_]_%31[^_]_%31[^X]", str1, str2, str3);

    std::cout << str1 << "/" << std::endl;
    std::cout << str2 << "/" << std::endl;
    std::cout << str3 << "/" << std::endl;
    
    return 0;
}
```
This program prints the following:
```
It is already null terminated
STR1/
STR2/
STR3/
```
The sequence ```%31``` matches at most 31 caracters (not null terminated). Adding ```%31[^_]``` means: **Match at most 31 consecutive caracters, but stop if you encounter a 
_ caracter**.
To match the rest of the string, you can use a normal %s.

## A faster approach with fscanf
While reading large buffers, ```sscanf()``` can be slower because it calls ```strlen()``` internally. A better approach is to parse the ```FILE*``` directly:
```cpp
#include <iostream>

int main() {
   
    char str1[32] {""};
    char str2[32] {""};
    char str3[32] {""};

    // create a file with our input
    FILE* f = fopen("input.txt", "w+");
    fputs("STR1_STR2_STR3", f);

    rewind(f);
    // use the file handler instead with fscanf
    fscanf(f, "%31[^_]_%31[^_]_%s", str1, str2, str3);

    std::cout << str1 << "/" << std::endl;
    std::cout << str2 << "/" << std::endl;
    std::cout << str3 << "/" << std::endl;

    return 0;
}
```
