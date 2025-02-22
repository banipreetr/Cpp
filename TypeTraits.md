## Integral Constant

```cpp
#include <bits/stdc++.h>
#include <compare>

using namespace std;

template<size_t N>
struct factorial : public std::integral_constant<int , N * factorial<N-1>::value> {};

template<>
struct factorial<0> : public std::integral_constant<int, 1> {};

int main() {
    cout << factorial<5>::value << endl;
}
```
