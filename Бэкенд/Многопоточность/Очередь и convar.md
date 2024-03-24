```cpp
template<class T>
class Queue {
public:
	void Push(T elem) {
		std::lock_guard g(lock_);
		buffer_.push(std::move(elem));
	}
	T Pop() {
		while (true) {
			std::unique_lock g(lock_);
			if (!buffer_.empty()) {
				auto elem std::move(buffer_.front());
				buffer_.pop();
				return elem;
			} else {
				g.unlock();
				std::this_thread::sleap_for(1s);
			}
		}
	}
private:
	std::queue<T>buffer_;
	std::mutex lock_;
}
```
Но конечно так себе
### std::conditional_varible
The `condition_variable` class is a synchronization primitive used with a [std::mutex](https://en.cppreference.com/w/cpp/thread/mutex "cpp/thread/mutex") to block one or more threads until another thread both modifies a shared variable (the _condition_) and notifies the `condition_variable`.
[cppref](https://en.cppreference.com/w/cpp/thread/condition_variable)
```cpp
template<class T>
class Queue {
public:
	void Push(T elem) {
		std::lock_guard g(lock_);
		buffer_.push(std::move(elem));
		
		non_empty.notify_one();
	}
	T Pop() {
		std::unique_lock g(lock_);
		if (buffer.empty()) {
			non_empty.wait(g);
		}
		auto elem std::move(buffer_.front());
		buffer_.pop();
		return elem;
	}
private:
	std::queue<T>buffer_;
	std::mutex lock_;
	std::conditional_varible non_empty;
}
```
> [!danger]
Теперь немного оптимизируем, потому что notify_one достаточно тяжелая функция.
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
		if (buffer.empty()) {
			++num_waiting;
			non_empty.wait(g);
			--num_waiting;
		}
		auto elem std::move(buffer_.front());
		buffer_.pop();
		return elem;
	}
private:
	std::queue<T>buffer_;
	std::mutex lock_;
	std::conditional_varible non_empty

	size_t num_waiting;	
}
```
> [!bug]
> Когда поток, который спит в wait, просыпается, а также приходит новый поток в pop. то есть оба хотят залочится. Тогда мы получается сделали notify, а другой поток отобрал элемент после пуша и тогда сработат pop при отсутствии элементов в очереди. 

> [!success]
> if -> while
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
private:
	std::queue<T>buffer_;
	std::mutex lock_;
	std::conditional_varible non_empty

	size_t num_waiting;	
}
```