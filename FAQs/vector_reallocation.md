## Why does the move constructor noexcept guarantee ensures that the reallocation of elements of the vector is faster?


This is because if the move constructor noexcept guarantee doesn't exists, it means that the move constructor of the contained type might throw, and standard implements the reallocation in a way that it would only use the move constructor if it satisfies the following conditions:
```cpp
if (std::is_nothrow_move_constructible<T>::value || !std::is_copy_constructible<T>::value) {
    // Use move
    new (dest) T(std::move(*src));
} else {
    // Use copy
    new (dest) T(*src);
}
```

To test this theory, see the following benchmarks:

```cpp

struct A {
    std::vector<int> arr;
    A(A&&) noexcept = default;
    A(const A&) = default;
    A(const std::vector<int>& other_arr) : arr{other_arr} {};
};

struct B {
    std::vector<int> arr;
    B(B&&) noexcept(false) = default;
    B(const B&) = default;
    B(const std::vector<int>& other_arr) : arr{other_arr} {};
};

static void BM_vector_noexcept_true(benchmark::State& state) {
    std::vector<A> vec;
    std::vector<int> arr;
    for(int i=0; i<1000; ++i) {
        arr.push_back(1000);
    }
    for(auto _ : state) {
        REPEAT(
            for(size_t i=0; i < 1e3; ++i) {
                vec.push_back(A{arr});
            }
        );
    }
    state.SetItemsProcessed(32*state.iterations());
}


static void BM_vector_noexcept_false(benchmark::State& state) {
    std::vector<B> vec;
    std::vector<int> arr;
    for(int i=0; i<1000; ++i) {
        arr.push_back(1000);
    }
    for(auto _ : state) {
        REPEAT(
            for(size_t i=0; i < 1e3; ++i) {
                vec.push_back(B{arr});
            }
        );
    }
    state.SetItemsProcessed(32*state.iterations());
}

BENCHMARK(BM_vector_noexcept_true);
BENCHMARK(BM_vector_noexcept_false);
```

which gives:

```bash
-----------------------------------------------------------------------------------
Benchmark                         Time             CPU   Iterations UserCounters...
-----------------------------------------------------------------------------------
BM_vector_noexcept_true    57198233 ns     57167970 ns           10 items_per_second=559.754/s
BM_vector_noexcept_false   68789011 ns     68788656 ns           12 items_per_second=465.193/s
```
