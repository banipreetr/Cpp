```cpp
#include <bits/stdc++.h>
#include <compare>

using namespace std;

auto add() {
    return 0;
}

template <typename Arg, typename... Args>
auto add(Arg arg, Args... args) {
    if constexpr (sizeof...(args) == 0) {
        return arg;
    }
    return arg + add(args...);
}

template<typename... Args>
auto add_fold(Args... args) {
    return (args + ...);
}

template<typename... Args>
void print_seq(Args... args) {
    ((std::cout << args << ' '), ...);
    std::cout << std::endl;
}

void print_rec() {
    cout << std::endl;
}

template<typename T, typename... Args>
void print_rec(T arg, Args... args) {
    if constexpr (sizeof...(args) == 0) {
        cout << arg << std::endl;
        return;
    }
    std::cout << arg << " ";
    print_rec(args...);
}

int main() {
    std::integer_sequence<int, 1, 2, 3, 4> seq;
    std::cout << add(1,2,3,4) << endl;
    std::cout << add_fold(1,2,3,4) << endl;
    
    print_seq(1,2,3,4,5,6);
    print_rec(1,2,3,4,5,6);

    
}
```
