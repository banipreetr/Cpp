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

### Problem with CRTP

Size of Base Class cannot depend on Derived class. For example, the following code will not compile:

```cpp
template <typename D>
struct B {
    using T = D::T;
    T* ptr_;
};

struct D : public B<D> {
    using T = int;
};
```

This assumes that for type `D` in `B`, the there exists `D::T`, but D is not defined yet, and it depends on `B` itself. So `D` is an incomplete type and you cannot use an incomplete type like that.
Anything that affects the size of the class must be fully declared. In this case, for class `B`, type `D` needs to be fully declared, but it is kind of an incomplete type.
Note, the body of a templated function is not instantiated until its called. Also, if you have a reference of the types and pointers (the size of which are kind of fixed and helps the compiler determine the size of the Base Class), then its completely okay!!


Note, the following code will not compile:


```cpp
template<typename D>
struct B  {
    D::value_type get() {
        return static_cast<D*>(this)->get();
    }
};


struct D : public B<D> {
    using value_type = int;
    value_type get() {
        return 6;
    }
};
```

However, if you use `auto` instead and let the compiler deduce the type, then it should be fine. So, the following will compile:

```cpp
template<typename D>
struct B  {
    auto get() {
        return static_cast<D*>(this)->get();
    }
};


struct D : public B<D> {
    using value_type = int;
    value_type get() {
        return 6;
    }
};
```

