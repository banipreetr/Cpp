Consider the example of implementing a smart pointer where you want to type-erase the Deleter type. You can do that with polymorphism as follows:

```cpp
#include <bits/stdc++.h>

template<typename T>
class smartpointer
{
    public:
        template <typename Deleter>
        smartpointer(T* ptr, Deleter del_)
        : ptr_{ptr}
        , d_(new deleter(del_))
        {}
        ~smartpointer()
        {
            (*(d_))(ptr_);
            delete d_;
        }
    private:
        struct deleter_base {
            virtual void operator()(void* ptr) = 0;
            virtual ~deleter_base() = default;
        };
        template<typename Deleter>
        struct deleter : public deleter_base {
            deleter(const Deleter& d) : d_{d} {}
            void operator() (void* ptr) override {
                d_(ptr);
            }
            ~deleter() = default;
            Deleter d_;
        };
        deleter_base* d_;
        T* ptr_;
};

int main()
{
    auto deleter = [](void* ptr) -> void {
        std::cout << "Custom deleter called\n";
        delete (int*)ptr;
    };
    smartpointer<int> sp(new int{5}, deleter);

}
```

We can also modify this program to use local buffer optimisation to avoid any memory allocation and store the deleter as part of the class:


```cpp
#include <bits/stdc++.h>

template<typename T>
class smartpointer
{
    public:
        template <typename Deleter>
        smartpointer(T* ptr, Deleter del_)
        : ptr_{ptr}
        {
            static_assert(sizeof(deleter<Deleter>) <= sizeof(buf_));
            ::new (static_cast<void*>(buf_)) deleter<Deleter>(del_);
        }
        ~smartpointer()
        {
            deleter_base* d = (deleter_base*)buf_;
            (*(d))(ptr_);
            d->~deleter_base();
        }
    private:
        struct deleter_base {
            virtual void operator()(void* ptr) = 0;
            virtual ~deleter_base() = default;
        };
        template<typename Deleter>
        struct deleter : public deleter_base {
            deleter(const Deleter& d) : d_{d} {}
            void operator() (void* ptr) override {
                d_(ptr);
            }
            ~deleter() = default;
            Deleter d_;
        };
        alignas(16) char buf_[16];
        T* ptr_;
};

int main()
{
    auto deleter = [](void* ptr) -> void {
        std::cout << "Custom deleter called\n";
        delete (int*)ptr;
    };
    smartpointer<int> sp(new int{5}, deleter);

}
```

We can also do type erasure without using inheritance. How we can do that is:


```cpp
template<typename TE> void erased_func(void* p) {
    TE* q = static_cast<TE*>(p);
    // .. do some work
}
```

That way, what we are doing is, we are creating a function pointer of type `void(*)(void*)` which does the type erasure stuff and creates a function pointer like:
```cpp
void(*)(void*) fp = erased_func<int>;
```

So for our smart pointer example

```cpp
#include <bits/stdc++.h>

template<typename T>
class smartpointer
{
    public:
        template <typename Deleter>
        smartpointer(T* ptr, Deleter del_)
        : ptr_{ptr}
        , destroy_(invoke_destroy<Deleter>)
        {
            static_assert(sizeof(Deleter) <= sizeof(buf_));
            ::new (static_cast<void*>(buf_)) Deleter(del_);
        }
        ~smartpointer()
        {
            this->destroy_(ptr_, buf_);
        }
    private:
        using destroy_t = void(*)(T*, void*);
        destroy_t destroy_;
        template<typename Deleter>
        static void invoke_destroy(T* p, void* d) {
            (*static_cast<Deleter*>(d))(p);
        }
        alignas(16) char buf_[16];
        T* ptr_;
};

int main()
{
    auto deleter = [](void* ptr) -> void {
        std::cout << "Custom deleter called\n";
        delete (int*)ptr;
    };
    smartpointer<int> sp(new int{5}, deleter);
}
```

What we are doing is - 
* We get `Deleter` in constructor and then we use that to create a function pointer (type erasure) to a static function of type `void(*)(T*, void*)`. T is fixed, `void*` is the type of deleter.
* invoke_destroy is templated on the type of `Deleter`, so `destroy_` function pointer already has the information of `Deleter`, so we do not care about it anymore.
* Now we store the `Deleter` object in `buf_` through placement new operator and then we just pass `buf_` at the destructor level when needed.
  
