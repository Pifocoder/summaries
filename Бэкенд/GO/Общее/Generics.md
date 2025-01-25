>[!important]
>Generic типы переходят в обычные типы на этапе компиляции

A type parameter list
```go
[P, Q aConstraint, R anotherConstraint]
```
aConstraint - имя какого то интерфейса
Пример  использования Generic параметра у функции
```go
func Sort[Elem interface{ Less(y Elem) bool }](list []Elem) {
  ...
}
```
```go
type book struct{...}
func (x book) Less(y book) bool {...}

var bookshelf []book
...
Sort[book](bookshelf) // generic function call
```
Пример использования generic типа, код стал более читаемым:
```go
type Lesser[T any] interface{
  Less(y T) bool
}

func Sort[Elem Lesser[Elem]](list []Elem)
```
Но это конечно сомнительно, т к чтобы пользоваться сортировкой нам надо, чтобы был метод Less у структуры, например у `[]int` этого нет
Решение - использование constraints
```go
// orderedSlice is an internal type that implements sort.Interface.
// The Less method uses the < operator. The Ordered type constraint
// ensures that T has a < operator.
type orderedSlice[T constraints.Ordered] []T

func (s orderedSlice[T]) Len() int           { return len(s) }
func (s orderedSlice[T]) Less(i, j int) bool { return s[i] < s[j] }
func (s orderedSlice[T]) Swap(i, j int)      { s[i], s[j] = s[j], s[i] }

func Sort[T constraints.Ordered](s []T) {
    sort.Sort(orderedSlice[T](s))
}
```
Важно отметить, что у методов нету type парамеров, потому что это сильно усложнилобы проку удовлетворияния класса интерфейсу

constraints.Ordered
```go
type Ordered interface {
    Integer | Float | ~string
}
```
|  - объединение множеств

~string - множество типов, которые являются type дефами на string, типа 
```go
type Mystring string
```

Встроенный constraint comparable, ему удовлетворяют все типы сравнимые через ==
```go
func SetFrom[T comparable](s []T) map[T]struct{} {
    m := make(map[T]struct{}, len(s))
    for _, v := range s {
        m[v] = struct{}{}
    }
    return m
}
```

Пересечение constraint-ов:
```go
type OrderedStringer interface {
    constraints.Ordered  // Type set
    fmt.Stringer         // And stringer as well
}
```
```go
type Int int
func (i Int) String() string { return strconv.Itoa(int(i)) }

func MaxString[T OrderedStringer](a, b T) string {
    if a > b {
        return a.String()
    }
    return b.String()
}
```

## Type inference
Нерабочий пример:
```go
func Scale[E constraints.Integer](s []E, c E) []E {
    r := make([]E, len(s))
    for i, v := range s {
        r[i] = v * c
    }
    return r
}
func ScaleAndPrint(p Point) {
    r := Scale(p, 2)
    fmt.Println(r.String())
}
```
>[!error]
>Compiler error: (type []uint32 has no field or method String)
>То есть при вызове scale произошел каст Point к []uint32 и Scale как раз вернуло []uint32 у которого очев нету метода String() 

Исправление: `func Scale[S ~[]E, E constraints.Integer](s S, c E) S`
```go
func Scale[S ~[]E, E constraints.Integer](s S, c E) S {
    r := make(S, len(s))
    for i, v := range s {
        r[i] = v * c
    }
    return r
}
```
Стоит заметить, что Scale вызывается без указания типов, благодаря type inference.
Так стоит делать всегда, потому что если так не получается сделать, значит ты что-то делаешь не так. 

Type check работает на этапе компиляции, такйо код не скомпилируется:
```go
func invalid[Tx, Ty Ordered](x Tx, y Ty) Tx {
  ...
  if x < y { ...// INVALID
  ...
}
```
>[!error]
>"<" requires that both operands have the same type

Пример constraint-а указателя:
```go
type Pointer[T any] interface {
  *T
}

func f[T any, PT Pointer[T]](p PT)
```

Но Type inference работает не всегда, вот здесь он не сработает, потому что тип output мы узнаем только в возвращаемом результате:
```go
func CallJSONRPC[Output any](method string) (Output, error) {
    var output Output

    resBytes, err := doCall(method)
    if err != nil {
        return output, err
    }

    err = json.Unmarshal(resBytes, &output)
    return output, err
}

res, err := CallJSONRPC[BatchReadResponse]("batch_read")
```

Итог
>[!important]
>Зачем использовать generic?
>1) Мы и правда можем заменить generic type cast + interface{}, но тогда это все будет проверять в runtime, а generic дают возможность проверять типы в compile time.
>2) Generic помогают эффективнее использовать память
>
>Когда не стоит использовать: Примеры дальше

Пример 1:
we can write
```go
func Concat[T fmt.Stringer](a, b T) string {
    return a.String() + b.String()
}
```
but why not just
```go
func Concat(a, b fmt.Stringer) string {
    return a.String() + b.String()
}
```

Пример 2:
Задача хотим сделать type switch на int и string, 
```go
func Mul[T string | int](t T, cnt int) T {
    switch v := any(t).(type) {
    case string:
        v = strings.Repeat(v, cnt)
        return v
    case int:
        v *= cnt
        return v
    }
    panic("impossible type")
}
```
>[!error]
>ошибка компиляции, go не модет проверить, что мы возвращаем тот же тип, что и передали. Очевидно, что мы возвращем тип из множества string | int, но не факт, что тот же

Костыльное решение:
```go
func Mul[T string | int](t T, cnt int) T {
    switch v := any(t).(type) {
    case string:
        v = strings.Repeat(v, cnt)
        return *(*T)(unsafe.Pointer(&v))
    case int:
        v *= cnt
        return *(*T)(unsafe.Pointer(&v))
    }
    panic("impossible type")
}
```

### Links
[- generics design proposal](https://go.googlesource.com/proposal/+/refs/heads/master/design/43651-type-parameters.md)

[- The Go Blog - Why Generics?](https://blog.golang.org/why-generics)

[- GopherCon 2020, Robert Griesemer - Typing [Generic] Go](https://www.youtube.com/watch?v=TborQFPY2IM)