`database/sql` - абстракция для работы с SQL базами данных, которая не поддерживает продвинутые функция работы с бд, поэтому используют драйверы, которые уже используют `database/sql`.
## Подключение к бд
Работа с подключениями, как с обстракцией с помощью `database/sql`

Для подключения к бд используется `sql.Open` (`db, err := sql.Open("pgx", "postgres://pgx_md5:secret@localhost:5432/pgx_test")`),  который создает connection pool к этой бд, но физически не полключается к бд.
Там, где мы делаем `sql.Open` нужно ещё и следущим образом импортировать stdlib драйвер (либо другой драйвер), чтобы он инициализировался.
```go
import (
    "database/sql"
    "log"
    _ "github.com/jackc/pgx/v5/stdlib"
)

func SQLOpen() {
    db, err := sql.Open("pgx", "postgres://pgx_md5:secret@localhost:5432/pgx_test")
    if err != nil {
        log.Fatal(err)
    }
    defer db.Close()
}
```
Как теперь проверить подключились ли мы, есть ли в  connection pool рбочее подключение, если подключение до этого не физическое?
```go
func IsItAliveQuestionMark(ctx context.Context) {
    db, err := sql.Open("pgx", "postgres://pgx_md5:secret@localhost:5432/pgx_test")
    if err != nil {
        log.Fatal(err)
    }
    defer db.Close()
    if err := db.PingContext(ctx); err != nil {
        log.Fatal(err)
    }
}
```

Работа с подключениями, как с подклбчение на прямую - физическое подключение
```go
import (
    "context"
    "log"
    "github.com/jackc/pgx/v5"
)
func PGXOpen() {
    ctx := context.Background()
    conn, err := pgx.Connect(ctx, "postgres://pgx_md5:secret@localhost:5432/pgx_test")
    if err != nil {
        log.Fatal(err)
    }
    defer conn.Close(ctx)
}
```

Если мы хотим настроить какие то параметры для определеного Connection, нам надо получить из Connection pool, какой то connection. (Мы не можем настроить параметры сразу для всего connection pool). Это делается с помощью `db.Conn`
```go
func Conn(ctx context.Context, db *sql.DB) {
    c, err := db.Conn(ctx)
    if err != nil {
        log.Fatal(err)
    }
    defer c.Close()
    _ = c.PingContext(ctx)
}
```
### Настройка *sql.DB
- func (db *DB) SetConnMaxIdleTime(d time.Duration)
- func (db *DB) SetConnMaxLifetime(d time.Duration)
- func (db *DB) SetMaxIdleConns(n int)
- func (db *DB) SetMaxOpenConns(n int)
### Статистика *sql.DB
- func (db *DB) Stats() DBStats
```go
type DBStats struct {
    MaxOpenConnections int // Maximum number of open connections to the database; added in Go 1.11
    // Pool Status
    OpenConnections int // The number of established connections both in use and idle.
    InUse           int // The number of connections currently in use; added in Go 1.11
    Idle            int // The number of idle connections; added in Go 1.11
    // Counters
    WaitCount         int64         // The total number of connections waited for; added in Go 1.11
    WaitDuration      time.Duration // The total time blocked waiting for a new connection; added in Go 1.11
    MaxIdleClosed     int64         // The total number of connections closed due to SetMaxIdleConns; added in Go 1.11
    MaxIdleTimeClosed int64         // The total number of connections closed due to SetConnMaxIdleTime; added in Go 1.15
    MaxLifetimeClosed int64         // The total number of connections closed due to SetConnMaxLifetime; added in Go 1.11
}
```
### Запрос с получением результатов
##### Запрос с множеством результатов
```go
func Query(ctx context.Context, db *sql.DB) {
    rows, err := db.QueryContext(ctx, "SELECT id, name FROM users")
    if err != nil {
        log.Fatal(err)
    }
    defer rows.Close()
    for rows.Next() {
        var id int
        var name string
        if err := rows.Scan(&id, &name); err != nil {
            log.Fatal(err)
        }
        log.Println(id, name)
    }
    if err = rows.Err(); err != nil {
        log.Fatal(err)
    }
}
```
>[!info] 
>`rows.Next()` работает интересныйм образом, он может как просто идти по данным, которыйе лежат в памяти приложения, которые мы одним разом получили из бд, так и ходить в бд, подгружая в приложение новые данные, новые результаты запроса. Так как `rows.Next()` возвращает false, когда в при запросе к новым данным произошла ошибка, нам надо это проверять после цикла for, потому что for мог закончиться не только, если мы прочитали все данные, но и если произошла ошибка при чтении результата из бд.

