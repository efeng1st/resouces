io包
==============
io包提供I/O元语的基本接口。

原语(primitive)在计算机语言中表示原子型语句， 即在执行过程中不可被打断的基本操作。可以理解为一段代码， 这段代码在执行过程中不能被打断。像原子一样，不可分割，不可打断的语句。

io包的主要任务就是包装例如os包中的那些原语的现有实现到共享的抽象功能的公开接口，以及其他一些相关原语。
因为这些接口和原语包装使用各种不同实现的低级操作, 除非另外告诉客户不应该假设它们并行执行是安全的。

## Seek常量
首先定义Seek相关的几个常量：
- SeekStart
- SeekCurrent
- SeekEnd

```go
// Seek whence values.
const (
	SeekStart   = 0 // seek relative to the origin of the file
	SeekCurrent = 1 // seek relative to the current offset
	SeekEnd     = 2 // seek relative to the end
)
```
## error对象

接下来定义了几个error对象:
- ErrShortWrite
- ErrShortBuffer
- EOF
- ErrUnexpectedEOF
- ErrNoProgress

```go
// ErrShortWrite means that a write accepted fewer bytes than requested
// but failed to return an explicit error.
var ErrShortWrite = errors.New("short write")

// ErrShortBuffer means that a read required a longer buffer than was provided.
var ErrShortBuffer = errors.New("short buffer")

// EOF is the error returned by Read when no more input is available.
// Functions should return EOF only to signal a graceful end of input.
// If the EOF occurs unexpectedly in a structured data stream,
// the appropriate error is either ErrUnexpectedEOF or some other error
// giving more detail.
var EOF = errors.New("EOF")

// ErrUnexpectedEOF means that EOF was encountered in the
// middle of reading a fixed-size block or data structure.
var ErrUnexpectedEOF = errors.New("unexpected EOF")

// ErrNoProgress is returned by some clients of an io.Reader when
// many calls to Read have failed to return any data or error,
// usually the sign of a broken io.Reader implementation.
var ErrNoProgress = errors.New("multiple Read calls return no data or error")
```

## Reader接口
Reader是包装基本Read方法的接口。
```go
// Implementations must not retain p.
type Reader interface {
	Read(p []byte) (n int, err error)
}
```
Read最多读取len(p)字节到p字节序列中。它返回读取的字节数(0 <= n <= len(p))和任意遇到的错误。 即便Read返回n < len(p), 在调用期间它仍然可以使用p的所有作为抓取空间。如果还有数据可用，但是不是len(p)字节，那么Read按惯例返回可用的信息，而不是等待更多数据。

当Read在成功读取n > 0字节后遇到错误或者文件结束条件, 它返回读取到的字节数。它可以在同一调用返回(非nil)错误, 或者在后续调用返回错误(n==0)。 

这种一般化例子的实例是Reader返回在输入流结束点非零值字节数, 可能返回err == EOF或err == nil。下一次Read应该返回0, EOF。

调用者总应在考虑error err之前，处理n>0字节返回。这样可以正确处理在读取一些字节以及允许的EOF行为后发生的I/O错误。

Read的实现在除了len(p) == 0之外，不鼓励返回零字节和nil错误。 调用者应该将返回零字节和nil表示没有发生什么； 特别的是它不表示EOF。
实现不能保留p。
