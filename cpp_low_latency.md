# Low Latency CPP

## Slowpath Removal in the hotpath
Lets consider the following: I need to check for A, B and C then call ```sendOrderToExchange``` if everything goes well. This has some incovenients:
* Pollutes the instruction cache with branching
* The prefetcher will have some hard time
* More jumping
* More instructions before ```sendOrderToExchange``` if the previous error checks are inlined
```cpp
if (checkErrorA())
    handleA();
if (checkErrorB())
    handleB();
if (checkErrorC())
    handleC();
else
    sendOrderToExchange();
```
To solve this, reduce branching as much as possible, do not inline error handling, and try to put conditions on literal values: A CPU is very good at this!
```cpp
int64_t errorFlags;
if (!errorFlags)
    sendOrderToExchange();
else
    HandleError(errorFlags); // <- Do not inline this!
```
