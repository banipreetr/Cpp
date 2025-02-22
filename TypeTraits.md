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

### True Type and False Type

```cpp
using true_type = std::integral_constant<bool, true>;
```

```cpp
using false_type = std::integral_constant<bool, false>;
```


Example:

```cpp
#include <bits/stdc++.h>

using namespace std;

template<size_t N>
struct is_even : std::conditional< (N%2 == 0), std::true_type, std::false_type>::type {};


int main() {
    static_assert(is_even<2>::value);
    static_assert(!is_even<3>::value);
}
```
