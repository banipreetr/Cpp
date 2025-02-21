### Variadic Templates and Folds


Question: How would you concatenate and print two std::integer_sequence?

Ans:


```cpp
#include <bits/stdc++.h>
using namespace std;

template<typename T, T... ints1, T... ints2>
auto concatenate(std::integer_sequence<T, ints1...> seq1, std::integer_sequence<T, ints2...> seq2) {
    return std::integer_sequence<T, ints1..., ints2...>();
}

template<typename T, T... ints>
void print(std::integer_sequence<T, ints...> seq) {
    ((cout << ints << " "), ...);
    cout << endl;
}

int main() {
    std::integer_sequence<int, 1,2,3> int_seq1;
    std::integer_sequence<int, 4, 5, 6> int_seq2;
    print(concatenate(int_seq1, int_seq2));
}
```
