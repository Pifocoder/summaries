У нас есть только один поток.
EventLoop - абстракция работы с потоками

```JS
console.log(1);

setTimeout(() => {
	console.log(2);
}, 0)

console.log(3);

//1 3 2
```
setTimeout - функция которая откладывает таймер, webapiб закладывает задачу в конец очереди

```JS
console.log(1);

const start = Date.now();

setTimeout(() => {
	console.log(Date.now() - start); // 4ms - 8ms
}, 0)

console.log(3);
```

## Promise
Так было раньше
```JS
const a = new XMLHttpRequest()

a.open()

a.onload(() => {
	const b = new XMLHttpRequest()
	b.open();
	b.onload((data2) => {
	
	})
})
```
fetch - promise
А теперь Promise
```JS
const p = new Promise((resolve, reject) =>{
	setTimeout(() => {
		resolve({ login: 'vas'});
	}, 1000);
})

const req = p;

req.then((data) => {//успешно
	console.log(data);
})
req.catch((err) => {//ошибка
	console.log(data);
})
```
Сработает then, т к resolve
Если бы был reject, то сработал бы catch

Внутри promise код синхронный
```JS
console.log(1);
const p = new Promise((resolve, reject) =>{
    console.log(2);
	setTimeout(() => {
		resolve({ login: 'vas'});
	}, 1000);
})

const req = p;

req.then((data) => {//успешно
	console.log(3);
	console.log(data);
})
req.catch((err) => {//ошибка
	console.log(data);
})
console.log(4);
//1 2 4 3
```

then и catch создают новые promise
Цепочки
```JS
req.then((data) => {//успешно
	console.log(3);
	console.log(data);
}).catch((err) => {
	//...
})
```
Правило цепочек - при reject вызовется первый следующий catch, если успечно, то then.
Цепочки promise независимые.

в очередь кладет задачу не resolve, а then/ctach.

Есть две очереди задач:
* макрозадачи - все остальные
* микрозадачи - пораждаются promise(then/catch) и ручное планирование (queue microtask)

Пока не выполнятся все микрозадачи не начнем выполнять макрозадачи.

## Как превратить XMLHttpRequest в promise
```JS
function callApi (url) {
	return new Promise((resolve, reject) => {
		const request = new XMLHttpRequest();
		//...
		request.onload = () => {
			resolve();
		}
		request.onerror = () => {
			reject();
		}
	})
}
```

Как работает это под капотом - turbofan.
v8 - движок chrome
webkit - движок safari
geka - mozilla

## Выполнение js
2 прохода:
* 1 проход - скан, чтобы можно было вызывать ф-ю из конца файла
* 2 проход:
	* интерпретирование (ignition)
	* Bytecode
	* Turbofan пытается соптимизировать Bytecode ну или деоптимизирует
![](https://i.imgur.com/ITuBlKy.png)


## Delay
```JS
function delay(timeout) {
	return new Promise((number) => {
		setTimeout(number, timeout);
	})
}
```

## Promise race
```JS
function timeout(p, time) {
	return new Promise((res, rej) => {
		p.then(res)
		delay(time).then(rej);
	})
}
```

## Promise all
ждем пока все promise выполнятся
```JS
Promise.all([
	delay(1).then(() => "qwe"),
	delay(10).then(() => "qwe")
]).then((data) => {
	console.log("123");
})
```

## Promise.allSettled
Вместо all, плказывает какие успешные, а какие нет

Пример работы с api
```JS
const card = document.getElementById('card-template');  
const cardList = document.querySelector('.card-list');  
  
function renderCard(url) {  
  const newCard = card.content.querySelector('.card').cloneNode(true);  
  newCard.querySelector('.card__link').href = url;  
  newCard.querySelector('.crad__image').src = url;  
  cardList.append(newCard);  
}  
  
function loadCards() {  
  fetch('https://api.thecatapi.com/v1/images/search/?limit=10')  
     .then(response => {  
     return response.json();//promise встроится в цепочку, т к метод json - async  
  })  
  .then(data => {  
     data.forEach(element => {  
        renderCard(element.url);  
     });  
  })  
}  
  
const button_load = document.querySelector('.add-cats');  
button_load.addEventListener('click', () => loadCards());  
loadCards();
```

```HTML
<!DOCTYPE html>  
<html lang="ru">  
   <head>  
   <meta charset="UTF-8">  
   <meta name="viewport" content="width=device-width, initial-scale=1.0">  
   <link rel="stylesheet" href="./index.css">  
   <title>Котики</title>  
</head>  
<body>  
<section class="card-list">  
</section>  
<button class="add-cats">LOAD MORE</button>  
<template id="card-template">  
   <div class="card">  
<img src="" class="crad__image">  
<a href ="" class="card__link" target="_blank">Cсылка</a>  
   </div>  
</template>  
<script src="./api.js"></script>  
</body>  
</html>
```

```CSS
.card-list {
    display: flex;
    flex-wrap: wrap;
    justify-content: space-around;
}
.card {
    display: flex;
    flex-wrap: wrap;
    flex-direction: column;
}
.card__link {
    line-height: 30px;
    font-size: 30px;
    text-decoration: none;
    color: black;
}
.crad__image {
    height: 300px;
    width: 270px;
    object-fit: cover;
}
.add-cats {
    margin: 0px auto;
    background-color: red;
    height: 200px;
    width: 1000px;
    
    font-size: 100px;
}
```