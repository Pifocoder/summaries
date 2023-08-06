std::range - диапазон, над которым реализованы много алгосов
`ranges::begin` - iterator
`ranges::end`- sentinel ()

```C++
#include <ranges>

std::vector<int>input = {1,2, 3,4, 5};
auto output =  input
   | std::views::filter([](const int n) {return n % 3 == 0;)})
   | std::views::transform([](const int n) {return n * n; )});
```
| - pipe (как в консоли), то есть input передается далее






TO BO CONTINUED