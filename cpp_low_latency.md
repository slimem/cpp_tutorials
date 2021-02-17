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

## Use templates instead of virtual functions
In a hotpath, virtual funtions are considered a little bit slow since the ```vtable``` pointer indirection can cause a cache miss.
```cpp
struct OrderSenderA {
    void sendOrder() { // <- Same name as B, but a totally different class
    ...
    }
};

struct OrderSenderB {
    void sendOrder() { // <- Same name as A, but a totally different class
    ...
    }
};

template <typename T>
struct OrderManager : public IOrderManager {
    void mainLoop final {
        _orderSender.sendOrder(); // <- Will be done at compile time
    };
    
    T _orderSender;
};

std::unique_ptr<IOrderManager> factory(const Config& conf) {
    if (conf.useA())
        return std::make_unique<OrderManager<OrderSenderA>>();
    else if (conf.useB())
        return std::make_unique<OrderManager<orderSenderB>>();
    else
        throw;
}

int main() {
    auto manager = factory(config);
    manager.mainLoop();
}
```
## Exception impact on performance
Exceptions have no impact on performance if they **are not thrown**. Otherwise, the next step afer throwing an exception is logging which is quite expensive.
## Branch reduction in the hotpath
Avoid if statements in the hotpath, use a templated approach.\
Consider this code:
```cpp
enum class Side { Buy, Sell };

void runLogic (Side side) {
    const float orderPrice = CalcPrice(side, fairValue, credit);
    chechRiskLimits(side, orderPrice);
    sendOrder(side, orderPrice);
}

float CalcPrice(Side side, float value, float credit) {
    return side == Side::Buy? value - credit : value + credit; // branching at runtime
}
```
The branching can be replaced by template specialization, that can be evaluated at compile time:
```cpp
template <>
void runLogic<Side::Buy>() {
    float orderPrice = CalcPrice<Side::Buy>(fairValue, credit);
    checkRiscLimits<Side::Buy>(orderPrice);
    sendOrder<Side::Buy>(orderPrice);
}

template<>
void calcPrice<Side::Buy>(float value, float credit) { // Template specialization
    return value - credit;
}

template<>
void calcPrice<Side::Sell>(float value, float credit) { // Template Specialization
    return value + credit;
}

```

## Data lookups
One cache line size is **64 bytes** on intel CPUs, so accessing to one member means that the whole structure will be loaded into the cache. So if you need data that seems unrelated to the structure below, you can put it: it will seem ugly, but you get it for free!
```cpp
struct object {
    int32_t id;
    char names[4];
    int16_t multiplier;
    int16_t var; // unrelated, but will be loaded to the cache line when accessing any member of object
};
```
