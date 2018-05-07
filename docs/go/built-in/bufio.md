Go语言的bufio
====================

bufio实现了缓冲I/O。它对io.Reader和io.Writer对象进行包装，创建出另外一个也实现了这些接口的对象(Reader或Writer), 但是它提供了缓冲I/O, 并提供了一些帮助处理文本I/O的方法。


## bufio.Reader
```go

const (
    defaultBufSize      = 4096
)

var (
    ErrInvalidUnreadByte    = errors.New("bufio: invalid use of UnreadByte")
    ErrInvalidUnreadRune    = errors.New("bufio: invalid use of UnreadRune")
    ErrBufferFull           = errors.New("bufio: buffer full")
    ErrNegativeCount        = errors.New("bufio: negative count")
)

type Reader struct {
    buf             []byte
    rd              io.Reader // 客户端提供的reader
    r, w            int       // buf的读写位置
    err             error
    lastByte        int
    lastRuneSize    int
}

const minReadBufferSize = 16
const maxConsecutiveEmptyReads = 100
```

bufio.Reader为io.Reader实现了缓冲能力。上面定义了三个常量和四个errors变量。

- defaultBufSize: 默认缓冲字节数。
- minReadBufferSize: 读取的最小缓冲字节数。
- maxConsecutiveEmptyReads: 


- ErrInvalidUnreadByte: 使用无效的UnreadByte错误。
- ErrInvalidUnreadRune: 使用无效的UnreadRune错误。
- ErrBufferFull: 缓冲区已满错误。
- ErrNegativeCount: 负的计数器错误。

Reader结构体包含一个字节分片、客户端提供的reader、以及buf字节分片的读写位置、错误、前一个字节、前一个Rune尺寸。

## bufio.NewReaderSize()函数
```go
func NewReaderSize(rd io.Reader, size int) *Reader {
	// Is it already a Reader?
	b, ok := rd.(*Reader)
	if ok && len(b.buf) >= size {
		return b
	}
	if size < minReadBufferSize {
		size = minReadBufferSize
	}
	r := new(Reader)
	r.reset(make([]byte, size), rd)
	return r
}
```

该函数返回一个缓冲区大小至少为给定size字节的bufio.Reader对象。如果传入的第一个参数本身就是一个buf.Reader, 那么只需要检查它的buf的尺寸是否比size大， 如果大的话就满足直接返回这个对象就好了。否则我们使用new关键词创建一个buf.Reader，并重置其缓冲区大小满足size。

## bufio.NewReader()函数
该函数返回默认缓冲区大小尺寸的bufio.Reader对象。代码如下:
```go
func NewReader(rd io.Reader) *Reader {
	return NewReaderSize(rd, defaultBufSize)
}
```

## bufio.Reader的相关方法

### 获取缓冲区大小和重置bufio.Reader的方法
- Size(): 返回bufio.Reader的缓冲区尺寸值。
- Reset(): 丢弃掉所有缓冲数据，重置所有状态，并将缓冲的reader切换为传入的io.Reader对象r。
- reset(): Reset的底层实现。

```go
func (r *Reader) Size() int { return len(r.buf) }

func (b *Reader) Reset(r io.Reader) { b.reset(b.buf, r) }

func (b *Reader) reset(buf []byte, r io.Reader) {
    *b = Reader{
        buf: buf,
        rd: r,
        lastByte: -1,
        lastRuneSize: -1,
    }
}
```

> 注意bufio.Reader的方法接收者都使用的是指针值接收者。可以直接修改自身属性，甚至重置自己。

### fill()方法
该方法读取新的一块数据填充到缓冲区中。详细代码如下:

```go
func (b *Reader) fill() {
    if b.r > 0 { // 跳过缓冲区原有数据进行填充
        copy(b.buf, b.buf[b.r:b.w])
        b.w -= b.r
        b.r = 0
    }

    if b.w >= len(b.buf) { // 缓冲区还是满的，无法填充新内容进去
        panic("bufio: tried to fill full buffer")
    }

    // 读取新数据: 这里根据maxConsecutiveEmptyReads常量来会限制读取的次数。
    for i := maxConsecutiveEmptyReads; i > 0; i-- {
        n, err := b.rd.Read(b.buf[b.w:])
        if n < 0 {
            panic(errNegativeRead)
        }

        b.w += n
        if err != nil {
            b.err = err
            return
        }

        if n > 0 {
            return
        }
    }

    b.err = io.ErrNoProgress
}
```

### Reader.Peek()方法

