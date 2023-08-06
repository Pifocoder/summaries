```C++
std::sort(v.begin(), v.end(), [](int x, int y) {return x%10 < y % 10;})
auto f = [](int x, int y) {return x%10 < y % 10;};
```
Название и тип функции мы не знаем (рандом)

## Списки захвата
Хотим использовать внешние переменные
```C++
int a = 10;
auto f = [a](int x) -> void {std::cout << x + a;};
```
a - const copy
```C++
int a = 10;
auto f = [a](int x) mutable->void {a += 1; std::cout << x + a;};
```
теперь a можно менять внутри
mutable показывает, что член класса является изменяемым, и его можно изменять в функциях, у которых указан модификатор const, а также у константных объектов.
```C++
int a = 10;
auto f = [&a](int x) {a += 1; std::cout << x + a;};
```
Захвать всех переменных
```C++
int a = 10;
auto f = [=](int x) {std::cout << x + a;};
```
`[this]` - захват всех полей класса
Другой способ

## Внутреннее устройство
```C++
struct Lambda {
	auto operator()(int x, int y) const -> bool {
		return x < y;
	}
	int a;
}
```

## Операции
Copy, std::move

# std::function
Класс в который можно запихнуть любой callable обьект
```C++
std::function<int(int,int)> f = foo;
f(10, 10);
```

function - полиморфная обертка над функциями. Объект function можно хранить, копировать, вызывать.

std::function можно использовать с указателями на методы:
```C++
struct S {
  void foo(int x) {
    std::cout << x + 1;
  }
};

int main() {
  void S(::* p)(int) = &S::foo;
  S s;
  (s.*p)(5);


  std::function<void(S&, int)> f = &S::foo;
  f(s, 5);
}
```

# Union
```C++
union U {
	char a;
	int b;
	string s;
}
int main() {
	U u;
	std::cout << u.a;
}
```
Продолжение  - https://gitlab.com/yaishenka/cpp_course/-/blob/main/lectures/lecture_20.md
