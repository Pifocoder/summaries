## stackfull coroutine
```cpp
static Coroutine* current = nullptr;//для вложенных coroutine
using Routine = std::function<void()>
class Coroutine {
public:
	Coroutine(Routine routine): routine_(std::move(routine)) {
		callee_ctx_.Setup(AllocateStack, this);
	}
	void Resume() {
		auto old = current;
	    current = this;
	    
		caller_ctx_.Switch(callee_ctx_);
		
		current = old;
	}
	static void Suspend() {
		assert(current);
		current->callee_ctx_.Switch(current->caller_ctx_);
	}

	bool IsComplited() {
		return is_complited;
	}
private:
	void Run() final {
		routine_();
		is_complited = true;
		
		callee_ctx_.Switch(caller_ctx_);
		
		std::abort();
	}
private:
	bool is_complited {false};
	
	Routine routine_;
	Context callee_ctx_;
	Context caller_ctx_;
}
```

# EPOLL
Api операционной системы.

Epoll
https://man7.org/linux/man-pages/man7/epoll.7.html

Asio echo server
https://github.com/chriskohlhoff/asio/blob/master/asio/src/examples/cpp14/echo/blocking_tcp_echo_server.cpp
https://github.com/chriskohlhoff/asio/blob/master/asio/src/examples/cpp14/echo/async_tcp_echo_server.cpp
https://github.com/chriskohlhoff/asio/blob/master/asio/src/examples/cpp20/coroutines/echo_server.cpp


# Boost
https://www.boost.org/
Симметричные coroutine.

```cpp
namespace ctx = boost::context;

TEXT_CASE("Async coroutines", "Simple") {
	int value = 1;
	
	ctx::fiber fiber([](ctx::fiber&& fiber)) {
		value = 2;
		fiber = std::move(fiber).resume();
		value = 3;
		return fiber;
	});
	REQUIRED(value == 1);
	fiber = std::move(fiber).resume();
	REQUIRED(value == 2);
	fiber = std::move(fiber).resume();
	REQUIRED(value == 3);
}
```

Есть разные способы увеличения стэка, это нужно, потому что по умолчанию размер - 128кб. 
1й подход - сегментация стэка, но это очень долго
2й подход - сборщик мусора + копирование стэка и увеличение в 2 раза.
А как сделать в плюсах :

## stackless
[cppref](https://en.cppreference.com/w/cpp/language/coroutines)
[статьи](https://lewissbaker.github.io/)
Объекты:
1) promise - задается пользователем
2) coroutine_handle - указатель, ктороый указывает на coroutine_state (методы: resume, destroy)
3) coroutine_state -  состояние в котором хранится индекс и локальные переменные

Напишем свою coroutine:
```cpp
#include <coroutine>

//простая stateless coroutine
struct Task {
	struct TaskPromise {
		Task get_return_object() {
			return Task{std::coroutine_handle<TaskPromise>::from_promise(*this)};
		}
		std::suspend_never initial_sispend() noexcept {
			return {};
		}
		std::suspend_never final_sispend() {
			return {};
		}
		void return_void() {
			
		}
		void unhandled_exception() {
		
		}
	};
	
	using promise_type = TaskPromise;
	
	std::coroutine_handle<> handle;
}

// пример использования
Task coro(int& value) {
	value = 2;
	co_wait std::suspend_always{};
	value = 3;
	co_wait std::suspend_always{};
	value = 4;
	
	co_return;
}
TEST_CASE("CoroTest", "Test") {
	int value = 1;
	Task task = coro(value);
	
	REQUIRED(task.handle);
	REQUIRED(value == 1);
	task.handle.resume();
	REQUIRED(value == 2);
	task.handle.resume();
	REQUIRED(value == 3);
	task.handle.resume();
	REQUIRED(value == 4);
	task.handle.destroy();
}
```
Теперь хотим вложенные:
```cpp
#include <coroutine>

//простая stateless coroutine
struct Task {
	struct TaskPromise {
		Task get_return_object() {
			return Task{std::coroutine_handle<TaskPromise>::from_promise(*this)};
		}
		std::suspend_never initial_sispend() noexcept {
			return {};
		}
		std::suspend_never final_sispend() {
			return {};
		}
		void return_void() {
			
		}
		void unhandled_exception() {
		
		}
	};
	
	using promise_type = TaskPromise;
	
	std::coroutine_handle<> handle;
}

// пример использования
void inner() {
	co_await std::suspend_always{};
}

Task coro(int& value) {
	value = 2;
	co_wait std::suspend_always{};
	value = 3;
	co_wait std::suspend_always{};
	value = 4;
	
	co_return;
}
TEST_CASE("CoroTest", "Test") {
	int value = 1;
	Task task = coro(value);
	
	REQUIRED(task.handle);
	REQUIRED(value == 1);
	task.handle.resume();
	REQUIRED(value == 2);
	task.handle.resume();
	REQUIRED(value == 3);
	task.handle.resume();
	REQUIRED(value == 4);
	task.handle.destroy();
}
```

