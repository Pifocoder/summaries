Event loop - это бесконечный цикл с двумя очередями:
![[Pasted image 20230720175825.png]]
microtask - создается с помощью конcтруктора future.microtask(пример - освобождение памяти)
event - внешнее событие (жест, таймер, ...)
future - результат выполнентя асинх ф-и
состояния future:
1) не начата или не завершена
2) завершена с результатом
3) завершена с ошибкой
![[Pasted image 20230720180247.png]]

Работа с future:
![[Pasted image 20230720180518.png]]
![[Pasted image 20230720180526.png]]
![[Pasted image 20230720180623.png]]
StackTrace - содержит стек вызова
StackTrace.current - стек trace сейчас

## Stream
бывают двух типов:
![[Pasted image 20230720181918.png]]

StreamController - управление stream
![[Pasted image 20230720182010.png]]
![[Pasted image 20230720182052.png]]
