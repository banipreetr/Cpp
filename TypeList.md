An implementation of typelist using metaprogramming:


```cpp
#include <bits/stdc++.h>
using namespace std;

template <typename... Ts>
struct TypeList {
};
// Primary template definition
template <typename... Ts>
struct LargestType;

// Specialization for single type
template<typename T>
struct LargestType<TypeList<T>> {
    using type = T;
};

// Specialization for multiple types
template<typename T, typename... Ts>
struct LargestType<TypeList<T, Ts...>> {
    using Rest = typename LargestType<TypeList<Ts...>>::type;
    using type = std::conditional_t<(sizeof(T) > sizeof(Rest)), T, Rest>;
};



int main() {
    using MyList = TypeList<int, double, long long int>;
    if constexpr(std::is_same_v<typename LargestType<MyList>::type, int>) {
        cout << "It is int\n";
    }
    else if constexpr(std::is_same_v<typename LargestType<MyList>::type, double>) {
        cout << "It is double\n";
    }
    else {
        cout << "It is long long int\n";
    }

    LargestType<TypeList<int>>::type t;
}
```
