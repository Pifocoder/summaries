## Compiler
Задумавлся как просто быстрая компиляция.
В go не ту дебажной сбеорки и всего такого, потому что даже в рабочей сборке, когда падает код нужно понимать, где что сломалось. 
Пример инлайнинг F в такой программе:
```go
package main

import (
	"os"
)
func F(a int) int {
	return 2 / (a - 1);
}
func main() {
	os.Exit(F(len(os.Args)))
}
```
При панике корректно скажет, что упали на 8 строчке, хотя функция заинлайнена в main при компиляции.
### Escape Analyze
Понимает где размещать объект: на куче или не стеке?
Пример:
```go
package main
import "os"
func main() {
	p0 := make([]byte, 10) 
	p0[0]='f'
	
	var p1 [10] byte
	copy (p1[:], p0)
	
	os.stdout.Write(p1[:])
}
```
p0 выделится на стеке, а p1 на куче, потому что p1 escap-ится в стороннюю функцию Write.
чтобы лучше понять что, куда escape-ится, можно сбилдить с флагом:
```
go build -o /tmp/escape -gcflags='-m=1' .
```
он выведет, где что сохранится.

чтобы увидеть доказательство можно применить друго флаг, он также покажет инлайнинг:
```
go build -o /tmp/escape -gcflags='-m=2' .
```
### Bound Check Elimination
```go
d := make([]byte, 16)

//компилятор как бы здесь поставит if на проверку длины, 
//либо не поставит, но сможет как то для себя доказать, что длина больше 17
d[17] = 12
```
В bound check мы можем делать разные оптимизации, например
```go
func (littleEndian) Uint64(b []byte) uint64 {
	_ = b[7] // bounds check hint to compiler; see golang.org/issue/14808
	return uint64(b[0]) | uint64(b[1])<<8 | uint64(b[2])<<16 | uint64(b[3])<<24 |
		uint64(b[4])<<32 | uint64(b[5])<<40 | uint64(b[6])<<48 | uint64(b[7])<<56
}
```
компилятор не будет ставит if перед послдещими (после `_ = b[7]` )обращениями по индексу к b.
## Garbage collector
### Finilizer 
В runtime есть такая штука как Finilizer. Идея в том, чтобы что-то сделать, когда обект будет чиститься garbage collector-ом. 
Пример работа с файлами:
```go
// newFile is like NewFile, but if called from OpenFile or Pipe
// (as passed in the kind parameter) it tries to add the file to
// the runtime poller.
func newFile(fd int, name string, kind newFileKind) *File {
	f := &File{&file{
			pfd: poll.FD{
				Sysfd:         fd,
				IsStream:      true,
				ZeroReadIsEOF: true,
			},
			name:        name,
			stdoutOrErr: fd == 1 || fd == 2,
		}}
		
	// ...
	
	runtime.SetFinalizer(f.file, (*file).close)
	return f
}
```
Т е файл будет закрыт, даже если мы не вызовем file.Close(), но это произойдет только при чистке GC, что может произойти не скоро.
Пример 2, при вызове обычного Close, finilizer отменяется, потому что он тоже жрет ресурсы:
```go
func (file *file) close() error {
	
	//...
	
	// no need for a finalizer anymore
	runtime.SetFinalizer(file, nil)
	return err
}
```
### Параметры GC
Есть env переменная GOGC, в которой можно поставить размер life set-а, тогда программ будет побреблять не более 2 * GOGC памяти. 
Других ручек для настройки нет.
### sync.Pool
Ситуация: У нас есть сложный тяжелый объкт, который мы при каждом запросе к нашему API создаем, это грустно. Это тратит очень много памяти. Оптимизировать это сможет pool таких объектов, т е чтобы обекты переиспользовались, это делается с помощью sync.Pool, пример использования:
```go
var pool = sync.Pool{
	New: func() any {
		return &Decoder{}
	},
}
type Decoder struct { 
	buf[1<< 20]byte
}
func New() *Decoder {
	return pool.Get().(*Decoder)
}
func (c *Decoder) Close() {
	pool.Put(c)
}
```
## Scheduler
![[Pasted image 20250125204709.png]]
Горутина изначально попадает в какую то локальную очередь какого то шарда( количество шардов задается GOMAXPROC), но также может попасть и в глобальную.
При GOMAXPROC = 1 может исполняться не более одной горутины одновременно.

`runtime.LockOsThread` - кастыль, который блокирует текущий тред системы какой то горутины, на неё.

### pprof
`import _ "net/http/pprof"
```go
go func() {
	log.Println(http.ListenAndServe("localhost:6060", nil))
}()
```
На отдельном дебажном порте, сервара, на котором работает что-то в проде, запускаем такую штуку, и в веб версии сможем посмотреть что происходит с программой
Можно использовать в купе с тулой
![[Pasted image 20250125210122.png]]
В пакете в runtime ещё есть:
- stacktraces
- mem stats - стата использования памяти
- gc stats - - стата использования памяти
- runtime/debug
Дебагеры:
- delve - лучше понимает что такое горутины и всякие гошные штуки.
- gdb