`rows.Close()` нужен, чтобы вернуть Connection, это условие `database/sql`.
## Запросы
##### Запрос с одним результатом
Если у нас всего один Row в результате запроса 1 или 0 Row, то мы можем использовать `QueryRowContext`:
```go
func QueryRow(ctx context.Context, db *sql.DB) {
    var id int
    var name string
    err := db.QueryRowContext(ctx, "SELECT id, name FROM users WHERE id = $1", 1).Scan(&id, &name)
    if err != nil {
        if errors.Is(err, sql.ErrNoRows) {
            log.Println("nothing found")
            return
        }
        log.Fatal(err)
    }
    log.Println(name)
}
```
Если запрос возвращает больше одной строки, метод вызовет ошибку `sql.ErrNoRows` или `sql.ErrMultipleRows`, в зависимости от ситуации.
##### Запрос без результатов 
`db.ExecContext`
```go
func Exec(ctx context.Context, db *sql.DB) {
    res, err := db.ExecContext(
        ctx,
        "UPDATE users SET name = $1 WHERE id = $2",
        "William Mandella", 1,
    )
    if err != nil {
        log.Fatal(err)
    }
    lastID, _ := res.LastInsertId()
    rowsAffected, _ := res.RowsAffected()
    log.Println(lastID, rowsAffected)
}
```
Возвращаемый тип - driver specific:
```go
type Result interface {
    LastInsertId() (int64, error)
    RowsAffected() (int64, error)
}
```
Запрос и аргументы передаются отдельно, чтобы предотвратить SQL инъекции.

Проблема - очень легко перепутать порядок элементов, потому что, что куда вставится зависит от позиции.
Решение - именованные аргументы.

