Go语言标准库 - log
==========================
log包实现了简单的日志记录包。 它定义了一种Logger类型，这个Logger类型带有一些格式化输出的方法, 包含Print系方法、Fatal系方法和Panic系方法。每个系列还分别包含ln, f方法。 例如: Print、Printf、Println。

Fatal系方法本质上就是Print系对应方法执行完，调用os.Exit(1). 而Panic系方法本质就是Print系对应方法执行完，调用panic()函数。
日志按行输出，每行可以记录日期、时间、代码文件和代码行，以及日志信息。

## log标志
log中定义了一些标志，用于控制Logger产生的每条日志记录要展示的字段信息。这些标志可以通过位或的方式组合起来控制日志输出内容。这些标志仅仅控制显示的字段，而不控制它们显示的顺序，也不格式化它们展示的格式。字段显示格式是内部固定好的，标志只用来控制是否现实而已。
```
const (
	Ldate         = 1 << iota     // the date in the local time zone: 2009/01/23
	Ltime                         // the time in the local time zone: 01:23:23
	Lmicroseconds                 // microsecond resolution: 01:23:23.123123.  assumes Ltime.
	Llongfile                     // full file name and line number: /a/b/c/d.go:23
	Lshortfile                    // final file name element and line number: d.go:23. overrides Llongfile
	LUTC                          // if Ldate or Ltime is set, use UTC rather than the local time zone
	LstdFlags     = Ldate | Ltime // initial values for the standard logger
)
```

上面定义了一些标志常量，需要注意的是我们这里使用了特殊的语法，使用了关键词iota。

### iota关键词
iota关键词在声明一块常量的时候具有特殊目的。
- 它指示编译器为每个常量复制这个表达式，直到常量定义块末尾或遇到赋值语句为止。
- 它会自动将值在前面的常量后iota值基础上递增1。初始值为0。

那么上面的常量声明块具体分析就如下所示:
```go
const (
    Ldate   = 1 << iota         // 1 << 0 = 000000001 = 1
    Ltime                       // 1 << 1 = 000000010 = 2
    Lmicroseconds               // 1 << 2 = 000000100 = 4
    Llongfile                   // 1 << 3 = 000001000 = 8
    Lshortfile                  // 1 << 4 = 000010000 = 16
    ...
)
```
上面就是常量声明背后发生的具体事情。<<操作符是向左位移操作符。上面每个常量都是将1左移一定位数。 这样每个常量都会有一个唯一的位与之对应，这点对于标志常量的使用结合就非常完美了。

LstdFlags就展示了给每个常量一个唯一位的背后目的。 LstdFlags = Ldate(1) | Ltime(2) = 000000011 = 3.

## Logger类型及创建方法

```go
type Logger struct {
    mu sync.Mutex // 确保原子写；保护下面的字段
    prefix string // 每行日志之前的前缀
    flag int      // 属性
    out io.Writer // 输出目标
    buf []byte    // 用于累计要写入的文本
}

func New(out io.Writer, prefix string, flag int) *Logger {
    return &Logger{out: out, prefix: prefix, flag: flag}
}
```

Logger结构体包含5个字段:
- mu: 用于确保原子写的，保护下面其他字段写安全的。因为Logger支持多goroutine同时用同一个Logger实例进行写操作。并且能保证它们之间互不干扰。 本质上就是使用互斥锁来实现并发写的。
- prefix: 用于打印日志记录的前缀字符串。默认为空，一般提供一个全大写的字符串，用于日志信息分组标志。
- flag: 用于保存控制日志记录显示字段信息的值。是一个整数类型，通过标志位的位与操作可得到这个值。
- out: 日志输出使用的Writer。
- buf: 用于累计日志要写入的文本内容。

我们可以通过New函数来创建和初始化一个Logger类型的变量。这个函数我们可以提供自定义io.Writer, prefix, flag等参数。

## Logger属性设置和属性获取方法
Logger提供了对自身属性设置和获取的一些方法:

- SetOutput(w io.Writer): 设置日志输出Writer。
- SetPrefix(prefix string): 设置日志信息前缀。
- Flags(): 获取Logger的标志值。
- SetFlags(flag int): 设置标志。
- Prefix(): 获取前缀字符串。
- SetPrefix(prefix string): 设置前缀字符串。

```go
func (l *Logger) SetOutput(w io.Writer) {
    l.mu.Lock()
    defer l.mu.Unlock()
    l.out = w
}
```


## 辅助函数和其他
log包内定义了一个辅助函数itoa, 用于将整数转换为指定宽度的ASCII字节序列。 它还定义了一个标准的日志实例std, 不使用前缀，将日志信息输出到stderr, 默认输出日期和时间。

formatHeader函数，用于根据设置的标志来格式化输出日志头信息。

