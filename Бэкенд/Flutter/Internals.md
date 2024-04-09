Способы взаимодействия с Engine:
1) Обмен сообщениями с платформой
2) Встроивание нативных виджетов(например карты)
3) Подключение C/C++/... библиотек
Шаги создания federated plugin:
1) создать пакеты
2) plugin_platform_interface
3) реализация
4) подправляем pubspec.yaml
### Каналы
method chanels  - используется в большинстве случаев, когда вызов метода во Flutter запускает нативный метод. Поддерживает асинхронные вызовы методов.
```dart
import 'package:flutter/material.dart';
import 'package:flutter/services.dart';
class CartPage extends StatelessWidget {  
	static const MethodChannel _channel = MethodChannel('co.wawand/stripe'); 
	
	@override  Widget build(BuildContext context) {    
		return Scaffold(     
		 // Widget...    
		); 
	}
}
```
event channel - для передачи потоков данных из нативного кода во Flutter. Возваращает Stream, а не future.
```dart
class TimeHandler : EventChannel.StreamHandler {
	override fun onListen(p0: Any?, sink: EventChannel.EventSink) {
    }
	override fun onCancel(p0: Any?) {
	}
 }

val eventChannel = EventChannel(flutterEngine.dartExecutor.binaryMessenger, "timeHandlerEvent"); // timeHandlerEvent event name
eventChannel.setStreamHandler(TimeHandler) // TimeHandler is event class
```
Пакет Pigeon - пакет для кодогенерации работы с Platform Channels

Отрисовка:
1) VirtualDisplay - картинка, очень быстро, но есть баги
2) Hybrid Composition - отрисовка на невидимом view(нативное view + flutter view)(меньше багов, но медленнее)(Начиная с 19 Android)
3) Texture lauer HC  - что-то новое

FFI - это **foreign function interface, или же интерфейс внешних функций**. Это механизм, позволяющий программе на одном языке вызывать функции, написанные на другом. В случае с Flutter-ом, это возможность вызывать из Dart функции Си из скомпилированных библиотек.
Есть ffigen
### Компиляция
JIT (Just In Time компилятор) - умеет что то докомпилировать в runtime, есть  runtime оптимизации.
AOT (Ahead Of Time компилятор) - компилирует все сразу, нет runtime оптимизации.
Flutter  собирает в JIT в дебаге и в AOT в release.

В центре flutter лежит Dart VM, что позволяет билдиться на разные платформы.
Kernel binary - представление памяти для удобного парсинга

`dart compile jit-snapshot` - помимо kernel binary содержит оптимизированный бинарный код
`dart compile aot-snapshot` - когда компилится подтягивает все либы и все такое, но для ускорения проводят tree shacking(TFA), в котором выбрасываются неиспользуемые участки кода.

Android pipeline:
1) flutter_tools - CLI, pub, devtools, сеть, ...
2) gradle - делает свои шаги, а потом в flutter_plugin вызывает flutter_tools, который компилит flutter код
3) apk
В IOS аналогично, но вместо flutter_plugin, flutter_assemble.
### Flavors
Есть необходимость собирать в разные окружения.
на android - productFlavors, а на IOS - Schemas.
Flutter с помощью flavor умеет это проксировать, а с помощтю target мы сможем узнать flavor.
### Deployment
Android:
1) Icon
2) permissons(например на вызод в интернет)
3) ...
4) signing (SigningConfigs)
Firebase Distribution - распространение среди тестировщиков.
### CI/CD
Firebase - помогает на CI/CD
Github Actions
Объекты:
1) workflow - тригерится на события в репозитории
2) workflow состоит из job, которые будут прогоняться на машине, они делятся на смысловые, типа analyze, build, ...
3) job состоят из step - это скрипты и переиспользование github actions
Пример
```yaml
name Flutter CI
on: push
jobs:
	myjob:
		runs-on: ubunty-latest
		steps:
			uses: action/checkout@v2
			uses: action/flutter-action@v1
			with:
				channel: 'stable'
			run: flutter pub get
			
```
есть матричный build, чтобы одновременно билдить на разные платформы.