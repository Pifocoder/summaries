## The Stack in x86
Стэк растет вниз

В ячейке стэка хранятся:
rsp - указатель на вершину стэка
rpd - указатель на конец предыдущей вершины стэка

[Отсюда](https://eli.thegreenplace.net/2011/02/04/where-the-top-of-the-stack-is-on-x86/)

## The Caller's rules
1) Чтобы вызывать функции, нам надо сохранять наши данные в каком то регистре. Используют r10 и r11 и делают push на стэк наших данных
2) Надо передавать какие то данные в функцтю, поэтому сохраним их в решистрах: rdi, rsi, rdx, rcx, r8, r9) если нехватает, то на стэк сохраняем
3) Потом вызывается функция call, которая записывает место возврата
4) Потом все чистим
Гарантии функции, которую вызывают: ...
[wiki](https://en.wikipedia.org/wiki/X86_calling_conventions#System_V_AMD64_ABI_convention)
```
foo:
	push callee-saved
	...
	pop callee-saved
	pop ret
	
main:
	push caller.saved
	call foo
	pop caller.saved
```
![](https://i.imgur.com/UVJu0OG.png)

[stackoverflow](https://stackoverflow.com/questions/18024672/what-registers-are-preserved-through-a-linux-x86-64-function-call)

```псевдо
Context ctx
```

```cpp
struct Context {
	void Setup(vaoid* stack, ITrampoline trampolhttps://en.wikipedia.org/wiki/X86_calling_conventions#System_V_AMD64_ABI_conventionine);
	void Switch(Context& to);
}
```
ITrampoline - какой угодно класс, у которого есть Run
Setup - сощдание контекста
Switch - переключение в заданный контекст