另外此包实现了一个基本的标准Logger std, Logger类型所具有的方法，都为std提供了方法。

### itoa函数
itoa函数将小的整数转换为固定宽度的十进制ASCII字节序列。如果给定负宽度，那么字节序列前面无需用零填充。

```go

func itoa(buf *[]byte, i int, wid int) {
    // 以逆序收集十进制
    var b [20]byte
    bp := len(b) - 1

    for i >= 10 || wid > 1 {
        wid--                               // wid递减，如果wid比实际整数位数要大，转换为宽度wid字节序列，在高位需要补零, 一旦wid为0，则无需补零了。
        q := i / 10                         // 求商数
        b[bp] = byte('0' + i - q * 10)      // 余数转字节类型
        bp--                                // 字节序列前移一位
        i = q                               // 用新的商作为下一轮循环的整数值
    }

    // i < 10                               // 余数或整数为小于10的，直接转换为字节类型
    b[bp] = byte('0' + i)
    *buf = append(*buf, b[bp:]...)          // 将整数字节序列附加到buf字节序列之后
}
```

整数由0-9组成，0-9转换为字节类型可以通过byte('0' + 0) 到 byte('0' + 9)的形式转换。
因为我们转换过程中，使用字节分片来存储ASCII, 需要保持整数的原始顺序，因此，我们需要将整数高位存储到字节分片的低索引处，低位存储到字节序列的高值索引处， 因此需要字节序列倒叙存储整数每一位的值。

例如我们将65535转换为8字节的字节序列， 我们按整数位数从低到高的过程处理如下:

- 当前整数65535, 最低位 = 65535(当前整数) - 65535/10(商) * 10 = 5(余数), 余数就是当前最低位的值，将其转换为字节放入byte字节序列中, 然后下一步将本步骤中的商值作为下一轮的当前整数。同时将wid减少1。
- 当前整数6553, 按照上面同样的方式获取商(655) 和 余数3, 然后将余数放入字节序列, wid减1。
- 重复上面的操作，直到当前整数值小于10或者wid=1的时候。


举例: 
- eg: itoa(buf, 65535, 5) // buf = [54 53 53 51 53], wid <= 5 buf的值都是左侧的形式。
- eg: itoa(buf, 65535, 8) // buf = [48 48 48 54 53 53 51 53], 整数转换为8个字节的字节序列，因为65535只有5字节，因此需要补充3字节高位0

如果我们不希望左侧填充零，那么我们可以传入wid = -1
如果我们明确知道整数位数以及要转换的字节位数比它大或相等，那么我们可以传入这个要转换的字节位数给wid.
其他情况，如果整数位数大于要转换的位数，则以整数位数为准。

### formatHeader函数
formatHeader函数是通过设置的标志来格式化日志条目的头信息。

- l.prefix: 如果提供有前缀，则将前缀添加到日志条目的最前面。
- data/time: 如果提供了相应的日期和时间标志，例如Ldate, Ltime, Lmicroseconds之类的标志，那么就将日期和时间写入buf字节序列中。
- file/line: 如果有提供Llongfile, Lshortfile标志，则会将源码文件名以及所在的代码行写入到日志记录的信息中。
```
var std = New(os.Stderr, "", LstdFlags)
func (l *Logger) formatHeader(buf *[]byte, t time.Time, file string, line int) {
    *buf = append(*buf, l.prefix...)          // 日志条目前缀信息
    if l.flag&(Ldate|Ltime|Lmicroseconds) != 0 { // 有日期和/或时间标志
        if l.flag&LUTC != 0 { // 如果使用LUTC标志，则使用UTC时间格式，而不用本地时间格式
            t = t.UTC()
        }

        if l.flag&Ldate != 0 {    // 如果设置了Ldate标志，则以YYYY/MM/DD格式将日期附加到缓冲字节序列之后 注意该字段末尾留一个空格
            year, month, day := t.Date() // t.Date()获取年月日值，都是整数值
            itoa(buf, year, 4)
            *buf = append(*buf, '/')
            itoa(buf, month, 2)
            *buf = append(*buf, '/')
            itoa(buf, day, 2)
            *buf = append(*buf, ' ')
        }

        if l.flag&(Ltime|Lmicroseconds) != 0 { // 如果有设置Ltime或Lmicroseconds标志，则将time附加上去, 格式HH:MM:SS(.MMMMMM), 注意该字段后面也留一个空格
            hour, min, sec := t.Clock()
            itoa(buf, hour, 2)
            *buf = append(*buf, ':')
            itoa(buf, min, 2)
            *buf = append(*buf, ':')
            itoa(buf, sec, 2)

            if l.flag&Lmicroseconds != 0 { // 如果需要记录微秒时间，则附加微秒时间, 毫秒时间记录格式为保留6位小数
                *buf = append(*buf, '.')
                itoa(buf, t.Nanosecond()/1e3, 6) // 取纳秒时间除以1000, 得到毫秒时间， 然后6位形式展示，不足前补零。
            }

            *buf = append(*buf, ' ')
        }
    }

    if l.flag&(Lshortfile|Llongfile) != 0 { // 设置日志记录文件名，长文件名或短文件名, 短文件名只显示文件名字，常文件显示完整路径。 文件所在行号信息
        if l.flag&Lshortfile != 0 {
            short := file
            for i := len(file) - 1; i > 0; i-- {
                if file[i] == '/' {
                    short = file[i+1:]
                    break
                }
            }

            file = short
        }

        *buf = append(*buf, file...)
        *buf = append(*buf, ':')

        itoa(buf, line, -1)
        *buf = append(*buf, ": "...)
    }
}
```

