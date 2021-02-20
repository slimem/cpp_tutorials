#CPP Long Variables

## Extra long factorial
```cpp
#include <bits/stdc++.h>

using namespace std;

// Complete the extraLongFactorials function below.
void extraLongFactorials(int n) {
    std::vector<int> r {1};
    
    for (int i = 2; i <= n; ++i) {
        for (int j = 0; j < r.size(); ++j) {
            r[j] *= i;
        }
        
        for (int j = 0; j < r.size(); ++j) {
            if (r[j] < 10) continue;
            
            if (j == (r.size() - 1)) {
                r.emplace_back(0);
            }
            
            r[j+1] += r[j] / 10;
            r[j] %= 10;
        }
    }
    for (auto it = r.rbegin(); it != r.rend(); ++it) std::cout << *it;
}

int main()
{
    int n;
    cin >> n;
    cin.ignore(numeric_limits<streamsize>::max(), '\n');

    extraLongFactorials(n);

    return 0;
}
```
