```cpp
#include<MPMCQueue>
using Task = std::function<void()>;

static thread_local ThreadPool current = nullptr;

class ThreadPool {
public:
	explicit ThreadPool(size_t num_threads) {
		for (size_t i = 0; i < num_threads; ++i) {
			workers_.emplace_back([this]() {
				Work()
			})
		}
	}
	~ThreadPool() {
		for (auto& worker : workers_) {
			worker.join()
		}
	}

	void Submit(Task task) {
		tasks_.Push(std::move(task))
	}
	void Wait() {
		while (tasks_.Size() > 0) {}
		is_closed_.store(true);
	}
	static ThreadPool* This() {
		return current;
	}
	
private:
	void Work() {
		while (!is_closed_) {
			auto task = tasks_.Pop();
			old = current;
			current = this;
			task();
			current = old;
		}
	}
	std::vector<std::thread> workers_;
	MPMCQueue<Task> tasks_;
	std::atomic<bool> is_closed_ = false;
}
```
> [!bug]
Не работает, когда есть потоки, которые никогда не заработают ни разу.

> [!quote]
> Реализация очереди  (из прошлого)
```cpp
template<class T>
class Queue {
public:
	void Push(T elem) {
		std::lock_guard g(lock_);
		buffer_.push(std::move(elem));
		if (num_waiting > 0) {
			non_empty.notify_one();
		} 
	}
	T Pop() {
		std::unique_lock g(lock_);
		while (buffer.empty()) {
			++num_waiting;
			non_empty.wait(g);
			--num_waiting;
		}
		auto elem std::move(buffer_.front());
		buffer_.pop();
		return elem;
	}
	size_t Size() {
		std::lock_guard g();
		return buffer_.size();
	}
private:
	std::queue<T>buffer_;
	std::mutex lock_;
	std::conditional_varible non_empty

	size_t num_waiting;	
}
```