## 日志记录输出方法

log暴露了底层的日志记录输出方法Output, 同时还提供了几类便利的日志输出方法Print, Fatal, Panic。

### Output方法

Output是为日志事件记录输出的方法。它接受两个参数:
- calldepth: 调用深度，用于寻找PC, 一般都会提供。 尽管在所有预定义中都使用的是2。
- s: 日志信息字符串。

```
func (l *Logger) Output(calldepth int, s string) error {
    now := time.Now() // get this early.
    var file string
    var line int
    l.mu.Lock()
    defer. l.m.Unlock()

    if l.flag&(Lshortfile|Llongfile) != 0 {
        // 在获取调用者信息的时候释放锁, 因为这个操作太昂贵了。
        l.mu.Unlock()
        var ok bool
        _, file, line, ok = runtime.Caller(calldepth)

        if !ok { // 没有获取到调用者信息，则提供一个默认的值
            file = "???"
            line = 0
        }
        l.mu.Lock() // 获取到调用者信息后，重新上锁
    }

    l.buf = l.buf[0:]
    l.formatHeader(&l.buf, now, file, line) // 先格式化日志信息头内容
    l.buf = append(l.buf, s...)             // 将日志内容信息附加后面
    if len(s) == 0 || s[len(s) - 1] != '\n' { // 如果日志信息为空或者末尾不是换行符，则自动添加换行符
        l.buf = append(l.buf, '\n')
    }

    -, err := l.out.Write(l.buf)
    return err
}
```

我们需要注意下上面互斥锁的使用。互斥锁如果加在操作昂贵的动作上加锁, 那么就会造成性能瓶颈。上面代码采用了先释放锁，等到这些昂贵资源查询出来的时候，再次上锁。

### Print系方法
```
// Printf call l.Output to print to the logger.
// Arguments are handled in the manner of fmt.Printf.
func (l *Logger) Printf(format string, v ...interface{}) {
    l.Output(2, fmt.Sprintf(format, v...))
}

// Print calls l.Output to print to the logger.
// Arguments are handled in the manner of fmt.Print.
func (l *Logger) Print(v ...interface{}) { l.Output(2, fmt.Sprint(v...)) }

// Println calls l.Output to print to the logger.
// Arguments are handled in the manner of fmt.Println.
func (l *Logger) Println(v ...interface{}) { l.Output(2, fmt.Sprintln(v...)) }
```
- log.Print()
- log.Printf()
- log.Println()

### Fatal系方法
```go
// Fatal is equivalent to l.Print() followed by a call to os.Exit(1).
func (l *Logger) Fatal(v ...interface{}) {
	l.Output(2, fmt.Sprint(v...))
	os.Exit(1)
}

// Fatalf is equivalent to l.Printf() followed by a call to os.Exit(1).
func (l *Logger) Fatalf(format string, v ...interface{}) {
	l.Output(2, fmt.Sprintf(format, v...))
	os.Exit(1)
}

// Fatalln is equivalent to l.Println() followed by a call to os.Exit(1).
func (l *Logger) Fatalln(v ...interface{}) {
	l.Output(2, fmt.Sprintln(v...))
	os.Exit(1)
}
```

- log.Fatal()
- log.Fatalf()
- log.Fatalln()

### Panic系方法

```go
// Panic is equivalent to l.Print() followed by a call to panic().
func (l *Logger) Panic(v ...interface{}) {
	s := fmt.Sprint(v...)
	l.Output(2, s)
	panic(s)
}

// Panicf is equivalent to l.Printf() followed by a call to panic().
func (l *Logger) Panicf(format string, v ...interface{}) {
	s := fmt.Sprintf(format, v...)
	l.Output(2, s)
	panic(s)
}

// Panicln is equivalent to l.Println() followed by a call to panic().
func (l *Logger) Panicln(v ...interface{}) {
	s := fmt.Sprintln(v...)
	l.Output(2, s)
	panic(s)
}
```

- log.Panic()
- log.Panicf()
- log.Panicln()
