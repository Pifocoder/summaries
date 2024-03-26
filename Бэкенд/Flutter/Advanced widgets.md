## Composition vs inheritance
Composition:
1) оптимизация (const конструкторы, оптимизации фреймворка)
2) Соблюдение flutter style guide
3) гибкость - не ограничиваем себя
Inheritence:
1) Меньше кода

## Внутрянка дерева виджетов
Widget - конфигурация интерфейса
Element - связь Render Object и Widget
Render Object  - рисует пиксели на экране
Layer - оптимизирует рендеринг

Виджеты бывают двух типов:
1) для композиции(composing)  - для описания чего то, то есть не рендерят ничего
2) для рендеринга(rendering) - вляют на интерфейс

Сравнени Element и Widget:
1) Element может меняться, Widget - нет
2) ...

Методы элемента
1) updateChild
2) inflateWidget
3) mount
4) update - поменяли у виджета параметры
5) deactivate/activate - удаляем/добавляем в дерево

Методы виджета:
1) reassemble - дебажная фича
2) didChangeDependencies - реагирует на смену состояния Inhereted Widget
3) sheduleBuildFor - помечен как dirty для ребилда в след цикле
4) rebuild 
5) performeRebuild - вызывает build у виджета

Сравение Stateless и Statefull виджетов:
Stateless вызывает ребилд,  при каждом изменении состояния, а у statefull rebuild вызвается в setState.

Layout - собирает все constraints и применяет и задает размеря и позицию на экране.
Hit testing - обрабатывает нажатия на экран и посылает ивенты слушатеям
Composition - знает о своем родителе в дереве и может посмотреть constrains детей.
### BuildContext
BuildContext - абстрактный класс и его имплементит Element, то есть это как бы элемент. С помощью BuildContext  можно ходить по дереву.
### Key
Иентифицировать виджеты, срванивать виджеты, искать виджет по ключу и мэнеджить стейты.

Примеры: 
1) UniqueKey - каждый раз новый ключ
2) ValueKey (как хэш для модели)
3) GlobalKey - способ откуда угодно обратиться к состоянию виджета
4) ObjectKey - более low level чем ValueKey

### Как работают layout
constraints - идут вниз по дереву
sizes -  идут вверх по дереву

### Scroll performance
shrinkWrap - говорит ListView посчитать высоту всех детей.
itemExtend - задает константную высоту элементов

### Devtools
widget inspector - можно посмотреть все внутрянку виджетов(размеры, constraint) в RunTime.