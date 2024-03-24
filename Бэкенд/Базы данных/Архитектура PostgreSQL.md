## Книжки
* Егор Рогов "PostgreSQL15"
* Новиков Горшкова Графеев  "Основы технологий баз данных"

Когда мы создаем Базу данных в postgreSQL, создается три базы даннх:
* postgres
* template0
* template1

## Организация хранения данных
Логичесвка схема:
* Пространство имен
* Схемы создаваемае по умолчанию: public, pg_catalog, information_schema, pg_toast, pg_temp
* Все обекты принадлежат какой-то схеме

Физическая схема:
* Каталог
* Иабличное пространство pg_default, pg_global

![](https://i.imgur.com/rLeeuCx.png)


## Каталог PGDATA
```SQL
SHOW data_history;
```


# Команды
## Запуск PostgreSQL
- `sudo -i -u postgres`.
- `psql`.
- `sudo -u postgres psql`.
- psql -U postgres -W  -h localhost


**Работа с базами данных**
**Список баз данных**
- `\l`.

**Текущая база данных**
- `SELECT current_database();`.

**Подключение к другой базе данных**
- `\c database_name`.

**Создание базы данных**
- `CREATE DATABASE database`.

## Схемы
**Список схем**
- `\dn` - возвращает схемы для каждой таблички
- `SELECT schema_name FROM information_schema.schemata;`
- `SELECT nspname FROM pg_catalog.pg_namespace;`

**Список таблиц в схеме**
- `\dt (схема public)`.
- `\dt schema_name`.

**Поиск таблиц**
- `SHOW search_path;`.
- `SET search_path TO schema_name,public;`.
Влияет на конкретную локальную базу данных

**Описание таблицы**
- `\d table_name`.
- `\d+ table_name`.

## Работа с данными
**Поиск PGDATA**
- `SHOW data_directory;`.

**Поиск расположения таблицы на диске**

- `SELECT pg_relation_filepath('t');`.
- `SELECT oid FROM pg_database WHERE datname = 'postgres';`.
- `SELECT relfilenode FROM pg_class WHERE relname = 't';`.

## Работа с табличными пространствами - файловая структура (каталог базы данных)
**Создание табличного пространства**
- `mkdir /data/dbs`.
- `chown postgres:postgres /data/dbs`.
- `CREATE TABLESPACE tbs_name LOCATION '/data/dbs';`.

**Использование табличного прстранства**
`CREATE DATABASE dbs TABLESPACE tbs_name` - создание базы внутри табличного пространства
`ALTER TABLE table_name SET TABLESPACE tbs_name;` - изменение названия

**Просмотр существующих табличных пространств**
- `\db`.
- `SELECT * FROM pg_tablespace;`

**Табличное пространство по умолчанию**
- `SET default_tablespace = tbs_name;`

Одна таблица может лежать только в одном табличном пространстве.

## Каталог
`pg_class` - информация о таблицах, индексах, последовательностях, т е отношения
`pg_tables` - представление на основе pg_class, инфа только о таблицах

# Структура хранения данных
Структура таблицы
* Файлы размером до 1ГБ
* При увеличении размера файла до 1ГБ создается новый файл и запись ведется в него

Страница ввода-вывода:
* минимальная 8кб

# Поцессы postgreSQL
Когда клиент подключается, он подключается к postmaster. 
Процесс - это некая изоляция какой то работы

postmaster (postgres) – основной процесс (управляет остальными процесами)
startup – восстановление после сбоя
autovacuum – освобождение места в таблицах и инедксах
wal writer – записывает на диск журналы предзаписи
checkpointer – выполняет контрольную точку
writer – записывает грязные страницы на диск

![](https://i.imgur.com/hMMCMX4.png)


# Журнал предзаписи
