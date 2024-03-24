```C++
void f() {
	int* p = new int;
	g();//если тут вылетит исключение, то будет утечка
	delete p;
}
```

```C++
void f() {
	std::unique_ptr<int> p(new int);//delete как бы сам вызовется
	g();
}
```

# unique_ptr
Примитивная реализация
```C++
template<typename T>
class UniquePtr {
	T* ptr;
	UniquePtr(T* ptr): ptr(ptr) {}
	
	UniquePtr(const UniquePtr&) = delete;
	UniquePtr& operator(const UniquePtr&) = delete;
	
	UniquePtr(UniquePtr&& u):ptr(u.ptr) {
		u.ptr = nullptr;
	}
	UniquePtr& operator=(UniquePtr&& u) {
		UniquePtr copy = std::move(u);
		swap(ptr, copy.ptr)
		return (*this);
	}
	~UniquePtr() {
		delete ptr;
	}
}

template<typename T>
class UniquePtr<T[]> {
	T* ptr;
	UniquePtr(T* ptr): ptr(ptr) {}
	
	UniquePtr(const UniquePtr&) = delete;
	UniquePtr& operator(const UniquePtr&) = delete;
	
	UniquePtr(UniquePtr&& u):ptr(u.ptr) {
		u.ptr = nullptr;
	}
	UniquePtr& operator=(UniquePtr&& u) {
		UniquePtr copy = std::move(u);
		swap(ptr, copy.ptr)
		return (*this);
	}
	~UniquePtr() {
		delete[] ptr;
	}
}
```
Мы разрешили только Move штуки
Копирования нет, но есть перемещение

Каждый unique ptr отвечает за свой указатель, но можно сделать так и будет UB
```C++
int* t = new int;
std::unique_ptr<int> p1(t);
std::unique_ptr<int> p2(t);//UB
```

Также еще должен быть класс deleter, чтобы в деструкторе можно было делать еще что-нибудь, а не только delete.
```C++
template<typename T>
class my_delete {
	void operator()(T* ptr) {
		std::cout << "del";
		delete ptr;
	}
}
template<typename T, typename Deleter = my_delete>
class UniquePtr: private Deleter {
}

int main() {
	std::unique_ptr<int, my_delete<int>> p(new int)
}
```

# shared_ptr
delete, когда удалим все копии, которые указывают на одно место
набросок
```C++
template<typename T>
class SharedPtr {
	T* ptr;
	SharedPtr(T* ptr): ptr(ptr) {}
}
```

Как же найти последний, счетчик
Идеи:
* static size_t count; - для одинаковых типов будет один и тот же счетчик
Надо хранить указатель на счетчик
```C++
template<typename T>
class SharedPtr {
	T* ptr;
	int* count;
	
	SharedPtr(T* ptr): ptr(ptr), count(new int(1)) {}
	SharedPtr(const SharedPtr& other): ptr(other.ptr), count(other.count) {
		++*count;//TODO if nullptr
	}
	~SharedPtr() {
		--*count;//TODO if nullptr (or if controlblock)
		if (*count == 0) {
			delete ptr;
			delete count;
		}
	}
}
```

## make_unique, make_shared
make_shared - избегаем двойного new
```C++
template<typename T>
class SharedPtr {
	T* ptr;
	int* count;
	ControlBlock* cb;
	
	struct ControlBlock {
		T object;
		int count;
	}
template<typename T, typename... Args>
SharedPtr<T> make_shared(Args&&... args) {
	auto cb = new typename SharedPtr<T>::ControlBlock(T(std::forward<Args>(args...)), 1);
	return SharedPtr<T>(cb);
}
```
Теперь у нас есть два сценария создания через make_shared и как до make_shared, поэтому количество кода в других ф-ях увеличится.

make_unique 
```C++
template<typename T, typename... Args>
SharedPtr<T> make_unique(Args&&... args) {
	return UniquePtr<T>(new T(std::forward<Args>(args)...))
}

auto p = std::make_shared<int>(50);//не написали new и это хорошо
```
## week_ptr
Проблема циклических ссылок.
week_ptr - умеет смотреть на обьект, проверять не уничтожился ли он
week_ptr работает только в связке с shared_ptr

```C++
template<typename T>
class WeekPtr {
	T* object:
	int* count;
	ControlBlock* cb;//friend SharedPtr
	
public:
	WeekPtr(const SharedPtr<T>& other);
	bool expired() const noexcept{//проверяет удален ли обьект
		//проблема sharedptr - все поудалял за собой когда последний удалился
		//решение - добавляем еще один счетчик
	}
	SharedPtr<T> lock() const noexcept;//возвращает  указатель на свой SharedPtr
	~WeekPtr();
};
```

