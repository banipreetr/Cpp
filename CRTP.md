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

If you still need the nested type for some reason to declare the Base class, put it as a template parameter.
Example:

```cpp
template<typename D, typename value_type>
struct B  {
    value_type val;
    auto get() {
        return static_cast<D*>(this)->get();
    }
};


struct D : public B<D, int> {
    using value_type = int;
    value_type get() {
        return val;
    }
};
```

And the above will compile...

So, in a nutshell, we are achieving a polymorphic behaviour through compile time and not at runtime.

## Practical Uses

With all in mind about CRTP, how would you use this with functions?
Example:

```cpp
template<typename D>
struct B  {
};


struct D : public B<D> {
};

template<typename D>
void func(B<D>* ptr) {
}


int main()
{
    B<D>* ptr = new D();
    func(ptr);
}
```

### Virtual like Functions

Now, talking about compile time polymorphism, how would we achieve virtual like behaviour?

We can do this:

```cpp
template<typename D>
struct B  {
    void f() {
        static_cast<D*>(this)->f();
    }
};


struct D : public B<D> {
    void f()
    {
    }
};
```

But what if `D` didn't define `f()` ? In that case there is an infinite recursion. Example:

```cpp
template<typename D>
struct B  {
    void f() {
        cout << "Inside base class\n";
        static_cast<D*>(this)->f();
    }
};


struct D : public B<D> {
};
```

In the above case, D::f() is B::f() which calls D::f(), which in turn calls B::f() because there is no implementation of D::f().

To avoid a pure virtual function type behaviour, wthout having this infinite recursion problem, what you can do is, rename the implementation:

And if the implementation function is not defined, then the compiler will throw error:

Example:

```cpp
template<typename D>
struct B  {
    void f() {
        static_cast<D*>(this)->f_imp();
    }
};


struct D : public B<D> {
    void f_imp() {
    }
};
```

And with this, you can also define a default implementation in the base class, so that it defaults to that implementation given that the derived class decides to not override it, example:


```cpp
template<typename D>
struct B  {
    void f() {
        static_cast<D*>(this)->f_imp();
    }
    void f_imp() {
        cout << "B::f_imp()\n";
    }
};

struct D : public B<D> {
    // void f_imp() {
    //     cout << "D::f_imp()\n";
    // }
};
```



### Deletion of Object


The main problem with CRTP kind of polymorphism is that the correct destructor won't be called. Consider an example:

```cpp
B<D>* ptr = new D();
...
...
delete ptr;
```

This will call destructor of B and since there is no virtual'ity in the destructor, it wouldn't know that D's destructor needs to be called.
But someone might think it could be straight forward as:

```cpp
~B() { static_cast<D*>(this)->~D(); }
```
But it is not that simple. The problem with this is:
* When destructor of the base class is reached, the actual object is not of the derived type anymore and calling any member function would result in an undefined behaviour and therefore doesn't work.
* Even if it worked somehow, as soon as ~D() finishes, ~B() will be called, and that would result in an infinite recursion


One could also do this:

```cpp
template<typename D>
void destroy(B<D>* obj) {
    delete static_cast<D*>(obj);
}
```

This can work, but what if someone forgets to call `destroy` ?

Something that you can actually do is, maybe make the destructor as virtual, which would again bring back the overhead of it being virtual and moreover it would also result in increasing the size of the class because of it being virtual....


### CRTP access control


With virtual functions, you can do this:

```cpp
struct B {
    virtual void f() = 0;
};

struct D : public B {
    private:
    void f() override {
        cout << "In private mode\n";
    }
};

int main()
{
    B* obj = new D();
    obj->f();
}
```

i.e, access a private function of derived class through runtime polymorphism. With CRTP, it can be a bit tricky...

Example, the following code wouldn't compile:

```cpp

template<typename D>
struct B  {
    void f() {
        static_cast<D*>(this)->f_imp();
    }
    private:
    void f_imp() {
        cout << "B::f_imp()\n";
    }
};

struct D : public B<D> {
    private:
    void f_imp() {
        cout << "D::f_imp()\n";
    }
};
```
Rather than that, what if we declare the class B<D> as friend:

```cpp
template<typename D>
struct B  {
    void f() {
        static_cast<D*>(this)->f_imp();
    }
    private:
    void f_imp() {
        cout << "B::f_imp()\n";
    }
};

struct D : public B<D> {
    private:
    void f_imp() {
        cout << "D::f_imp()\n";
    }
    friend class B<D>;
};
```

Also, you can make a mistake like this:

```cpp
struct D : public B<D> {
    private:
    void f_imp() {
        cout << "D::f_imp()\n";
    }
    friend class B<D>;
};

struct D1 : public B<D> {
    private:
    void f_imp() {
        cout << "D1::f_imp()\n";
    }
    friend class B<D1>;
};
```
Focus on `struct D1 : public B<D>`. This is okay and would compile but this is a subtle error

`B<D1>* obj = new D1()` won't compile although.

What we can do is make the Base class abstract by making the constructor private and making type `D` as friend (so that it can access the constructor, otherwise it would complain)


So
```cpp
template<typename D>
struct B  {

    friend D;
    void f() {
        static_cast<D*>(this)->f_imp();
    }
    private:
    B() = default;
    void f_imp() {
        cout << "B::f_imp()\n";
    }
};

struct D : public B<D> {
    private:
    void f_imp() {
        cout << "D::f_imp()\n";
    }
    friend class B<D>;
};

struct D1 : public B<D> {
    private:
    void f_imp() {
        cout << "D1::f_imp()\n";
    }
    friend class B<D1>;
};
```
wont compile as it would complain that the constructor of B<D>::B() is deleted for `D1`.


