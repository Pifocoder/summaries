в std::function можно хранить любой callable обьект
```C++
std::function<bool, (int, int)> f = [](int x, int y) {
	return x < y;
};
//можно присваивать
std::function<bool, (int, int)> f;
f = [](int x, int y) {
	return x < y;
};
//вызываем
std::cout << f(1, 2) << '\n';

struct MyCompare {
	bool opaerator()(int x, int y) {
		return x > y;
	}
}
f = MyCompare();
std::cout << f(2, 3);

bool another_compare(int x, int y) {
	return x < 2 * y;
}

f = another_compare;
```

Указатель на метод класса - это штука, которая хранит адрес метода в классе.
```C++
struct S {
	int x;
	void print (int y) {
		std::cout << x + y;
	}
}
void (S::*pm)(int) = &S::print;
S s{5};
(s.*pm)(3);//выведется 8

//чтобы сделать srd::function, нам нужно передать еще и обьект, так как непонятно от чего вызывать
std::function<void(S&, int)> f2 = pm;
f2(s, 4);
```

## Внутрянка
```C++
template <typename... Args>
class function;

template<typename Ret, typename... Args>
class function<Ret(Args...)> {
	struct Base {
		virtual operator()(Args...) = 0;
		virtual ~Base() = default;
	}
	template<typename T>
	struct Derived: Base {
		T functor;
		virtual Ret operator()(Args... args) override {
			return functor(args);
		}
	}
	Base* fptr = nullptr;
public:
	template <typename F> 
	function (const F& f): fptr(new Derived<F>(f)) {}
	~function() {
		delete fptr
	}
	Ret operator() (Args... args) const {
		return fptr->operator()(args...);
	}
}
```

> [!INFO]
> function не поддерживает нестандартные аллокаторы
> Как сделать копирование это нетривиально, потому что в такой реализации мы не знае, какой тип подставить в fptr = new ???? (other->fptr)б поэтоу дулаем еще одну виртуальную функцию

```C++
template <typename... Args>
class function;

template<typename Ret, typename... Args>
class function<Ret(Args...)> {
	struct Base {
		virtual operator()(Args...) = 0;
		virtual ~Base() = default;
		virtual Base* getCopy() const = 0;
		
	}
	template<typename T>
	struct Derived: Base {
		T functor;
		virtual Ret operator()(Args... args) override {
			return functor(args);
		}
		virtual Base* getCopy() const override {
			return new Derived<Functor>(functor);
		}
	}
	Base* fptr = nullptr;
public:
	template <typename F> 
	function (const F& f): fptr(new Derived<F>(f)) {}
	~function() {
		delete fptr
	}
	function& operator=(const function& other) {
		delete fptr;
		fptr = other.fptr->getCopy();
		return *this;
	}
	Ret operator() (Args... args) const {
		return fptr->operator()(args...);
	}
}
```
Анологично move, т е пишем getMove()....

>[!Warning]
> 1) В операторе () приняли по значению и это нормально, так как всякие & написаны в самой функции, но если там rvalue ссылки, то мы с ними плохо обращаемся. нам нужен forward
> 2) Не буgetCopyдет работать для указателей на методы, так как нужно вызывать по другому (Head head, Tail... tail)![](https://i.imgur.com/vuM1gNK.png)

> 3) Правильнее хранить не указатель, а реальный обьект во многих случаях.
> 4) Virtual - overhead, поэтому в стандартной библиотеке реализована mini таблица virtuak функций
> 5) Не достаточно, чтобы обьекты были move-бильны, они должны и копироваться

```C++
struct is_function<Ret(Args...)>: std::true_type {};
//...
```
[cpp ref is_function](https://en.cppreference.com/w/cpp/types/is_function)

```C++
template<typename T>
struct is_array: std::false_type {};
template<typename T>
struct is_array<T[]>: std::true_type {};
template<typename T, std::size_t N>
struct is_array<T[N]: std::true_type {};
```
[cpp_ref is_array](https://en.cppreference.com/w/cpp/types/is_array)

std::rank - если массив, то выводит размероность, а если нет, то 0
```C++
template<typename T>  
struct rank {  
// using value = 0;  
static constexpr int value = 0;  
};  
  
template<typename T>  
struct rank<T[]> {  
static constexpr int value = rank<T>::value + 1;  
};  
template<typename T, std::size_t N>  
struct rank<T[N]> {  
static constexpr int value = rank<T>::value + 1;  
};  
int main() {  
	std::cout << rank<int>::value << "\n\n";  
	
	std::cout << rank<int[5]>::value << '\n';  
	std::cout << rank<int[5][5]>::value << '\n';  
	std::cout << rank<int[][5][5]>::value << '\n';
}
```

std::is_member_poiner - проверяет явлается ли членом класса

std::conjunction - решение логических уравнений

## SFNE приколы
его конспект

оператор    ,     -- вычислыет все выражение и возвращает последний
