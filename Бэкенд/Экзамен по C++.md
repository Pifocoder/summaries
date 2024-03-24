# Приоритет операций в c++
1 - самый высокий
если приоритеты совпадают то в силу assocativity
![[Pasted image 20230610200906.png]]

# Указатели
![[Pasted image 20230610092527.png]]

## Указатели на методы
```C++
struct Base {  
	void f() {  
		std::cout << "func" << std::endl;  
	}  
	int g(double y) {  
		std::cout << y << std::endl;
		return 1;  
	}
	int x;
};  
int main() {
	Base b;
	
	void (Base::* ptr)() = &Base::f;   
	(b.*ptr)();  
	
	int (Base::* ptr2)(double) = &Base::g;  
	int (Base::* ptr3) = &Base::x;
	std::cout << (b.*ptr2)(2.0); 
}
```

## Констанотные указатели
```C++
int val = 20;
int* const p = &val;
*p = 10;//можно
++p; //нельзя

const int* pp;
pp = &val;
*pp = 10//нельзя
++pp;//можно

const int* const ppp = &val;
*ppp = 10//нельзя
++ppp;//нельзя
```

## Перегрузка функций
![[Pasted image 20230610100008.png]]
1) создается набор кандидатов
2) фильрация, остаются только жизнемпомобные, то есть те у который кол-во переменных совпадает с требуевым, учитывая всякие переменные по умолчанию и многоточия, а так же требуется чтобы для каждого аргумента существовала какая-то последовательности преобразований в аргумент ф-и, а также если ссылочный тип то проверяется lvalue == rvalue
3) Среди жизнеспособных выибирается ЛУЧШАЯ
	1) Существует хотя бы один элемент неявное преобразование которого лучше 
	2) При равенстве 1, неявное преодбразование возвращаемого типа лучше
	3)  При равенстве 2, нешаблонная лучше специализации
	4) При равенстве 3, лучшая специализация

# Понятие CE, RE, UB
CE - не получилось содать исполняемыц файл
1) Лексическая - не получилось разбить на токены (24zjcnaksnckjasc)
2) синтаксическая, забыл ;
3) Семантическая, вызов не определенного метода

RE - ошибка времени исполнения
1) stack overflow, компилятор что то не смог прймать в Compile time

UB - когда что -то не прописано в стандарте

# Память
Стаческая выделяется перед вызовом программы, значения переменыых инициализируется один раз
Динамическая память - new/delete

# Ссылки
как бы новое название переменной
```C++
type x;
type& y = x;

const type& y = 5;
const type& yy = x;//y - нельзя менять

```

## Mutable
Мы делаем поле класса mutable, например mutable int x, есди хотим сделать константный метод, в котором ихменяется как раз это поле.

# Friend
```C++
#include <iostream>
  
class Auto
{
    friend void drive(const Auto&);
public:
    Auto(std::string autoName, unsigned autoPrice) 
    { 
        name = autoName; 
        price = autoPrice;
    }
    void print()
    {
        std::cout << name << " : " << price << std::endl;
    }
  
private:
    std::string name;   // название автомобиля
    unsigned price;  // цена автомобиля
};
  
void drive(const Auto &car) 
{ 
    std::cout << car.name << " is driven" << std::endl;
}
  
int main()
{
    Auto tesla("Tesla", 5000);
    tesla.print();  //
    drive(tesla);
}
```

# Explicit
Если мы хотим запретить неявные касты

# Виртуальные функции
полиморфный объект (у класса которого есть хотя бы один виртуальный
метод) в памяти.

## RTTI and dynamic cast
```C++
class Base {
	virtual void f() {
		std::cout << 1;
	}
};
class Dirived: Base {
	void f() override {
		std::cout << 2;
	}
};

int main() {
	inr x;
	std::cin >> x;
	Base* p = x > 0 ? new Base() : new Derived();
	
	p->f();//не понятно что будет, поэтому в RunTime нужно знать тип
}
```
Это поле - typeid, оно лежит в каждом обьекте
typeid(p).name() (количество букв в название типов  + название типов), также типы можно сравнивать ==

Хотим кастить вниз


## Касты
![[Pasted image 20230613000952.png]]
static_cast - статический каст, так как не знает в Run time ничего (все проверяет в Compile time)
dynamic_cast - каст в run time с проверкой того типа, который в run time (ТОЛЬКО ПОЛИМОРФНЫЕ ТИПЫ). Если каст не получился словим RE, std::bad_cast

