## Basic

From using virtual, we can move to using templates to define virtual functions.
Example:

```cpp
#include <bits/stdc++.h>
using namespace std;


template <typename D>
struct B {
    void f() {
        static_cast<D*>(this)->f();
    }
};

struct D1 : public B<D1>
{
    void f() {
        cout << "Derived 1\n";
    }
};


struct D2 : public B<D2>
{
    void f() {
        cout << "Derived 2\n";
    }
};



int main() {
   B<D1>* obj1 = new D1();
   B<D2>* obj2 = new D2();

    obj1->f();
    obj2->f();

}
```


That way, we can reduce the time overhead taken by virtual functions by using CRTP.
