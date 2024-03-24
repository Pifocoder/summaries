Widget - это неизменяемый обьект, все поля final. Widget - это компонент пользовательского интерфейса.

## Виджеты компоновки
### Row
Располагает дочерние элементы в горизонтальном порядке
Параметры:
- `mainAxisAlignmebt` - как дочерние элементы расположены вдоль горизонтальной оси
- `crossAxisAlignment` - как дочерние элементы расположены вдоль вертикальной оси
- `mainAxisSIze` - сколько места виджет компонента будет пытаться занять по горизонтальной оси
### Column
Аналогично row
### Stack
Располагает детей относительно своих границ и своих размеров.

Align - относительное позиционирование.
Positioned - пикселями
### ListView
Row/Column со скроллом. Поддерживает ленивую отрисовку.
Параметры:
- `scrollDirection` - направление скролла

Example:
```dart
body : ListView.builder(
	itemCount: Product.products.length,
	itemBuilder: (context, index) {
	    return ProductLisyItem(
	      title: Text(items[index]),
	    );
	  },
	)
```
Добавим разделители
```dart
body : ListView.separated(
	itemCount: Product.products.length,
	itemBuilder: (context, index) {
	    return ProductLisyItem(
	      title: Text(items[index]),
	    );
	  },
	separatedBuilder : (context, index) => Divider(
		indent : Constants.indent,
		endIndent : ...
	)
)
```
Чтобы работать с API нужно обернуть во FutureBuilder и показывать либо загрузку, либо лист элементов
### GridView
ListView в сетке
### SingleChildScrollView
Контейнер для другого виджеты дающий возможность скролла.

## Основные виджеты
StatelessWidget:
- Отоброжает переданные ему данные, то есть никак не модифицируя
- Не имеет состояния
- Позволяет декомпозировать UI
- Перерисовывается только при команде родителя
StatefulWidget
- Имеет достоп к объекту State
- У State есть собственный жизненный цикл

Изменение состояния виджеты - setState

## Под капотом
Есть Widget, Element, RenderObject
Widget - то просто конфигурация.
Element и RenderObject - очень тяжеловестные штуки
RenderObject - занимается отрисовкой

## Inherited widget
Мы создаем Inherited Widget на общем предке 
И помогает не прокидывать в детей
BuildContext

## Builder
widget альтернативный определению StatelessWidget

### LayoutBuilder
Аналогичен виджету билдер.
Используем когда меняем ориентацию

### FutureBuilder
Виджет, который перестраивает builder на основе взаимодействия с Future.
AsycncSnapshot - текущее состояние взаимодействия с Future.

### StreamBuilder
аналогично FutureBuilder

## Другие полезные виджеты
ClipRRect - виджет, который закругляет
SizedBox - виджет для отступов(типа между элементами list)(заглушка)
