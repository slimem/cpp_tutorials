## CPP new and delete

# Difference between new and malloc
* ```new``` is an operator, ```malloc``` is a function.
* ```new``` calls constructor, ```malloc``` doesn't.
* ```new``` returns appropriate pointer, ```malloc``` returns ```void *``` and pointer needs to typecast to appropriate type.

# A dynamic array of object
The following allocates dynamically an array[5] of ints. Please note that the returned type is a pointer to the allocated type (after ```new``` operator)
```cpp
int* foo = new int [5];
delete[] foo
```
Following the same rule, we can allocate an array of pointers to int: The returned type will be a pointer to an array of pointers:
```cpp
int** foo = new int* [5];
delete[] foo;
```

# Delete effects
Deleting a ```NULL``` pointers has no effect, so it is unnecessairy to check for ```NULL``` before ```delete```:
```cpp
int* p = NULL;
delete p;
```
On the other hand, calling ```delete``` twice to an allocated pointer is undefined behaviour:
```cpp
int* p = new int;
delete p;
delete p; // <= Access violation
```
