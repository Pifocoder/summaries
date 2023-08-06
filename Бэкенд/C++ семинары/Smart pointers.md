Unique - лекция
Shared
```C++
struct BaseControlBlock {
	size_t count;
};
template<typename T>
struct MakeSharedControlBlock: BaseControlBlock {
	T object;
};
template<typename T>
struct PinterControlBloc:BaseControlBlock {
	T* object;
};

template<typename T>
struct SharedPtr {
	ControlBlock<T>* control_block_;
}
```

# std::any
может храниться все что угодно
```C++
std::any tmp = std::vector<int>({1, 2, 3, 4});
tmp = "text";
tmp = 10;
auto x = std::any_cast<int&> (tmp);
```

Реализация
```C++
struct Any {
	Base* ptr;
	
	template<typename T>
	Any (const U& object): ptr(new De) {
		
	}
	struct Base {
		virtual ~Base() = default;
	}
	template<typename T>
	struct Derived: Base {
		T object;
		Derived(const T& value): object(value) {}
	}
}
```