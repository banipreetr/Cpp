Consider example:

```cpp
#include <bits/stdc++.h>
#include <compare>

struct caseInsensitiveString {
    caseInsensitiveString(std::string v)
    : str(std::move(v))
    {}
    std::string str;
    auto operator<=>(const caseInsensitiveString& other) const {
        if (str.size() < other.str.size()) {
            return std::weak_ordering::less;
        }
        else if (str.size() > other.str.size()) {
            return std::weak_ordering::greater;
        }
        for(int i=0; i<str.size(); ++i) {
            if (std::tolower(str[i]) < std::tolower(other.str[i])) {
                return std::weak_ordering::less;
            }
            else if (std::tolower(str[i]) > std::tolower(other.str[i])) {
                return std::weak_ordering::greater;
            }
        }
        return std::weak_ordering::equivalent;
    }
};

int main() {
    caseInsensitiveString obj1{"Hello"};
    caseInsensitiveString obj2{"heLlO"};

    auto checkEquivalence = [&](auto&& left, auto&& right) -> void {
        auto cmpr = left <=> right;
        if (cmpr == std::weak_ordering::equivalent) {
            std::cout << "Is equivalent\n";
        }
        
    };
    checkEquivalence(caseInsensitiveString{"Hello"}, caseInsensitiveString{"heLlO"});    
}
```
