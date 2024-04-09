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
3) didUpdateWidget - вызывается при изменении конфигурации
4) sheduleBuildFor - помечен как dirty для ребилда в след цикле
5) build/rebuild 
6) initState - старт
7) dispose - удаление навсегда
8) performeRebuild - вызывает build у виджета
[example](https://gist.github.com/artembark/b579c55af442bc280d1190dd6898ce6f)
[lifecycle example](https://github.com/glebosotov/flutter-education-widget-lifecycle)

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
[example](https://github.com/glebosotov/flutter-education-layout)
[demo](https://medium.com/flutter-community/flutter-the-advanced-layout-rule-even-beginners-must-know-edc9516d1a2)
### Scroll performance
shrinkWrap - говорит ListView посчитать высоту всех детей.
itemExtend - задает константную высоту элементов
[example](https://github.com/glebosotov/flutter-education-scroll-performance)
## Slivers
Они помогают делать классные скролящиеся штуки
Например, когда там нужно, чтобы в скролящемся месте был и List, а после него Grid.
Можно это реализовать с помощью колонки, но тогда, если список большой, то все будет грузиться сразу, поэтому все будет очень долго.
Большой исчерпывающий пример использования: [habr](https://habr.com/ru/articles/657215/)
[example](https://github.com/glebosotov/flutter-education-slivers)
## Отладка
Devtools:
* widget inspector - можно посмотреть все внутрянку виджетов(размеры, constraint) в RunTime.
Profiling
Можно дебаги и логи смотреть, а также получить статистику сколько билдятся какие виджеты. Можно искать лаги и все такое.