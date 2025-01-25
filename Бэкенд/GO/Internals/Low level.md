## Unsafe
`unsafe` выглядит как обычный пакет, но реализован в компиляторе
Вычислится на этапе компиляции:
```go
unsafe.Sizeof(float64(0))
```
#### unsafe Pointer
unsafe Pointer это указатель на какой-то тип (_настоящий_ `void*`), поддерживает два преобразования:
```
*T -> unsafe.Pointer
unsafe.Pointer -> *T
```
Пример использования:
```go
func Float64bits(f float64) uint64 {
    return *(*uint64)(unsafe.Pointer(&f))
}
```
Арифметика указателей
Хотим сделать указатель на поле структуры:
```go
var x struct {
	a bool
	b int16
	c []int
}

// equivalent to pb := &x.b
pb := (*int16)(unsafe.Pointer(
	uintptr(unsafe.Pointer(&x)) + unsafe.Offsetof(x.b)))
*pb = 42
```
>[!important]
>Из за сборщика мусора есть ограничения на работу с указателями. Одни из них: все операции должны проиходить в одном выражении.

String builder этим пользуется:
```go
// A Builder is used to efficiently build a string using Write methods.
// It minimizes memory copying. The zero value is ready to use.
// Do not copy a non-zero Builder.
type Builder struct {
    buf  []byte
}

// String returns the accumulated string.
func (b *Builder) String() string {
    return *(*string)(unsafe.Pointer(&b.buf))
}

// *-----*-----*-----*
// * ptr * len * cap * // []byte
// *-----*-----*-----*
//
// *-----*-----*
// * ptr * len *       // string
// *-----*-----*
```
## CGO
расширение в go
Задача позвать сишный код из go-шной программы:
```cpp
#include <bzlib.h>

int bz2compress(bz_stream *s, int action,
                char *in, unsigned *inlen, char *out, unsigned *outlen) {
  s->next_in = in;
  s->avail_in = *inlen;
  s->next_out = out;
  s->avail_out = *outlen;
  int r = BZ2_bzCompress(s, action);
  *inlen -= s->avail_in;
  *outlen -= s->avail_out;
  s->next_in = s->next_out = NULL;
  return r;
}
```
```go
package bzip

/*
#cgo CFLAGS: -I/usr/include
#cgo LDFLAGS: -L/usr/lib -lbz2
#include <bzlib.h>
#include <stdlib.h>
bz_stream* bz2alloc() { return calloc(1, sizeof(bz_stream)); }
int bz2compress(bz_stream *s, int action,
                char *in, unsigned *inlen, char *out, unsigned *outlen);
void bz2free(bz_stream* s) { free(s); }
*/
import "C"
```
```go
type writer struct {
    w      io.Writer // underlying output stream
    stream *C.bz_stream
    outbuf [64 * 1024]byte
}
func NewWriter(out io.Writer) io.WriteCloser {
    const blockSize = 9
    const verbosity = 0
    const workFactor = 30
    w := &writer{w: out, stream: C.bz2alloc()}
    C.BZ2_bzCompressInit(w.stream, blockSize, verbosity, workFactor)
    return w
}
func (w *writer) Write(data []byte) (int, error) {
    if w.stream == nil {
        panic("closed")
    }
    var total int // uncompressed bytes written
    for len(data) > 0 {
        inlen, outlen := C.uint(len(data)), C.uint(cap(w.outbuf))
        C.bz2compress(w.stream, C.BZ_RUN,
            (*C.char)(unsafe.Pointer(&data[0])), &inlen,
            (*C.char)(unsafe.Pointer(&w.outbuf)), &outlen)
        total += int(inlen)
        data = data[inlen:]
        if _, err := w.w.Write(w.outbuf[:outlen]); err != nil {
            return total, err
        }
    }
    return total, nil
}
```
комментарий в go - это как заголовочный файл для соответствующей функции в Си.

Как в этом случае работает память? 
У Go и у Си отдельные кучи
При использованиии сишного кода возникают проблемы с GC, например, если мы в go храним указатель на аодрес сишой кучи, то GC просто не будет её чистить, так как это не его память. А вот если наоборот, то это беда. Тут жесткие правила после сишного вызова, си не может сохранять у себя указатели на гошную память.
## Syscall
- syscall - замороженный пакет с системными вызовами
- x/sys/unix содержит актуальные реализации системных вызовов