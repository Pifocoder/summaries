Механизм для чтения и изменения значений, не зная их реальный тип момент компиляции.
Пример использования: `fmt`, `encoding/json`, testify DeepEqual.

И как же это все работает?
Пакет `reflect` определяет два основных типа: `reflect.Type` и `reflect.Value`.

Начнем с `reflect.Type
```go
func TypeOf(interface{}) Type
```
TypeOf принимает интерфейсный объект, а возвращает как бы ту часть интерфейса, которая отвечает за мета информацию типа.
```go
t := reflect.TypeOf(3)  // a reflect.Type
fmt.Println(t.String()) // "int"
fmt.Println(t)          // "int"
```
`Sprintf` формат `%T` использует `TypeOf` внутри. (`fmt.Printf("%T\n", 3) // "int"`)

Теперь `reflect.Value`
`reflect.Value` хранит значение любого типа.
```go
v := reflect.ValueOf(3)
fmt.Println(v)          // "3"
fmt.Printf("%v\n", v)   // "3"
fmt.Println(v.String()) // NOTE: "<int Value>"
```
Метод `.Interface()` - обратная операция к `reflect.ValueOf`.
```go
v := reflect.ValueOf(3) // a reflect.Value
x := v.Interface()      // an interface{}
i := x.(int)            // an int
fmt.Printf("%d\n", i)   // "3"
```


Пример использования:
```go
func Any(value interface{}) string {
    return formatAtom(reflect.ValueOf(value))
}

// formatAtom formats a value without inspecting its internal structure.
func formatAtom(v reflect.Value) string {
    switch v.Kind() {
    case reflect.Invalid:
        return "invalid"
    case reflect.Int, reflect.Int8, reflect.Int16, reflect.Int32, reflect.Int64:
        return strconv.FormatInt(v.Int(), 10)
    case reflect.Uint, reflect.Uint8, reflect.Uint16, reflect.Uint32, reflect.Uint64, reflect.Uintptr:
        return strconv.FormatUint(v.Uint(), 10)
    // ...floating-point and complex cases omitted for brevity...
    case reflect.Bool:
        return strconv.FormatBool(v.Bool())
    case reflect.String:
        return strconv.Quote(v.String())
    case reflect.Chan, reflect.Func, reflect.Ptr, reflect.Slice, reflect.Map:
        return v.Type().String() + " 0x" + strconv.FormatUint(uint64(v.Pointer()), 16)
    default: // reflect.Array, reflect.Struct, reflect.Interface
        return v.Type().String() + " value"
    }
}
```
```go
var x int64 = 1
var d time.Duration = 1 * time.Nanosecond
fmt.Println(format.Any(x))                  // "1"
fmt.Println(format.Any(d))                  // "1"
fmt.Println(format.Any([]int64{x}))         // "[]int64 0x8202b87b0"
fmt.Println(format.Any([]time.Duration{d})) // "[]time.Duration 0x8202b87e0"
```