![[Pasted image 20230611142128.png]]
![[Pasted image 20230611142136.png]]
![[Pasted image 20230611142151.png]]
## Virtual function 
1) компилятор, когда создает наследника, сначала проставляет ему указатель vtable родителя, а когда родитель закончил инициализация, то vptr начинает указывать на таблицу наследника
```C++
struct Base {
	virtual void f() = 0;
	Base() {
		f();
	}
}
struct Derived {
	void f() override {
		std::cout << "der";
	}
	Derived() {
		
	}
}
int main() {
	Derived d;
}
```
то есть тут будет ошибка компиляции, но если вызывать f через доп функцию h, то будет RE.
Во время деструктирования обьектов с vptr все идет в обратном порядке.

2) аргументы по умолчанию - это compile time штуки, и при использовании виртуальных функций(которые runtime) будут подставляться те которые, были в у обьекта, которого мы по compile time вызываем.
3) Указатели на методы в случае виртуальных функций работают как обычные функции, так как в случае выиртуальных функций указатель на метод хранит не адрес функции, а сдвиг относительно начала таблицы и сдвиг относительно начал обьекта, если мы вызываемся от родителя, но если обычный указатель то все проще.

# Шаблоны
## Перегрузка шаблонной функции
• Частное важнее общего;
• Чем меньше шаблонов, тем лучше, но каст хуже(то есть лучще без каста, но с шаблонами)
```C++
template<typename T>  
void function(T x) {  
	std::cout << x << " ";  
}  
void function(double x) {  
	std::cout << x * 2 << " " ;  
}  
void function(int x) {  
	std::cout << x * 3 << " " ;  
}  
int main() {  
	function(2);  
	function(1.2);  
	long long x = 2;  
	function(x);  
}
//6 2.4 2
```

## Специализация шаблонов
```C++
template <typename T>
class Class {};

template <>
class Class<int> {};
```
Важен порядок специализации

Когда у компилятора есть неоднозначность: тип или объект, то по умолчанию он выбирает объект. Чтобы обратиться типу, то надо писать typename.
## Fold Expression 
```C++
struct d {  
	bool empty() const {  
		return true;  
	}  
};  
template<typename... Args>  
bool all_empty(const Args&... args) {  
	return (args.empty() && ...);  
}
```

# Исключения
```C++
try {  
	throw std::out_of_range("It's cool");  
} catch (const std::out_of_range& err) {  
	std::cout << err.what() << std::endl;  
}
```

# Контейнеры и итераторы
![[Pasted image 20230611195446.png]]

# Move семантика
```C++
template<typename T>
T&& forward(std::remove_reference_t<T>& val) {
	return static_cast<T&&>(val);
}
template<typename T>
std::remove_reference_t<T>&& move(T&& x) noexcept {
	return static_cast<std::remove_reference_t<T>&&>(x);
}
```
![[Pasted image 20230611231018.png]]

# Типы
```C++
decl
```

# std::function
```C++
int foo(int x, int y) {
  return x + y;
}
struct S {
  int operator()(int x, int y) const {
    return x / y;
  }
}
int main() {
  std::function<int(int, int)> f = foo;
  f(10, 10);
  f = S();
  f(10, 10); // мувнул в себя S
  S s;
  f = s;
  f(10, 10); // скопировал в себя S
  f = [](int x, int y) {return x * y; };
}
```
## any
```C++
std::any a = 5;
std::vector<int> v = {1, 2, 3, 4 ,5};
a = std::move(v);
a = 'a';
std::cout << std::any_cast<char>(a);
```
Реализация function
```C++
template <typename... Types>
struct function;
template <typename Res, typename... Args>
struct function<Res(Args...)> {
  struct Base {
    virtual Base* get_copy() = 0;
    virtual void destroy() = 0;
    virtual Res operator()(Args&&...) const = 0;
    virtual ~Base() = default;
  };
  template <typename Functor>
  struct Derived: public Base {
    Functor f;
    Derived(const Functor& f): f() {}
    Base* get_copy() override {
      return new Derived<Functor>(f);
    }
    void destroy() override {
      f.~functor();
    }
    Res operator()(Args&&... args) const override {
      return std::invoke(f, std::forward<Args>(args)...); // тут std::invoke из-за того что это может быть указателем на член
    }
  }
}
```

# Концепты
![[Pasted image 20230612213554.png]]

![[Pasted image 20230612214123.png]]

![[Pasted image 20230612214041.png]]

Но они достаточно тупые, так как сравниваются лексемами
![[Pasted image 20230612214312.png]]
Проверяет компилиируется ли, а не сравнивает(нет вычислений)
функционал сохранения reqirments

![[Pasted image 20230612214548.png]]

![[Pasted image 20230612214947.png]]

https://github.com/ivanovmipt/cpp-examples/tree/master

