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