Продолжаем писать нашу coroutine.
```cpp
#include <coroutine>

//простая stateless coroutine
class Task {
public:
	struct TaskPromise {
		Task get_return_object() {
			return Task{this};	
		}
		std::suspend_always initial_suspend() noexcept {
			return {};
		}
		std::suspend_always final_suspend() {
			if (Continuation) {
				Continuation.resume();
			}
			return {};
		}
		void return_void() {//будет вызван при завершении coroutine
			
		}
		void unhandled_exception() {// будет вызван, когда в теле coroutine бросят исключени и не поймают
			 abort();
		}
		std::coroutine_handle<> Continuation;
	};

	bool await_ready() {
		return false;
	}
	void await_suspend(std::coroutine_handle<> handle) {
		handle.promise().Continuation = handle;
		std::coroutine_handle<TaskPromise>::from_promise(*Promise_).resume();
	}
	void await_resume() {
		
	}
	using promise_type = TaskPromise;
	void Run() {
		std::coroutine_handle<TaskPromise>::from_promise(*Promise_).resume();
	}
	~Task() {
		std::coroutine_handle<TaskPromise>::from_promise(*Promise_).destroy();
	}
private:
	Task(TaskPromise* promise) : Promise_(promise) {}
	std::coroutine_handle<> handle;
	TaskPromise* Promise_;
}

class ManualSheduler {
public:
	void Resume() {
		Handle_.resume();
		Handle_ = std::coroutine_handle<>{};
	}
	void Shedule(std::coroutine_handle<> handle) {
		Handle_ = handle;
	}
private:
	std::coroutine_handle<> Handle_;
}

struct ManualSheduleAwaitable {
	ManualSheduleAwaitable(ManualSheduler& sheduler) : Sheduler_(sheduler) {}
	
	bool await_ready() {
		return false;
	}
	void await_suspend(std::coroutine_handle<> handle) {
		Sheduler_.Schedule(handle);
	}
    void await_resume() {
	}
	private:
	ManualSheduler Sheduler_;
}

Task coro_func(int& value, ManualSheduler& sheduler) {
	auto value  = 1;
	auto awaitable = ManualSheduleAwaitable{sheduler};
	co_await awaitable;
	// преобразование компилятора строчки co_await awaitable; в следущее:
	//if (awaitable.await_ready()) {
	//    return awaitable.await_resume();
	//} else {
	//   Save local variables in state, save suspend index
	//   awaitable.await_suspend(coro_handle);
	//   return to caller
	//    
	//   someone calls coro_handle.resume()
	//   resume
	//}
	//
	value  = 2;
	co_return;
}

TEST_CASE("CoroTest", "Test") {
	ManualSheduler sheduler;
	int value = 0;
	auto coro = coro_func(value, sheduler);
	REQUIRE(value == 1);
	sheduler.Resume();
	REQUIRE(value == 2
	);
}

```
>[!bug]
>Проблема c деструтктором 

>[!success]
> add FinalSuspendAwaitable
```cpp
#include <coroutine>

//простая stateless coroutine
class Task {
public:
	struct TaskPromise {
		Task get_return_object() {
			return Task{this};	
		}
		std::suspend_always initial_suspend() noexcept {
			return {};
		}
		auto final_suspend() noexcept {
			struct FinalSuspendAwaitable {
				bool await_ready() {
					return false;
				}
				void await_suspend(std::coroutine_handle<TaskPromise> handle) {
					if (handle.promise().Continuation) {
						handle.promise().Continuation.resume();
					}
				}
				void await_resume noexcept {}
			}
			
			return FinalSuspendAwaitable{};
		}
		void return_void() {//будет вызван при завершении coroutine
			
		}
		void unhandled_exception() {// будет вызван, когда в теле coroutine бросят исключени и не поймают
			 abort();
		}
		std::coroutine_handle<> Continuation;
	};

	bool await_ready() {
		return false;
	}
	void await_suspend(std::coroutine_handle<> handle) {
		handle.promise().Continuation = handle;
		std::coroutine_handle<TaskPromise>::from_promise(*Promise_).resume();
	}
	void await_resume() {
		
	}
	using promise_type = TaskPromise;
	void Run() {
		std::coroutine_handle<TaskPromise>::from_promise(*Promise_).resume();
	}
	~Task() {
		std::coroutine_handle<TaskPromise>::from_promise(*Promise_).destroy();
	}
private:
	Task(TaskPromise* promise) : Promise_(promise) {}
	std::coroutine_handle<> handle;
	TaskPromise* Promise_;
}

class ManualSheduler {
public:
	void Resume() {
		Handle_.resume();
		Handle_ = std::coroutine_handle<>{};
	}
	void Shedule(std::coroutine_handle<> handle) {
		Handle_ = handle;
	}
private:
	std::coroutine_handle<> Handle_;
}

struct ManualSheduleAwaitable {
	ManualSheduleAwaitable(ManualSheduler& sheduler) : Sheduler_(sheduler) {}
	
	bool await_ready() {
		return false;
	}
	void await_suspend(std::coroutine_handle<> handle) {
		Sheduler_.Schedule(handle);
	}
    void await_resume() {
	}
	private:
	ManualSheduler Sheduler_;
}

Task coro_func(int& value, ManualSheduler& sheduler) {
	auto value  = 1;
	auto awaitable = ManualSheduleAwaitable{sheduler};
	co_await awaitable;
	// преобразование компилятора строчки co_await awaitable; в следущее:
	//if (awaitable.await_ready()) {
	//    return awaitable.await_resume();
	//} else {
	//   Save local variables in state, save suspend index
	//   awaitable.await_suspend(coro_handle);
	//   return to caller
	//    
	//   someone calls coro_handle.resume()
	//   resume
	//}
	//
	value  = 2;
	co_return;
}

TEST_CASE("CoroTest", "Test") {
	ManualSheduler sheduler;
	int value = 0;
	auto coro = coro_func(value, sheduler);
	REQUIRE(value == 1);
	sheduler.Resume();
	REQUIRE(value == 2
	);
}

```
>[!bug]
>возобновление корутины через await_suspend приводит к переполнению стэки при большом количестве итераций

>[!success]
>из await_suspend будем возвращать корутину, которую надо будет возобновить (как в доках)
>код надо изменить как await_suspend, так и в final_suspend



