# CPP Operator Overloading

## Hiding new and new[] operators
By making an empty private ```new``` and ```new[]``` operators, we can restrict dynamic allocation of objects:
```
