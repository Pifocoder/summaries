## Base (дохуя)
https://github.com/8symbols/oop

## Null safety
Dart compiler detects some potential errors at compile time.
Переменные, которые могут быть null, помечаются после названия типа '?'
```dart
String? name  // Nullable type. Can be `null` or string.

String name   // Non-nullable type. Cannot be `null` but can be string.
```
Dart doesn’t set initial values to non-nullable types
```dart
int a;
print(a);//CE
```
Dart все проверяет, чтобы мы например ничего плохого не напечатали(прощлый пример), но если мы уверены, что переменная быдет использоваться ТОЛЬКО после инициализации, то пишем late. (если обманем, то будет RE)
```dart
late int a;
a = 2;
print(a);
```
также можно использовать late для ленивой инициализации
```dart
late String temperature = readThermometer(); 
```
## Const and final
Главное различие между const и final состоит в том, что значение const должно быть определено при компиляции, а значение константы final определяется во время выполнения.

## Операторы
![Pasted image 20230704151042](https://i.imgur.com/PLJUIOg.png)
### Type test operators
![Pasted image 20230704151350](https://i.imgur.com/JYXWOEs.png)
```dart
(employee as Person).firstName = 'Bob';
if (employee is Person) {
  // Type check
  employee.firstName = 'Bob';
}
```

### Conditional expressions
_condition_ ? _expr1_ : _expr2_
If _condition_ is true, evaluates _expr1_ (and returns its value); otherwise, evaluates and returns the value of _expr2_.

_expr1_ ?? _expr2_
If _expr1_ is non-null, returns its value; otherwise, evaluates and returns the value of _expr2_.

### libraries
```dart
// Import only foo.
import 'package:lib1/lib1.dart' show foo;

// Import all names EXCEPT foo.
import 'package:lib2/lib2.dart' hide foo;
```
ленивый импорт
```dart
import 'package:greetings/hello.dart' deferred as hello;
```

## Records
```dart
var record = ('first', a: 2, b: true, 'last');

print(record.$1); // Prints 'first'
print(record.a); // Prints 2
print(record.b); // Prints true
print(record.$2); // Prints 'last'

(num, Object) pair = (42, 'a');

var first = pair.$1; // Static type `num`, runtime type `int`.
var second = pair.$2; // Static type `Object`, runtime type `String`.
```

## Spread operator
он вставляет все элементы другого контейнера в наш
```dart
var list = [1, 2, 3];
var list2 = [0, ...list];
assert(list2.length == 4);
```
## ООП
### Generics
шаблоны
```dart
class Person<T>{
    T id;   // идентификатор пользователя
    String name; // имя пользователя
    Person(this.id, this.name);
 
    void display() => print("id: $id \t name: $name");
}
class Employee<T> extends Person<T>{
    String company;
    Employee(super.id, super.name, this.company);
    @override
    void display(){
        super.display();
         print("Works in $company");
    }
}
```

extends - наследование(нет множественного)
implements - для интерфейсов(абстрактных классов)
mixin
```dart
// Worker может использоваться только как миксин
mixin Worker{
    String company = "";
    void work()=>print("works in $company");
}
class Student{
    String university;
    Student(this.university);
    void study() =>print("studies in $university");
}
class WorkingStudent extends Student with Worker{
     
    String name;
    WorkingStudent(this.name, super.university, String job){
        company = job;
    }
    @override
    void work()=>print("$name works in $company");
    @override
    void study() =>print("$name studies in $university");
}
void main (){
 
    WorkingStudent tom = WorkingStudent("Tom", "MIT", "Google");
    tom.work();      // Tom works in Google
    tom.study();     // Tom studies in MIT
}
```

Геттеры и сеттеры — это специальные методы, которые обеспечивают доступ для чтения и записи свойств объекта. Напомним, что каждая переменная класса определяет геттер, и (если это возможно) — сеттер. Вы можете создать дополнительные свойства для реализации геттеров и сеттеров, используя ключевые слова **get** и **set**:
```dart
 class Student {   
   String name;   
   int age;   
   String get stud_name {   
      return name;   
 }
   void set stud_name(String name) {   
      this.name = name;   
 }
```


## Асинхронное программирование
асинхронные операции - операции, которые позволят выполняться другим операциям, пока выполняется наша.

