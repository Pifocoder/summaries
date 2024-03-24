## Идея и простые примеры
```C++
vector<int>v = {1, 4, 2, 3, 6, 3};
std::sort(v.begin(), v.end(), [](int x, int y) {return abs(x - 5) < abs(y - 5);})
```
'[ ]' - lambda expression
```C++
auto f = [](int x, int y) {
	return abs(x - 5) < abs(y - 5);
};
std::sort(v.begin(), v.end(), f);
```

```C++
std::map<int, std::string, delctype(f)>m;
```

Указание типа возвращаемого значение
```C++
auto f = [](int x, int y) -> bool{
	return abs(x - 5) < abs(y - 5);
};
```
Разные типы возвращать нельзя (как в функциях)

Пример
Imidiate invacation
Создал и сразу вызвал
```C++
[](int x, int y) -> bool{
	return abs(x - 5) < abs(y - 5);
}(2, 4);
```
Если нам нужно как то умно инициализировать const переменную, то ришем lambda
```C++
std::cin >> x;
const int c = [](int a){
	if (a < 10) {
		return 1;
	}
	if (a < 20) {
		return 2;
	}
	if (a < 30) {
		return 3;
	}
}(x);
```
Тернарный оператор может все это сделать , но это слишком сложно

## Списки захвата
Хочу исползовать в функции переменные которые не передаются как аргумент
```c++
int mid = 4;
auto f = [](int x, int y) -> bool{
	return abs(x - mid) < abs(y - mid);
};
```
Но так не сработает

```c++
int mid = 4;
auto f = [mid](int x, int y) -> bool{
	return abs(x - mid) < abs(y - mid);
};
```
Создастся копия mid
Чтобы  захватить по ссылке, надо так
```C++
auto f = [&mid](int x, int y) -> bool{
	return abs(x - mid) < abs(y - mid);
};
```
Нужно следить за временем жизни mid, он не должен умереть раньше f

Когда мы захватываем, нельзя менять захваченные элементы, даже при копировании.
Но если надо менять, то 
```C++
auto f = [mid](int x, int y) mutable{
	++mid;
	return abs(x - mid) < abs(y - mid);
};
```
Но если захватываем по ссылке, то менять можно без mutable
Захватываем все по значению:
```C++
auto f = [=](int x, int y) mutable{
	++mid;
	return abs(x - mid) < abs(y - mid);
};
```
Но компилятор оптимизирует, то есть не захватывает переменные, которые не используются в lambda
Захватываем все по ссылке:
```C++
auto f = [&](int x, int y) mutable{
	++mid;
	return abs(x - mid) < abs(y - mid);
};
```

Захватываем все по значению, а mid по значению
```C++
auto f = [=, &mid](int x, int y) mutable{
	++mid;
	return abs(x - mid) < abs(y - mid);
};
//наоборот
auto f = [&, mid](int x, int y) mutable{
	++mid;
	return abs(x - mid) < abs(y - mid);
};
```

### Захват с инициализацией
(можно не называть по другому)
```C++
auto f = [m = mid + 1](int x, int y) mutable{
	++m;
	return abs(x - m) < abs(y - m);
};
```
move
```C++
auto f = [m = std::move(mid)](int x, int y) mutable{
	++m;
	return abs(x - m) < abs(y - m);
};
```
const ref
```C++
auto f = [&mid = std::as_const(mid)](int x, int y) mutable{
	++m;
	return abs(x - m) < abs(y - m);
};
```

Пример с классами
```C++
struct A {
	int a = 10;
	A() {}
	A(const A&) {std::Cout << "copy";}
	A(A&&) {
		std::cout << "move"
	}
	void method() {
		auto f = [this](int x, int y) {//можно (aa = a)
			return abs(x - a) + abs(y - a);
		}
	}
}
```

## Внутрянка lambda
goldbolt.org - показывает асемблер
cppinsides - смотрим как компилятор преобразовывает на код
каждый раз создается класс для lambda
```C++
    auto f = [](int x, int y) {
		return abs(x - 5) < abs(y - 5);
	};
	//эквивалентно
    inline /*constexpr */ bool operator()(int x, int y) const
    {
      return abs(x - 5) < abs(y - 5);
    }
```

```C++
    auto f = [](int x, int y) mutable {
		return abs(x - 5) < abs(y - 5);
	};
	//эквивалентно
    inline /*constexpr */ bool operator()(int x, int y)
    {
      return abs(x - 5) < abs(y - 5);
    }
```

В lambda можно писать auto, потому что, лямбда это оператор (), он становится шаблонным при использовании auto.
```c++
auto f = [](auto x, auto y) {
	return x < y;
}
```
Это в insights
```C++
class __lambda_5_14
  {
    public: 
    template<class type_parameter_0_0, class type_parameter_0_1>
    inline /*constexpr */ auto operator()(type_parameter_0_0 x, type_parameter_0_1 y) const
    {
      return x < y;
    }
    
    #ifdef INSIGHTS_USE_TEMPLATE
    template<>
    inline /*constexpr */ bool operator()<int, int>(int x, int y) const
    {
      return x < y;
    }
    #endif
    
    
    #ifdef INSIGHTS_USE_TEMPLATE
    template<>
    inline /*constexpr */ bool operator()<const char *, const char *>(const char * x, const char * y) const
    {
      return x < y;
    }
    #endif
    
    private: 
    template<class type_parameter_0_0, class type_parameter_0_1>
    static inline /*constexpr */ auto __invoke(type_parameter_0_0 x, type_parameter_0_1 y)
    {
      return __lambda_5_14{}.operator()<type_parameter_0_0, type_parameter_0_1>(x, y);
    }
    
  };
  
  __lambda_5_14 f = __lambda_5_14{};
  std::cout.operator<<(f.operator()(0, 1));
  std::cout.operator<<(f.operator()("aaa", "bbb"));
```
Также можно использовать шаблонные типы
```C++
auto f =  []<tyename T>(const T& first, const T& second) {
	return first < second;
};
```