Конкурентность - нам нужен результат от параллельного запроса
параллельность - независимость
![[Pasted image 20231205182056.png]]
Горутина - это надстройка над тредом
![[Pasted image 20231205182947.png]]
```go 
func main() {
	go test()
	go run()
	time.Sleep(1)
}
func test() {
	fmt.PrintLn("test")
}
func run() {
	fmt.PrintLn("run")
}
```

GMP модель
G - го-рутина
M - машина 
P - процессор

Каналы - способ передачи данных из одной го-рутины в другую
Буферизированный канал - это конал, в котором могут лежать данные
![[Pasted image 20231205185454.png]]

Mutex -патерн обращения к shared memory(без гонки)
![[Pasted image 20231205190909.png]]
НО порой это очень замедляет, поэтому придумали RWmutex
Есть два типа Lock.
Lock - как у обычной
RLock - Lock на чтение
![[Pasted image 20231205191438.png]]
![[Pasted image 20231205191325.png]]
## Wait Group
Примитив синхронизации для выполнения многопоточного кода
По сути - счетчик
Add(delta) - добавить delta горутин
Done - делает Add(-1)
Wait - останавливает выполнение, пока счетчик не станет равен нулю
![[Pasted image 20231205191910.png]]

## SELECT
# Atomic
![[Pasted image 20231205192359.png]]
Atomic в go - метод синхронизации горутин


Конкурентный кэш
```go
type entry struct {
    res   result
    ready chan struct{} // closed when res is ready
}
func (memo *Memo) Get(key string) (value interface{}, err error) {
    memo.mu.Lock()
    e := memo.cache[key]
    if e == nil {
        // This is the first request for this key.
        // This goroutine becomes responsible for computing
        // the value and broadcasting the ready condition.
        e = &entry{ready: make(chan struct{})}
        memo.cache[key] = e
        memo.mu.Unlock()
        e.res.value, e.res.err = memo.f(key)
        close(e.ready) // broadcast ready condition
    } else {
        // This is a repeat request for this key.
        memo.mu.Unlock()
        <-e.ready // wait for ready condition
    }
    return e.res.value, e.res.err
}
```

Batcher - 2 cond var