Теперь хотим сделать более сложну печать Debug Display:
```go
type Movie struct {
    Title, Subtitle string
    Year            int
    Color           bool
    Actor           map[string]string
    Oscars          []string
    Sequel          *string
}

Display("strangelove", strangelove)
```
Вывод:
```
Display strangelove (display.Movie):
strangelove.Title = "Dr. Strangelove"
strangelove.Subtitle = "How I Learned to Stop Worrying and Love the Bomb"
strangelove.Year = 1964
strangelove.Color = false
strangelove.Actor["Gen. Buck Turgidson"] = "George C. Scott"
strangelove.Actor["Brig. Gen. Jack D. Ripper"] = "Sterling Hayden"
strangelove.Actor["Maj. T.J. \"King\" Kong"] = "Slim Pickens"
strangelove.Actor["Dr. Strangelove"] = "Peter Sellers"
strangelove.Actor["Grp. Capt. Lionel Mandrake"] = "Peter Sellers"
strangelove.Actor["Pres. Merkin Muffley"] = "Peter Sellers"
strangelove.Oscars[0] = "Best Actor (Nomin.)"
strangelove.Oscars[1] = "Best Adapted Screenplay (Nomin.)"
strangelove.Oscars[2] = "Best Director (Nomin.)"
strangelove.Oscars[3] = "Best Picture (Nomin.)"
strangelove.Sequel = nil
```
Реализация:
```go
func Display(name string, x interface{}) {
    fmt.Printf("Display %s (%T):\n", name, x)
    display(name, reflect.ValueOf(x))
}
func display(path string, v reflect.Value) {
    switch v.Kind() {
    case reflect.Invalid:
        fmt.Printf("%s = invalid\n", path)
    case reflect.Slice, reflect.Array:
        for i := 0; i < v.Len(); i++ {
            display(fmt.Sprintf("%s[%d]", path, i), v.Index(i))
        }
    case reflect.Struct:
        for i := 0; i < v.NumField(); i++ {
            fieldPath := fmt.Sprintf("%s.%s", path, v.Type().Field(i).Name)
            display(fieldPath, v.Field(i))
        }
    case reflect.Map:
        for _, key := range v.MapKeys() {
            display(fmt.Sprintf("%s[%s]", path,
                formatAtom(key)), v.MapIndex(key))
        }
    case reflect.Ptr:
        if v.IsNil() {
            fmt.Printf("%s = nil\n", path)
        } else {
            display(fmt.Sprintf("(*%s)", path), v.Elem())
        }
    case reflect.Interface:
        if v.IsNil() {
            fmt.Printf("%s = nil\n", path)
        } else {
            fmt.Printf("%s.type = %s\n", path, v.Elem().Type())
            display(path+".value", v.Elem())
        }
    default: // basic types, channels, funcs
        fmt.Printf("%s = %s\n", path, formatAtom(v))
    }
}
```

Изменение значений через reflect:
```go
d := reflect.ValueOf(&x).Elem()
d.SetInt(3)
fmt.Println(x) // "3"
```
Приватные поля можно читать, но нельзя менять.

Методы структуры/интерфейса в reflect
```go
func Print(x interface{}) {
    v := reflect.ValueOf(x)
    t := v.Type()
    fmt.Printf("type %s\n", t)

    for i := 0; i < v.NumMethod(); i++ {
        methType := v.Method(i).Type()
        fmt.Printf("func (%s) %s%s\n", t, t.Method(i).Name(),
            strings.TrimPrefix(methType.String(), "func"))
    }
}
```
`v.NumMethod()` - при вызове от структуры вернет количество публичных методов, при вызове от интерфейса вернет количество публичных и привытных методов.

Для интерфейсов:
Результат `ValueOf` - всегда конкретный тип.
```go
var w io.Writer
t := reflect.TypeOf(w) // t == ???

var v interface{} = w  // v == nil 
t = reflect.TypeOf(v)  // t == ???
```
Чтобы получить `reflect.Type` равный интерфейсу, нужно использовать промежуточный указатель.
```go
var w io.Writer
ptrT := reflect.TypeOf(&w) // ptrT == *io.Writer
t    := ptrT.Elem()        // t == io.Writer
```

Итог:
- `reflect` создаёт хрупкий код. Там, где была бы ошибка компилятора, возникает `panic` во время исполнения.
- Поведение `reflect` кода нужно документировать отдельно, и оно не ясно из типов аргументов.
- `reflect` работает медленнее, чем специализация под конкретный тип.

Пример использования:
C помощью reflect перевернули map:
```go
func ReverseMap(forward interface{}) interface{} {
	v := reflect.ValueOf(forward)
	t := reflect.TypeOf(forward)
	keyType := t.Key()
	valueType := t.Elem()

	switch v.Kind() {
	case reflect.Map:
		result := reflect.MakeMap(reflect.MapOf(valueType, keyType))
		for _, key := range v.MapKeys() {
			value := v.MapIndex(key)
			result.SetMapIndex(value, key)
		}
		return result.Interface()
	default:
		return nil
	}
}
```