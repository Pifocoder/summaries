## Move-семантика и rvalue ссылки
### Идея и базовые примеры
``` C++
template<typename T>
void swap(T& x, T& y) {
	T t = x;
	x = y;
	y = t;
}
```

Если типы большие, то это жесть (для строк работает за линию)
Также медленно работает reserve дла массива строк
введем ф-ю std::move
``` C++
std::vector<std::string>v;
v.push_back(std::move("acnaskcnakjscnakjscnaksc"));
//копирование не произойдет
//и тут
template<typename T>
void swap(T& x, T& y) {￼String
	T t = std::move(x);
	x = std::move(y);
	y = std::move(t);
}
```

Как сделать std::move для своих типов
нужно доопределить к свои типам construct oper asign, ...
move конструктор
``` C++
class String {
  public:
	String(String&& s): arr(s.arr), sz(s.sz), cap(s.cap) {
		s.arr = nullptr;
		s.sz = s.cap = 0;
	}
	
	String& operator=(String&& s) {
		String copy = std::move(s);
		swap(copy);
		return *this;
	}
  private:
	char* arr;
	size_t sz;
	size_t cap;
};
```

## Move  конструктор по умолчанию
``` C++
String (String&& str) = default;
```

Если в классе определени оператор копирования, но не определен Move конструктор, то Move конструктор будет работать, как copy конструктор

## Правило пять
Если copy контструктор не трив, то нужно определить move конструктор и move оператор присваиванию и наоборот.


## std::move implementation

``` C++
template<typename T>
T&& naive_move(T& x) noexcept {
	return static_cast<T&&>(x);
}
```

## Правила
T& &=> T&
T&& & => T&
T& && => T&
T&& && => T&&
Именно по этому не работает реализация выше

## Настоящий move

``` C++
template<typename T>
std::remove_reference_t<T>&& move(T&& x) noexcept {
	return static_cast<std::remove_reference_t<T>&&>(x);
}
```
Почему мы принимаем именно T&&?

# Формальное определение lvalue и rvalue
lvalue и rvalue - это категории выражений (expressions), а не типов.
Определение, просто таблица

| lvalue                            | rvalue                                 |
| --------------------------------- | -------------------------------------- |
| identifier x                      | literal                                |
| binary = += -= ...                | binary +-* / ...  for standart type    |
| unary * or [] for standart type   | unary & + -                            |
| prefix ++ or -- for standart type | postfix ++ or --for standart type      |
| a & b : c if b and c are lvalue   | a ? b : c if one of b and c rvalue     |
| a, b; if b if lvalue              | a, b; if b rvalue                      |
| if ret type is lvalue reff        | if ret type is rvalue reff ot not reff |
| cast to lvalue                    | cast to non-ref or rvalue              |

