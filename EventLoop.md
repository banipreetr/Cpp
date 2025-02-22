## My implementation of EventLoop


```cpp
#include <bits/stdc++.h>

using namespace std;

struct EventLoop {
    using Event = std::function<void()>;
    std::queue<Event> eventQueue_;
    bool running_{true};
    std::mutex mx_;
    std::condition_variable cv_;
    std::vector<std::thread> workers_;

    EventLoop(size_t numWorkers = 1)
    {
        for(int i=0; i<numWorkers; ++i) {
            workers_.emplace_back([&]() { this->run(); });
        }
    } 

    void postEvent(const Event& event) {
        std::lock_guard<std::mutex> lk(mx_);
        eventQueue_.push(event);
        cv_.notify_one();
    }
    void run() {
        while (running_) {
            std::function<void()> event;
            {
                std::unique_lock<std::mutex> lk(mx_);
                cv_.wait(lk, [&]() -> bool {
                    return !running_ || !eventQueue_.empty();
                });
                if (!running_) {
                    break;
                }
                event = std::move(eventQueue_.front());
                eventQueue_.pop();
            }
            event();
        }
    }
    void stop() {
        {
            std::lock_guard<std::mutex> lk(mx_);
            running_ = false;
        }
        cv_.notify_all();
        for(auto&& worker : workers_) {
            if (worker.joinable()) {
                worker.join();
            }
        }
    }

    ~EventLoop() {
        stop();
    }
};

int main() {
    EventLoop eventLoop(4);
    eventLoop.postEvent([]() { std::cout << "Event 1\n"; });
    eventLoop.postEvent([]() { std::cout << "Event 2\n"; });
    eventLoop.postEvent([]() { std::cout << "Event 3\n"; });
    eventLoop.postEvent([]() { std::cout << "Event 4\n"; });
    std::this_thread::sleep_for(std::chrono::milliseconds(5*1000));

    eventLoop.stop();
}
```
