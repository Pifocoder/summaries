* Мльтипарадигменный язык
* поддерживает  объектно ориентированные, императивные и функциональные стили

## История
ECMAScript - скриптовый язык для Netscape
ES - спицификация
* ES1 - 1997 год
* ES2 - 1998 год
* ES3 - 1999 год
* ES5 - 2009 год
* ES6 - 2015 год (let/const, стрелочные)
Сейчас каждый год выходит что-то новое

Определение геолокации openstreetmap

querySelectorAll возвращает не массив

$0.innerText = 'Мой текст'
поменяет текст
$0.innerHTML - подвержен атакам
outerHTML включить еще и оболочку
textContent 

## Cross site scipting
Закидывает JS на другие страницы

#  insertAdjacentHTML insertAdjacentText
Они добавляют разметку и текст в документ и не затрагивают существующие элементы.


# Погружение и всплытие
![](https://i.imgur.com/XhiW90S.png)

```JS
const button = document.querySelectorAll('button');
const pp = document.querySelector('p');
let col = 0;

document.addEventListener('click', function() {
	console.log("Vsplitie doc");
})

document.addEventListener('click', function() {
	console.log("Pogrugenie doc");
}, {capture: true})

button.forEach(function(button) {
	button.addEventListener('click', function(){
		console.log("vsplitie button");
		col++;
		pp.innerText = 'Кликов ' + col;
	});
	
	button.addEventListener('click', function() {
		console.log("Pogrugenie button");
	}, {capture: true})
});
```

Остановили погружение, застрелились
```JS
document.addEventListener('click', function(event) {
	console.log("Pogrugenie doc");
	event.stopPropagation();
}, {capture: true})
```

# Event
isTrusted - true/false (пользователь кликнул/программно кликнули)
path - путь
pointerType - мышка/палец
target - 
currientTarget - 

## Делигирование событий
Повесть обработчик на общий div и добавить условие:
```JS
if (event.targettagName === 'BUTTON') {
...
}
```
Тем самым мы сэкономили несколько обработчиков

Теперь если события разные, тогда:
```HTML
<button data-action="increase">Увеличить</button>
<button data-action="decrease">уменьшить</button>
```

```JS
if ((event.target.dataset.action === 'increase') {
	++number;
} else if (event.target.dataset.action === 'decrease'){
	--number;
}
```

## Удаление обработчиков
```JS
function abac(event) {
	if ((event.target.dataset.action === 'increase') {
		++number;
	} else if (event.target.dataset.action === 'decrease'){
		--number;
	}
	if (number === 8) {
		document.removeEventLisener('click', abac);
	}
}
document.addEventLisener('click', abac);
//document.removeEventLisener('click', abac);
```

### Аргументы обработчиков событий
Для первого клика
```JS
	document.addEventLisener('click', abac, {once : true});
```

# Дебаг
* Можно через браузер -> sorces
* А можно поставить в коде
```JS
debagger;
```
* Стрелка вниз - заходим в функцию
* Стрелка вверх - выходим (пройдя всю функцию)
* Пауза - остановка на ошибках
Scopes - переменные

### Watch выражения
Watch в браузере
event.target

### CallStack
Можем посмотреть путь
