Request и Response:
1) Стартовая строка - метод + uri + протокол
2) Заголовки :
	1) запроса ![[Pasted image 20230803175825.png]]refer - откуда пришли, 
	2) ответа![[Pasted image 20230803180711.png]] x0requestid - id запроса, last-modified - время прошлого запроса
3) пустая строка
4) Тело - какой-то контент в каком то виде

### Коды ответов
![[Pasted image 20230803180840.png]]

# Rest API
1) клиент-серверная архитектора
2) отсутствие состояния - помогает для масштабирование
3) кэширование
4) единообрахие интерфейса

# Критерии качества API
1) структурная целостность
2) очевидность
3) консистентность - вся работа похожая
4) читаемость
5) устойчивость к изменениям

# Структура API
1) слабая связанность компонентов (заказ не знает про корзину(про внутрянку))
2) высокоуровненвые абстракции
3) изоляция уровней абстракции (только соседи, корзина и доставка не связаны)
4) разгроничение областей ответственности

![[Pasted image 20230803230017.png]]
![[Pasted image 20230803230508.png]]
![[Pasted image 20230803230614.png]]
![[Pasted image 20230803231014.png]]
item, user, order - можно не использовать если не нужно
не стоит использовать автоинкремент (лучше не генерить на уровне базы)

### Когда нужно вернть 404
Когда выполняются все эти условия:
1) правда неправ клиент
2) когда запрашивается массив опций и пустой массив - это ошибка
3) нештатная ситуация в процессе обработки запроса

## Идемпотентность
это свойство операции сохранять объект при её применении(повторном)
Пример, когда мы создаем заказ и что-то залагало, то мы(клиент) делаем запрос (не ожидая ответа от бэка) снова и снова, но в итоге мы хотим только один заказ
Как это решить?
idempotency_token -  но как их создавать, иожно для каждого заказа генерить бэком его(draft_id)
![[Pasted image 20230806103402.png]]

## Атомарность
Что делать, если пришли данные, но они не все корректные.
например первый товар в заказе норм, а у второго - траблы

## Машинно читаемые ошибки
ошибка должны быть понятна без человека, то есть
![[Pasted image 20230806114243.png]]
code -  это прописанные названия
diff - как бы решение

Порядок ошибок - важен
1) неразрешимые ошибка (закрытие сервиса)
2) критичные для клиента ошибки
3) остальные

![[Pasted image 20230806115016.png]]

## Версионирование
версия API - это мощные глобальные изменения всего
версия end поинта - обратно несовместимые изменения end поинта, например изменения количества обязательных параметров в запросе
![[Pasted image 20230806120002.png]]

# RPC
удаленный вызов процедуры
client node  - сожержит бизнес логику
![[Pasted image 20230806131621.png]]

## JSON-RPC 
способ реализации взаимодействия между клиентом и сервером, отправка json туда и обратно
![[Pasted image 20230806131757.png]]
id - передает клиент, чтобы понять на какой запрос пришел ответ

![[Pasted image 20230806131943.png]]

![[Pasted image 20230806132023.png]]
бэку отвечать не нужно

![[Pasted image 20230806132123.png]]

![[Pasted image 20230806132148.png]]

# SOAP
Simple Object Access Protocol
это альтернативна json-rpc
Запрос:
![[Pasted image 20230806132657.png]]
Ответ:
![[Pasted image 20230806132755.png]]

# gRPC
![[Pasted image 20230806133453.png]]

В качестве IDL используют proto
![[Pasted image 20230806133605.png]]
можно из proto получать: C++/Java/Python/Go/etc

Отличия:
1) в proto3 нет обязательных полей, поэтому есть обратная совместимость
2) запрос можно отменить как на клиенте, так и на сервере
3) можно выставить таймаут с обеих сторон
4) есть заготовки типичных ошибок
5) можно передавать метаданные
6) есть интерцепторы в runtime библиотеках
7) НЕЛЬЗЯ ПОМОТРЕТЬ НА ЗАПРОС, ЭТО БИНАРЬ, можно использовать grpc_cli
![[Pasted image 20230806134619.png]]
![[Pasted image 20230806134717.png]]
![[Pasted image 20230806134736.png]]
Можно в интерактивном резиме в cli evans
Аналог Postman - BloomRPC

Можно взять бинарь так
![[Pasted image 20230806134923.png]]
