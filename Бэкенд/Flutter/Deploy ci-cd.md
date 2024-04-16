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
example
```
flutter run --target=lib/app/targets/beta/main.dart --flavor=beta
```
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
### Deployment
Android:
1) Icon
2) permissons(например на вызод в интернет)
3) ...
4) signing (SigningConfigs)
Firebase Distribution - распространение среди тестировщиков.
### [Семинар](https://github.com/nikberX/flutter_flavors_example)
[github market place](https://github.com/marketplace?type=) - место, где собраны все скрипты
Используем в uses: 
1) [action для использования код, который мы загружаем](https://github.com/marketplace/actions/checkout)
2) [action для flutter ](https://github.com/marketplace/actions/flutter-action)

.github/workflows/example-ci.yml
```yaml
name: "Example-ci"
on: [push]
jobs:
    example-job:
        runs-on: ubuntu-latest 
        timeout-minutes: 10
        steps:
            - uses: actions/checkout@v4
            - uses: subosito/flutter-action@v2
              with:
                channel: 'stable'
            - run: flutter analyze .
```
CI для нашего прилоения
```yaml
name: 'Example ci/cd'  
on:   # e.g. on: [push, workflow_dispatch]  
  pull_request:  
  push:  
  workflow_dispatch:  
# on:  
#   push:  
#     branches:  
#       - main  
#       - 'releases/**'  
#     branches-ignore:    #       - 'releases/**-alpha'  
jobs:  
  analyze-flutter-project:  
    runs-on: ubuntu-latest  
    timeout-minutes: 10  
    steps:  
      - uses: actions/checkout@v4  
      - uses: subosito/flutter-action@v2  
        with:  
          channel: 'stable'  
      - run: flutter pub get  
      - run: flutter analyze .  
      - run: dart format --set-exit-if-changed .  
  
  run-tests:  
    runs-on: ubuntu-latest  
    timeout-minutes: 10  
    needs: [analyze-flutter-project]  
    steps:   
      - uses: actions/checkout@v4  
      - uses: subosito/flutter-action@v2  
        with:  
          channel: 'stable'  
      - run: flutter pub get  
      - run: flutter test  
  
  build-android:  
    needs: [run-tests, analyze-flutter-project]  # Завязываемся, на прошлые jobs
    runs-on: ubuntu-latest  
    timeout-minutes: 10  
    steps:   
      - uses: actions/checkout@v4  
      - uses: subosito/flutter-action@v2  
        with:  
          channel: 'stable'  
      - run: flutter pub get  
      - run: flutter build apk  
  
  build-ios:  
    needs: [run-tests, analyze-flutter-project]  
    runs-on: macos-latest  
    timeout-minutes: 10  
    steps:   
      - uses: actions/checkout@v4  
      - uses: subosito/flutter-action@v2  
        with:  
          channel: 'stable'  
      - run: flutter pub get  
      - run: flutter build ipa --no-codesign  
  
  build-android-flavors:  
    needs: [run-tests, analyze-flutter-project]  
    strategy:  
      matrix:  
        flavor: [production, beta]  
    runs-on: ubuntu-latest  
    timeout-minutes: 10  
    steps:   
      - uses: actions/checkout@v4  
      - uses: subosito/flutter-action@v2  
        with:  
          channel: 'stable'  
      - run: flutter pub get  
      - run: flutter build apk --flavor=${{ matrix.flavor }} --target=lib/app/ta
rgets/${{ matrix.flavor }}/main.dart  
  
  build-android-release:  
    needs: run-tests  
    runs-on: ubuntu-latest  
    timeout-minutes: 10  
    steps:   
      - uses: actions/checkout@v4  
      - uses: subosito/flutter-action@v2  
        with:  
          channel: 'stable'  
      - run: flutter build apk --flavor=production --target=lib/app/targets/production/main.dart  
      - name: "Upload apk artifacts"  
        uses: actions/upload-artifact@v1  
        with:  
          name: release-apk  
          path: build/app/outputs/flutter-apk/app-production-release.apk
```
В этом проекте используется flavor, в сомом коде это выглядит так:
![](https://i.imgur.com/4216wVa.png)
Также нужно добавит информацию о flavor в AndroidManifest и в launch.json
github context дает возможность обращаться к переменным сборки