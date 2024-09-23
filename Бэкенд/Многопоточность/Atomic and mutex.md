Примитивная схема - даем поисполняться потокам некоторый квант времени, останавливаем, когда хотим
Кооперативная схема - останавливаем в определенное время, например, когда ждем ответа от сервера, то есть в не полезное время (в случае fiber)

В Go примитивно-кооперативный режим, потому что это просто язык программирования.

## std::atomic
Пример:
Хотим 100 раз сделать a += 1. Разделим выполнение 2 потока:
 a += 1 эквивалентно t = a; a = t + 1
 пусть изначально a = 0:
 1) в первом потоке выполним t = a
 2) переключились на 2й поток
 3) выполним t = a
 4) выполним a = t + 1
 5) переключились на 1й поток
 6) выполним a = t + 1
По итогу a = 1, а не 2, для этого нужны atomic.

Мы передаем в atomic тип, а так же там есть операции, которые мы можем исполнять, например: fetch_add
```cpp
struct Counters { int a; int b; }; // user-defined trivially-copyable type
std::atomic<Counскопипастишьters> cnt;         // specialization for the user-defined type
```
[cppref](https://en.cppreference.com/w/cpp/atomic/atomic)

## Виды ошибок
### Data race
Более одного потока обращаются к ячейке, хотя бы одно из обращений - запись.
В C++ это сразу UB.

### Race condition
Состояние - гонка
То есть от порядка планировки может зависеть результат

## Односвязанный список
Простейшая реализация
```cpp
Node {
	int v;
	Node* next = nullptr;
}

head_ = nullptr;
Push(int v) {
	node = nde Node(v);
	node->next = head_;
	head_ = node;
}
```
Проблема: создаем по node в 2х потоках, но и простейшая бага.

Решение: хотим в 2 последние строки пускать только один поток. С помощью mutex

## Mutex
```cpp
Mutex m_;
Push(int v) {
	node = nde Node(v);
	m_.lock()
	node->next = head_;
	head_ = node;
	m_.unlock()
}
```
Свойства, которые  на  mutex:
1) Взаимноисключение (Muture exclusion)
2) Mutex должен прогрессировать, когда нибудь lock захватится(Limness)

в std::mutex есть ещё try_lock он помогает не уснуть
```cpp
#include <chrono>
#include <iostream>
#include <mutex>
#include <thread>
 
std::chrono::milliseconds interval(100);
 
std::mutex mutex;
int job_shared = 0; // both threads can modify 'job_shared',
                    // mutex will protect this variable
 
int job_exclusive = 0; // only one thread can modify 'job_exclusive'
                       // no protection needed
 
// this thread can modify both 'job_shared' and 'job_exclusive'
void job_1() 
{
    std::this_thread::sleep_for(interval); // let 'job_2' take a lock
 
    while (true)
    {
        // try to lock mutex to modify 'job_shared'
        if (mutex.try_lock())
        {
            std::cout << "job shared (" << job_shared << ")\n";
            mutex.unlock();
            return;
        }
        else
        {
            // can't get lock to modify 'job_shared'
            // but there is some other work to do
            ++job_exclusive;
            std::cout << "job exclusive (" << job_exclusive << ")\n";
            std::this_thread::sleep_for(interval);
        }
    }
}
 
// this thread can modify only 'job_shared'
void job_2() 
{
    mutex.lock();
    std::this_thread::sleep_for(5 * interval);
    ++job_shared;
    mutex.unlock();
}
 
int main() 
{
    std::thread thread_1(job_1);
    std::thread thread_2(job_2);
    
    thread_1.join();
    thread_2.join();
}
```
[std::lock_guard](https://en.cppreference.com/w/cpp/thread/lock_guard) - помогает при ошибках и исключениях
В деструкторе unlock и в конструкторе lock.
[std::unique_lock](https://en.cppreference.com/w/cpp/thread/unique_lock) - умный lock_guard, для более сложных вещей, чтобы лочиться и анлочиться(на сложные операции)

Есть также блокировка на чтение, может быть несколько блокировок на чтения. Идея в том, чтобы никто не писал, когда кто-то читает. [std::shared_mutex](https://en.cppreference.com/w/cpp/thread/shared_mutex)

# Реализуем Mutex
1 версия
```cpp
#pragma once

#include<atomic>

class SpinLock {
public:
    void Lock() {
        while (state_.load() == true) {}
        state_.store(true);
    }
    void Unlock() {
        state_.store(false);
    }

private:
    std::atomic<bool> state_{false};
};
```
> [!bug]
одновременный выход из while

Решение:
```cpp
#pragma once
#include<atomic>
class SpinLock {
public:
    void Lock() {
        while (state_.exchange(true) == true) {}
    }
    void Unlock() {
        state_.store(false);
    }

private:
    std::atomic<bool> state_{false};
};
```

### Volatile
Говорить компилятору не оптимизировать код

### Store buffering
Когда переменная попадает в кэш, а мы её берем из RAM.


