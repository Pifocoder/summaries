## Универсальные ссылки и правила сжатия ссылок
T&& - универсальная ссылка, если T - шаблон (yне класса)
```C++
template<typename T>
void f(T&& x) {}
int main() {
	f(5);
	int y = 5;
	f(y);//T - T&
}
```

## Perfect forwarding problem

```C++ 
template <typename Args>
void emp_back(const Args&& args) {
	if (size_ == cap) {
		reserve;
	}
	new (arr + size_) T(args);//args - lvalue
	size_++;
}
```
std::move ?
```C++
template <typename Args>
void emp_back(const Args&& args) {
	if (size_ == cap) {
		reserve;
	}
	new (arr + size_) T(std::move(args));//но мы не разрешали move
	size_++;
}
```
Проблема...
Решение
```C++
template <typename Args>
void emp_back(const Args&& args) {
	if (size_ == cap) {
		reserve;
	}
	new (arr + size_) T(std::forward<Args>(args));
	size_++;
}
```

args lvalue T -> Args T& -> decltype(args) T&
args rvalue T -> Args T -> decltype(args) T&&
То есть прокинули lvalue как lvalue и rvalue как rvalue

Perfect forward работает, если 
```C++
f(T) == f(std::forward<T>(t)) (с точки зрения типов)
```

Задачка
std::exchange
```C++
template<typename T>  
T exchange(const T& obj, T&& old) {  
  T last_obj = std::move(obj);  
  obj = std::forward<T>(old);  
  return last_obj;  
}
```
