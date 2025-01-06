mongoDb - это документоориаентированная база данных 
mogosh - запуск клиента  mongo
```bash
db.books.insertOne(
	{
		title : "TItls",
		author: "Author"
	}
)
db.books.insertMany([
	{
		title : "TItls",
		author: "Autho"
	}
]
)
```

Поиск
```bash
db.books.find() //вернет все
db.books.findOne({author : "Autho"})
```
Обновление
```bash
db.collenction.updateOne(<filter>, <update>, <options>)
db.collenction.updateMany(<filter>, <update>, <options>)
db.collenction.replaceOne(<filter>, <update>, <options>)
```
Их возврат:
```bash
{
	acknowledged: true,//получилось/нет
	...
}
```
Индекс
```bash
db.collection.creatIndex({author : 1})
```
![[Pasted image 20231114195633.png]]
1 - отвечает, что выбрали, то есть тут индекс по ключб short_url.

## MongoDb Compass 
визуализация

## Mongo in go
Модуль bson  - пакет для типов из MongoDb
bson.D - Slice
bson.M - Map
bson.A - Array

https://gitlab.com/getBolted/mongodb-workshop
## schema Validation
```
db.createCollection("students, {
	validator:: {
		bsonType : "object"
		title: "Student"
	}
}")
```

![[Pasted image 20231114195014.png]]
```
```
validationLevel:
strick - очень жестко, с типами, все четко дожно быть
moderate - жестко только с нивыми
levelOff - пофиг, просто рисуем схему 

## Репликация
Mongo DB позволяет из коробки делать репликацию, то есть быть более надежной
Это плюс по сравнению с Postgress

Но нам надо писать данные на три instance, это создает проблему. что мы можем запросить несуществующие данные, теряется консистентность.
![[Pasted image 20231114200856.png]]
Поэтому бахаем мастера
```docker
test: echo "try { rs.status() } catch (err) { rs.initiate({_id:'rs0',members:[{_id:0,host:'127.0.0.1:27017',priority:1},{_id:1,host:'127.0.0.1:27018',priority:0.5},{_id:2,host:'127.0.0.1:27019',priority:0.5}]}) }" | mongosh --port 27017 --quiet
```
![[Pasted image 20231114201008.png]]

Узнаем кто мастер:
```console
db.runCommand("ismaster") 
```

## Mongo in go
```go
client, err := mongo.Connect(ctx, options.Client().ApplyURI(a.cfg.Mongo.Uri))  
if err != nil {  
    return fmt.Errorf("new mongo client create error: %w", err)  
}  
  
err = client.Ping(ctx, readpref.Primary())  
if err != nil {  
    return fmt.Errorf("new mongo primary node connect error: %w", err)  
}  
  
a.mongoClient = client  
database := client.Database(a.cfg.Mongo.Database)
```
Миграции
![[Pasted image 20231114202144.png]]
Переход по состояниям,  
up - переход на след версию
down - откат к предыдущей
```go
if a.cfg.Migrations.Enabled {  
    migrationSvc := migration.NewMigrationsService(a.log, database)  
    err = migrationSvc.RunMigrations(a.cfg.Migrations.Path)  
    if err != nil {  
       return fmt.Errorf("run migrations failed")  
    }  
}
```

## Тестирование
mock 
//go:generate mockgen -source=stats.go -destination=mocks/stats_mock.go