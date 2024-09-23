## futex
futex - fast user-space locking
Реализуем первый Mutex с futex:
```cpp
#pragma once
#include<atomic>
class SpinLock {
public:
    void lock() {
        while (state_.exchange(1)) {
	        FutexWait(&state_, 1);
        }
    }
    void unlock() {
        state_.store(1);
        FutexWake(&state_, 1);
    }

private:
    std::atomic<int32_t> state_{0};
};
```

Condvar
```cpp
class Condvar {
public:
	template<class Lockable>
	void Wait(Lockable* lock) {
		auto old = counter_.fetch_add(1);
		waiters.fetch_add(1);
		lock.unlock();
		FutexWait(&counter_, old + 1);
		lock.lock();
		waiters.fetch_sub(1);
	}
	void NotifyOne() {
		auto num_waited = waiters.load();
		if (num_waited > 0) {
			counter_.fetch_sub(1);
			FutexWake(&counter_, 1);
		}
	}
	void NotifyAll() {
		auto num_waited = waiters.load();
		if (num_waited > 0) {
			counter_.store(0);
			FutexWake(&counter_, num_waited);
		}
	}
private:
	std::atomic<uint32_t>counter_{0};
	std::atomic<uint32_t>waiters{0};
}
```

