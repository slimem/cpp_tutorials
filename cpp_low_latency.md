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