```C++
class SharedPtr {
	T* ptr;
	ControlBlock* cb;
	struct BaseControlBlock {
		int shared_ptr_count;
		int week_ptr_count;
	}
	struct ControlBlock: BaseControlBlock {
		T object;
		int count;
	}
	~SharedPtr() {
		--cb->shared_ptr_count;
		if (cb->shared_ptr_count == 0) && (cb->week_ptr_count == 0) {
			delete cb;
		} else {
			if (cb->shared_ptr_count == 0) {
			 if (ptr) delete ptr;
			 else static_cast<ControlBlock*>(cb)->object.~T();
			}
		}
	}
}
```
Теперь понятно как дописать WeekPtr.

## enable_shared_from_this and CRTP idiom
хотим сделать это:
```C++
struct S {
	SharedPtr<S> getPointerToObject() {
		if (...) {
			return this;
		}
	}
}
```
Но мы не можем создавать sharedPtr на себя
решение - enable_shared_from_this
```C++
struct S: public  std::enable_shared_from_this<S>{//CRTP idiom
	SharedPtr<S> getPointerToObject() {
		if (...) {
			return shared_from_this;
		}
	}
}

template<typaname S>
struct EnableSharedFromThis {
	template<typename U>
	friend class SharedPtr<U>;
	WeekPtr<T> wptr;

	Shared<T> shared_from_this() const noexcept{
		return wptr.lock();
	}
}

template<typename T>
class SharedPtr {
	///...
	SharedPtr(T* ptr): ptr(ptr), count(new int(1)) {///не дописано 50-я мин
		if constexpr (std::is_base_of_v<EnebleSharedFromThis<T>,T>) {
			ptr->wptr = *this;
		}
	}
}
```
CRTP - curiosly recurving template patern

Нужно бахнуть аллокаторы)
И добавить нестандартный delet-er

Но ни deleter ни аллокатор не являюься шаблонными параметрами
АХУЕТЬ
То есть они могут исзменяться линамически(типы)

Polymorphic allocator

Нам нужно хранить указатель на вазов делитера и на его уничтожение
Type Erasure idiom
```C++
class SharedPtr {
	//...
	struct BaseControlBlock {
		int shared_count;
		int weak_count;
		virtual void UseDeleter(T* ptr) = 0;
		virtual ~BaseControlBlock() = default;
	}
	template<typename Deleter>//template<typename Deleter, typename Alloc>
	struct ControlBLokRegular: BaseControlBlock {
		//Alloc alloc;
		Deleter deleter; 
		T* object;
		virtual void useDeleter(T* ptr) {
			deleter(ptr);
		}
	}
	//template<typename Alloc>
	struct ControlBLokMakeShared: BaseControlBlock {
		//Alloc alloc;
		T object;
	}
	SharedPtr(T* ptr, Deleter deleter): cb(new ControlBLokRegular{1, 0, ptr, deleter}){
		//...
	}
	~SharedPtr() {
		--cb->shared_ptr_count;
		if (cb->shared_ptr_count == 0) && (cb->week_ptr_count == 0) {
			cb->UseDeleter()
		} else {
			if (cb->shared_ptr_count == 0) {
			 if (ptr) delete ptr;
			 else static_cast<ControlBlock*>(cb)->object.~T();
			}
		}
	}
}
```
Теперь аллокатор
![](https://i.imgur.com/7ZPFDq3.png)

```C++
class SharedPtr {
	//...

	struct BaseControlBlock {
		int shared_count;
		int weak_count;
		virtual void UseDeleter(T* ptr) = 0;
		virtual ~BaseControlBlock() = default;
	}
	template<typename Deleter, typename Alloc>
	struct ControlBLokRegular: BaseControlBlock {
		Alloc alloc;
		Deleter deleter; 
		T* object;
		virtual void useDeleter(T* ptr) {
			deleter(ptr);
		}
	}
	template<typename Alloc>
	struct ControlBLokMakeShared: BaseControlBlock {
		Alloc alloc;
		T object;
	}
	SharedPtr(T* ptr, Deleter deleter): cb(new ControlBLokRegular{1, 0, ptr, deleter}){
		//...
	}
	~SharedPtr() {
		--cb->shared_ptr_count;
		if (cb->shared_ptr_count == 0) && (cb->week_ptr_count == 0) {
			cb->UseDeleter(ptr)
		} else {
			if (cb->shared_ptr_count == 0) {
			 if (ptr) delete ptr;
			 else static_cast<ControlBlock*>(cb)->object.~T();
			}
		}
	}

	T* ptr;
	BaseControlBlock* cb;
}
template<typename T, typename Alloc, typename... Args>
SharedPtr<T> allocate_shared(const Alloc& alloc, Args&&... args) {
	using AllocControlBlock = std::allocator_traits<Alloc>::<ControlBlockMakeShared<Alloc>>;
	AllocControlBlock newAlloc= alloc;
}
```