``` C++
String("abc" ) = "cde";
Работает, но не должно

### Ref qilifire
* *& - метод только для lvalue
* *&& - этот метод только для rvalue.
* ничего - для любого
Ставится перед {}
Это может позволить делать некоторые операции эффективнее.

# Rvalue ссылки и их свойства

``` C++
int mian() {
	int x = 0;
	int&& r = x;//CE //x - lvalue
	int&& r = 1;
	int&& rr = r;//CE //нам все равно на тип, категория переменной lvalue
	int&& r = std::move(x);
	//Теперь мы можем написать 
	r = 2;//x станет равнымv 2

	const int&& c = 5;
	c = 6;//CE
	const int&& y = r;//CE r - rvalue
}
```

``` C++
void push_back (T&& value) {
	...
	new (newarr + sz) T(std::move(value));//если сделать без move, все равно копирнется
	...
}
```

# Perfect forward
### emplace_back
```C++
vector<std::string> vs;
vs.emplace_back(5, 'a');
```
то есть мы избегаем одного промежуточного создания
```C++
class Vector {
	...
	template<typename... Args>
	void emplace_back(const Args&... args) {
		if (sz >= cap) {
            T* newarr = reinterpret_cast<*T>(new char [cap*2 * sizeof(T)]);
            size_t i = 0;
            try {
                for (size_t i = 0; i < sz; ++i) {
                    new (newarr + i) T(args);//хотим тут написать move, но не можем, так как непонятно какие аргументы rvalue а какие нет
                }
                new (arr + sz) T(value);
            } catch (...) {
                for (size_t j = 0; j < i; ++j) {
                    (newarr + j)->~T();
                }
                delete[] reinterpret_cast<char*>(newarr);
                throw;
            }
            
            for (size_t i = 0; i < sz; ++i) {
                (arr + i)->~T();
            }
            
            delete[] reinterpret_cast<char*>(arr);
            arr = newarr;
            cap *= 2;
        } else {
            new (arr + sz) T(args);
        }
        ++sz;
	}
	void push_back(T&& value) {
		emplace_back(std::move(value));
	}
	void push_back(const T& value) {
		emplace_back(value);
	}
}
```

push_back выражаем серез emplace_back
```C++
void push_back(T&& value) {
	emplace_back(std::move(value));
}
void push_back(const T& value) {
	emplace_back(value);
}
```

Оу, чет как-то без move грустно
```C++
new (newarr + i) T(args);
```
хотим тут написать move, но не можем, так как непонятно какие аргументы rvalue а какие нет
Вот как раз это проблема Perfect forwarding

## Универсальная ссылка
То есть мы хотим уметь принимать как lvalue, так и rvalue и дальше пробрасывать с сохранением вида value.
Придумали костыль:
Если функция принимает тип T&& и T - шаблонный параметр ф-и, то это ссылка не считается rvalue, по ней можно принимать и lvalue и rvalue - универсальная ссылка
Замечание
здесь не универсальная ссылка, тк T - шаблонный параметр Vector
```C++
void push_back(T&& value) {
	emplace_back(std::move(value));
}
```

## Как работает forward?
```C++
template <typename T>
T&& forward(std::remove_reference_t<T>& x) noexcept {
	static_assert(!std::is_lvalue_reference_v<T>);
	return static_cast<T&&>(X);
}
```

Почему вот эти forward не работают
```C++
template <typename T>
T&& forward(T&& x) noexcept {
	return static_cast<T&&>(X);
}
```

```C++
template <typename T>
T&& forward(T& x) noexcept {
	return static_cast<T&&>(X);
}
```
Идеальная передача позволяет создавать функции-обертки, передающие параметры без каких-либо изменений (lvalue передаются как lvalue, а rvalue – как rvalue) и тут std::move нам не подходит, так как она безусловно приводит свой результат к rvalue.

# xvalues and copy elision
```C++
BigInteger operator+(const BigInteger& a, const BigInteger& b) {
	BigInteger copy = a;
	copy += b;
	return copy;
}
```
Компилятор сделает Return Value Optimization, в Асемблерном коде компилятор создаст переменную copy уже на месте, где должно лежать возвращаемое значение.(RVO) (Компилятор не обязан так делать)

```C++
BigInteger operator+(const BigInteger& a, const BigInteger& b) {
	return BigInteger(a += b);
}
```
Тут тоже не будет создание BifInteger локально, он создаст его сразу на месте возвращаемого значения. (100% так будет) 

copy elision - это оптимизация копирования, компилятор не копирует и не мувает, а сразу создает переменную там где надо, там не только RVO, а много других  приколов.
НО компилятор оптимизирует lvalue.

```C++
struct S {
	S() {
		std::cout << "create";
	}
	S(const S&) {
		std::cout << "copy";
	}
};
S x = S(S(S(S(S(S)))))
```
Вызовется только 1 конструктор create.

## Expressions since C++11
prvalue - pure rvalue (чистое rvalue)
xvalue - просроченое lvalue
glvalue - generalized lvalue.

xvalue - это то что было мувнуто, то есть обьект который существует в памяти, но сейчас трактуется как rvalue

благодаря этим штукам и оптимизируются всякие штуки

```C++
S(S(S(S(S(S))))).x;
```
было prvalue, вообще бы ничего не вызвалось, но теперь стал xvalue, применилось temporary materialization

## std::move_if_noexcept
Возращает std::conditional

```C++
template<typename T>
std::conditional_t< std::is_nothrow_move_constuctible_v<T>, T&&, const T&>
move_if_noexcept(T& x) noexcept {
	return std::move(x);
}
```
При перекладываннии обьектов move, только есил move-конструктор noexcept

## Доп материалы
[Move vs forward](https://habr.com/ru/articles/568306/)
![](https://i.imgur.com/mSPEyTM.png)
]]