##### Именованные аргументы.
реализуются на уровне драйвера
```go
type NamedArg struct {
    Name  string
    Value interface{}
}
```
```go
func Insert(ctx context.Context, db *sql.DB) {
    _, err := db.ExecContext(
        ctx,
        "INSERT INTO users(name) VALUES(@name)",
        sql.Named("name", "Amos Burton"),
    )
    if err != nil {
        log.Fatal(err)
    }
}
```
#### Nulls в аргументах
Особенно интересно это в Select.
```go
func Results(ctx context.Context, db *sql.DB) {
    rows, err := db.QueryContext(ctx, "SELECT name FROM users WHERE id = $1", 1)
    if err != nil {
        log.Fatal(err)
    }
    defer rows.Close()
    for rows.Next() {
        var s sql.NullString
        if err := rows.Scan(&s); err != nil {
            log.Fatal(err)
        }
        if s.Valid {
            //
        } else {
            //
        }
    }
}
```
```go
func Insert(ctx context.Context, db *sql.DB, name interface{}) {
    _, err := db.ExecContext(
        ctx,
        "INSERT INTO users(name) VALUES(@name)",
        sql.Named("name", name),
    )
    if err != nil {
        log.Fatal(err)
    }
}
```
```go
func DoStuff(ctx context.Context, db *sql.DB) {
    // Nulls
    Insert(ctx, db, nil)
    Insert(ctx, db, sql.NullString{})
    // Values
    Insert(ctx, db, "The Shrike")
    Insert(ctx, db, sql.NullString{String: "The Shrike", Valid: true})
}
```
## Транзакции
`db.BeginTx(ctx, nil)` вместо nil можно передать нужный уровень изоляции, типа Read Commited. Rollback после Commit не делает ничего, а до откатывает. хороший пример:
```go
func Begin(ctx context.Context, db *sql.DB) {
    tx, err := db.BeginTx(ctx, nil)
    if err != nil {
        log.Fatal(err)
    }
    defer tx.Rollback()
    _, err = tx.ExecContext(ctx, `UPDATE users SET name = "Tyador Borlú" WHERE id = 1`)
    if err != nil {
        log.Fatal(err)
    }
    if err = tx.Commit(); err != nil {
        log.Fatal(err)
    }
}
```
## Prepared Statements
Подготовка запросов. Парсинг самого запроса. Используется когда запросы супер большие.
Плюсы:
- решают проблему sql-injection
- производительность (парсинг заранее)
Минусы:
- удобство
- производительность (Ухудшается оптимизация, которую делает сама бд)
- несовместимы с некоторыми режимами работы пулеров коннектов (пуллер - это штука на уровне системы, которая работает с connections к бд, проблема в том, что мы можем сделать Prepared Statements на одном connection, а системный puller вернем нам другой connection)
```go
func Prepare(ctx context.Context, db *sql.DB) {
    stmt, err := db.PrepareContext(ctx, "SELECT name FROM users WHERE id = $1")
    if err != nil {
        log.Fatal(err)
    }
    defer stmt.Close()
    for i := 1; ; i++ {
        var name string
        if err = stmt.QueryRowContext(ctx, i).Scan(&name); err != nil {
            log.Fatal(err)
        }
        log.Println(name)
    }
}
```
Внутри бд парсит запроси ивозращает нам какой то токен этого запроса. В данном случае `stmt.QueryRowContext` смотрить делал ли он `PrepareContext`, на connection, на которой сейчас делает запрос и если нет, то делает на нем `PrepareContext`.
# Проблемы
#### Context - вечные запросы
```go
func NoContext(ctx context.Context, db *sql.DB) {
    // У Conn() нет версии без контекста
    c, err := db.Conn(ctx)
    if err != nil {
        log.Fatal(err)
    }
    defer c.Close()
    // Потенциально вечный Ping
    _ = db.Ping()
}
```
#### Неосвобождение ресурсов
```go
func RowsExhaust(ctx context.Context, db *sql.DB) {
    rows, err := db.QueryContext(ctx, "SELECT id, name FROM users")
    if err != nil {
        log.Fatal(err)
    }
    if rows.Next() {
        var id int
        var name string
        if err := rows.Scan(&id, &name); err != nil {
            log.Fatal(err)
        }
        log.Println(id, name)
    }
}
```
rows - будет занимать connection к бд.
Лечение
```go
defer rows.Close()
```
Аналогично:
```go
func ConnExhaust(ctx context.Context, db *sql.DB) {
    c, err := db.Conn(ctx)
    if err != nil {
        log.Fatal(err)
    }
    _ = c.PingContext(ctx)
}
```
Лечение
```go
defer c.Close()
```
Аналогично:
```go
func TxExhaust(ctx context.Context, db *sql.DB) {
    tx, err := db.BeginTx(ctx, nil)
    if err != nil {
        log.Println(err)
        return
    }
    _, err = tx.ExecContext(ctx, `UPDATE users SET name = "Surl/Tesh-echer" WHERE id = 1`)
    if err != nil {
        log.Println(err)
        return
    }
    if err = tx.Commit(); err != nil {
        log.Println(err)
    }
}
```
Лечение
```go
defer tx.Rollback()
```
#### Deadlock в транзакции:
```go
func TxDeadlock(ctx context.Context, db *sql.DB) {
    tx, err := db.BeginTx(ctx, nil)
    if err != nil {
        log.Fatal(err)
    }
    defer tx.Rollback()
    _, _ = tx.QueryContext(ctx, "SELECT id, name FROM users")
    _, _ = tx.QueryContext(ctx, "SELECT id, name FROM users")
}
```
Первый Select выполнится, но из за незакрытых rows этого Select, второй запрос в этой транзакции зависнет
#### SELECT FOR UPDATE
Нужен чтобы не было таких случаев:
Обе транзакции прочитали, что у человека баланс 100, и решили, что каждая может сделать перевод на 70 рублей с этого счета, после этого, счет клиента станет отрицательным.
## Удобства и расширения
`github.com/jmoiron/sqlx` - расширение `database/sql`, у которого есть все функции `database/sql`, но и именованные поля для всех бд драйверов, а также StructScan 

