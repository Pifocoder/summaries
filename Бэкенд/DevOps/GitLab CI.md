#### stages
Стадии задаются директивой [stages](https://docs.gitlab.com/ee/ci/yaml/#stage), если она не указана, то пайплайн будет содержать следующие стадии: build, test, deploy. Также пайплайн всегда содержит две служебные стадии .pre и .post, задачи в .pre выполняются перед всеми остальными, .post – после.  
  
Для более точного упорядочивания задач используют директиву [needs](https://docs.gitlab.com/ee/ci/yaml/#needs), которая явно указывает зависимые задачи

В stage дириктива image указывает в какой Docker-образе запускается задача.
В stage дириктива  script – shell-скрипт, выполняющийся в задаче.

### variables
Параметры пайплайна задаются при помощи переменных, директивой [variables](https://docs.gitlab.com/ee/ci/yaml/#variables) в пайплайне или в настройках проекта/группы.
```
stages:
  - build
  - publish
variables:
  NODEJS_IMAGE: registry.devops-teta.ru/materials/ci/images/nodejs:18.18.2-bookworm
  KANIKO_IMAGE: registry.devops-teta.ru/materials/ci/images/kaniko:1.9.1  
 
# @NoArtifactsToSource
Lint:
  stage: .pre
  needs: []
  image:
    name: $NODEJS_IMAGE
    entrypoint: [""]
  script:
    - echo 1
 
# @NoArtifactsToSource
Build Package:
  stage: build
  needs: []
  image:
    name: $NODEJS_IMAGE
    entrypoint: [""]
  script:
    - echo 2
 
# @NoArtifactsToSource
Publish Package:
  stage: publish
  needs:
    - job: Build Package
      artifacts: true
  image:
    name: $KANIKO_IMAGE
    entrypoint: [""]
  script:
    - echo 3
```

### cache
GitLab CI поддерживает кеширование файлов для ускорения сборок. В отличие от артефактов кеш хранится в раннере и необязателен для успешного запуска задачи.
Параметры кеширования задаются директивой [cache](https://docs.gitlab.com/ee/ci/yaml/#cache). Ключом кеширования является либо строка, либо хеш файлов. 
```
  cache:
    - key: npm-packages
      paths:
        - .cache
```
По умолчанию GitLab CI отделяет кеш для запущенных в защищенных ветках задач, опция unprotect: true отключает это поведение. Таким образом мы ускорим сборки в защищенных ветках, но пользователи смогут переопределить пайплайн в незащищенной ветке и переписать значение кеша.

Корректная фс:
![](https://i.imgur.com/sHruGB9.png)
