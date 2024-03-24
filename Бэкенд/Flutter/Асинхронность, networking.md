Event loop - это бесконечный цикл с двумя очередями:
![](https://i.imgur.com/xDFFHw8.png)

microtask - создается с помощью конcтруктора future.microtask(пример - освобождение памяти)
event - внешнее событие (жест, таймер, ...)
future - результат выполнентя асинх ф-и
состояния future:
1) не начата или не завершена
2) завершена с результатом
3) завершена с ошибкой
![](https://i.imgur.com/hBjytF3.png)


Работа с future:
![](https://i.imgur.com/Qmw0nXX.png)

![](https://i.imgur.com/arXyV6I.png)

![](https://i.imgur.com/7NGWsHL.png)

StackTrace - содержит стек вызова
StackTrace.current - стек trace сейчас

## Stream
бывают двух типов:
![](https://i.imgur.com/FZTAU9f.png)


StreamController - управление stream
![](https://i.imgur.com/rwBHRfR.png)

![](https://i.imgur.com/v1JCBCC.png)

