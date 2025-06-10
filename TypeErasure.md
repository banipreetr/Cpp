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