sqlx - именованые аргументы + защита от SQL-injection: `NamedExecContext`
```go
func Insert(ctx context.Context, db *sqlx.DB) {
    _, err := db.NamedExecContext(
        ctx,
        "INSERT INTO users(name) VALUES(:name)",
        map[string]interface{}{
            "name": "Jukka Sarasti",
        },
    )
    if err != nil {
        log.Fatal(err)
    }
}
```
#### golang.yandex/hasql
- удобная работа с многохостовыми кластерами
- поддерживает `database/sql` и `sqlx`
hasql - подключение и использование
```go
func Open() {
    dbFoo, _ := sql.Open("pgx", "host=foo")
    dbBar, _ := sql.Open("pgx", "host=bar")
    cluster, err := hasql.NewCluster(
        []hasql.Node{hasql.NewNode("foo", dbFoo), hasql.NewNode("bar", dbBar)},
        checkers.PostgreSQL,
    )
    if err != nil {
        log.Fatal(err)
    }
    node := cluster.Primary()
    if err == nil {
        log.Fatal(err)
    }
    log.Println("Node address", node.Addr())
    ctx, cancel := context.WithTimeout(context.Background(), time.Second)
    defer cancel()
    if err = node.DB().PingContext(ctx); err != nil {
        log.Fatal(err)
    }
}
```
## Другие драйверы
#### go-sqlmock
`github.com/DATA-DOG/go-sqlmock`
```go
func TestSelect(t *testing.T) {
    db, mock, err := sqlmock.New()
    if err != nil {
        log.Fatal(err)
    }
    defer db.Close()
    mock.ExpectBegin()
    mock.ExpectExec("SELECT name FROM users WHERE id = ?").
        WithArgs(1).
        WillReturnError(sql.ErrNoRows)
    mock.ExpectCommit()
    tx, err := db.Begin()
    require.NoError(t, err)
    _, err = db.Exec("SELECT name FROM users WHERE id = ?", 1)
    require.NotNil(t, err)
    require.Equal(t, err, sql.ErrNoRows)
    require.NoError(t, tx.Commit())
    require.NoError(t, mock.ExpectationsWereMet())
}
```
#### Clickhouse
 `github.com/ClickHouse/clickhouse-go`
 [- ClickHouse](https://github.com/ClickHouse/clickhouse-go)
 БД с большим количеством столбцов.
```go
func Example(ctx context.Context) {
    db, _ := sql.Open("clickhouse", "tcp://127.0.0.1:9000?debug=true")
    defer db.Close()
    // Начало батча
    tx, _ := db.BeginTx(ctx, nil)
    defer tx.Rollback()
    // Описание батча
    stmt, _ := tx.PrepareContext(ctx, "INSERT INTO example (id) VALUES (?)")
    defer stmt.Close()
    // Добавление данных
    for i := 0; i < 100; i++ {
        _, _ = stmt.ExecContext(ctx, i)
    }
    // Отправка батча в ClickHouse
    _ = tx.Commit()
}
```
Clickhouse на самом деле не поддерживает транзакции, а на самом деле это делает batch и отправляется в бд, потому что Clickhouse любит большие запросы.

Другие дрйверы:
[- PostgreSQL](https://github.com/jackc/pgx)
[- MySQL](https://github.com/go-sql-driver/mysql)
#### Redis
Не-SQL драйвер
[- Redis](https://github.com/go-redis/redis)
```go
func Example(ctx context.Context) {
    rdb := redis.NewUniversalClient(&redis.UniversalOptions{
        MasterName: "master",
        Addrs:      []string{":26379"},
    })
    defer rdb.Close()
    if err := rdb.Ping(ctx); err != nil {
        log.Fatal(err)
    }
    if err := rdb.Set(ctx, "key", "value", time.Hour).Err(); err != nil {
        log.Fatal(err)
    }
    value, err := rdb.Get(ctx, "key").Result()
    if err != nil {
        log.Fatal(err)
    }
    log.Println(value)
}
```
[- MongoDB](https://github.com/mongodb/mongo-go-driver)


