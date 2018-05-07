Go语言TCP/IP网络编程
===================

乍一看，通过TCP/IP层连接两个进程会感觉可怕， 但是在Go语言中可能比你想象的要简单的多。

## TCP/IP层发送数据的应用场景
当然很多情况下，不是大多数情况下，使用更高级别的网络协议毫无疑问会更好，因为可以使用华丽的API, 它们隐藏了很多技术细节。现在根据不同的需求，有很多选择，比如消息队列协议, gRPC, protobuf, FlatBuffers, RESTful网站API, websocket等等。

然而在一些特殊的场景下，特别是小型项目，选择任何其他方式都会感觉太臃肿了，更不用说你需要引入额外的依赖包了。

幸运的是，使用标准库的net包来创建简单的网络通信不比你所见到的要困难。

因为Go语言中有下面两点简化。

### 简化1: 连接就是io流
net.Conn接口实现了io.Reader, io.Writer和io.Closer接口。 因此可以像对待其他io流一样对待TCP连接。

你可能会认为:"好，我能在TCP中发送字符串或字节分片，非常不错，但是遇到复杂的数据结构怎么办? 例如我们遇到的是结构体类型的数据?"

### 简化2: Go语言知道如何有效的解码复杂的类型
当说到通过网络发送编码的结构化数据，首先想到的就是JSON。 不过先稍等一下 - Go语言的标准库encoding/gob包提供了一种序列化和发序列话Go数据类型的方法，它无需给结构体、Go语言不兼容的JSON添加字符串标签, 或者等待使用json.Unmarshal来费劲的将文本解析为二进制数据。

gob编码解码可以直接操作io流，这一点很完美的匹配第一条简化。

下面我们就通过这两条简化规则一起实现一个简单的App。

## 这个简单APP的目标

这个app应该做两件事情:
- 发送和接收简单的字符串消息。
- 通过gob发送和接收结构体。

第一部分，发送简单字符串，将演示无需借助高级协议的情况下，通过TCP/IP网络发送数据是多么简单。
第二部分，稍微深入一点，通过网络发送完整的结构体，这些结构体使用字符串、分片、映射、甚至包含到自身的递归指针。

辛亏有gob包，要做到这些不费吹灰之力。
```text
客户端                                        服务端

待发送结构体                                解码后结构体
testStruct结构体                            testStruct结构体
    |                                             ^
    V                                             |
gob编码       ---------------------------->     gob解码
    |                                             ^
    V                                             |   
   发送     ============网络=================    接收
```

## 通过TCP发送字符串数据的基本要素

### 发送端上
发送字符串需要三个简单的步骤:
- 打开对应接收进程的连接。
- 写字符串。
- 关闭连接。

net包提供了一对实现这个功能的方法。
- ResolveTCPAddr(): 该函数返回TCP终端地址。
- DialTCP(): 类似于TCP网络的拨号。

这两个方法都是在go源码的src/net/tcpsock.go文件中定义的。
```go
func ResolveTCPAddr(network, address string) (*TCPAddr, error) {
	switch network {
	case "tcp", "tcp4", "tcp6":
	case "": // a hint wildcard for Go 1.0 undocumented behavior
		network = "tcp"
	default:
		return nil, UnknownNetworkError(network)
	}
	addrs, err := DefaultResolver.internetAddrList(context.Background(), network, address)
	if err != nil {
		return nil, err
	}
	return addrs.forResolve(network, address).(*TCPAddr), nil
}
```

ResolveTCPAddr()接收两个字符串参数。
- network: 必须是TCP网络名，比如tcp, tcp4, tcp6。
- address: TCP地址字符串，如果它不是字面量的IP地址或者端口号不是字面量的端口号， ResolveTCPAddr会将传入的地址解决成TCP终端的地址。否则传入一对字面量IP地址和端口数字作为地址。address参数可以使用host名称，但是不推荐这样做，因为它最多会返回host名字的一个IP地址。

ResolveTCPAddr()接收的代表TCP地址的字符串(例如localhost:80, 127.0.0.1:80, 或[::1]:80, 都是代表本机的80端口), 返回(net.TCPAddr指针, nil)(如果字符串不能被解析成有效的TCP地址会返回(nil, error))。

```go
func DialTCP(network string, laddr, raddr *TCPAddr) (*TCPConn, error) {
	switch network {
	case "tcp", "tcp4", "tcp6":
	default:
		return nil, &OpError{Op: "dial", Net: network, Source: laddr.opAddr(), Addr: raddr.opAddr(), Err: UnknownNetworkError(network)}
	}
	if raddr == nil {
		return nil, &OpError{Op: "dial", Net: network, Source: laddr.opAddr(), Addr: nil, Err: errMissingAddress}
	}
	c, err := dialTCP(context.Background(), network, laddr, raddr)
	if err != nil {
		return nil, &OpError{Op: "dial", Net: network, Source: laddr.opAddr(), Addr: raddr.opAddr(), Err: err}
	}
	return c, nil
}
```
DialTCP()函数接收三个参数:
- network: 这个参数和ResolveTCPAddr的network参数一样，必须是TCP网络名。
- laddr: TCPAddr类型的指针, 代表本地TCP地址。
- raddr: TCPAddr类型的指针，代表的是远程TCP地址。

它会连接拨号两个TCP地址，并返回这个连接作为net.TCPConn对象返回(连接失败返回error)。如果我们不需要对Dial设置有过多控制，那么我们就可以使用Dial()代替。

```go
func Dial(network, address string) (Conn, error) {
	var d Dialer
	return d.Dial(network, address)
}
```
Dial()函数接收一个TCP地址，返回一个一般的net.Conn。 这已经足够我们的测试用例了。然而如果你需要只有在TCP连接上的可用功能，可以使用TCP变体(DialTCP, TCPConn, TCPAddr等等)。

成功拨号之后，我们就可以如上所述的那样，将新的连接与其他的输入输出流同等对待了。我们甚至可以将连接包装进bufio.ReadWriter中，这样可以使用各种ReadWriter方法，例如ReadString(), ReadBytes, WriteString等等。
```go src/net/dial.go
func Open(addr string) (*bufio.ReadWriter, error) {
    conn, err := net.Dial("tcp", addr)
    if err != nil {
        return nil, errors.Wrap(err, "Dialing "+addr+" failed")
    }
    // 将net.Conn对象包装到bufio.ReadWriter中
    return bufio.NewReadWriter(bufio.NewReader(conn), bufio.NewWriter(conn)), nil
}
```

> 记住缓冲Writer在写之后需要调用Flush()方法, 这样所有的数据才会刷到底层网络连接中。

最后，每个连接对象都有一个Close()方法来终止通信。

### 微调(fine tuning)
Dialer结构体定义如下:

```go src/net/dial.go
type Dialer struct {
	Timeout time.Duration
	Deadline time.Time
	LocalAddr Addr
	DualStack bool
	FallbackDelay time.Duration
	KeepAlive time.Duration
	Resolver *Resolver
	Cancel <-chan struct{}
}
```
- Timeout: 拨号等待连接结束的最大时间数。如果同时设置了Deadline, 可以更早失败。默认没有超时。 当使用TCP并使用多个IP地址拨号主机名，超时会在它们之间划分。使用或不使用超时，操作系统都可以强迫更早超时。例如，TCP超时一般在3分钟左右。
- Deadline: 是拨号即将失败的绝对时间点。如果设置了Timeout, 可能会更早失败。0值表示没有截止期限， 或者依赖操作系统或使用Timeout选项。
- LocalAddr: 是拨号一个地址时使用的本地地址。这个地址必须是要拨号的network地址完全兼容的类型。如果为nil, 会自动选择一个本地地址。
- DualStack: 这个属性可以启用RFC 6555兼容的"[欢乐眼球(Happy Eyeballs)](https://en.wikipedia.org/wiki/Happy_Eyeballs)"拨号，当network是tcp时，address参数中的host可以被解析被IPv4和IPv6地址。这样就允许客户端容忍(tolerate)一个地址家族的网络规定稍微打破一下。
- FallbackDelay: 当DualStack启用的时候, 指定在产生回退连接之前需要等待的时间。如果设置为0， 默认使用延时300ms。
- KeepAlive: 为活动网络连接指定保持活动的时间。如果设置为0，没有启用keep-alive。不支持keep-alive的网络协议会忽略掉这个字段。
- Resolver: 可选项，指定使用的可替代resolver。
- Cancel: 可选通道，它的闭包表示拨号应该被取消。不是所有的拨号类型都支持拨号取消。 已废弃，可使用DialContext代替。
有两个可用选项可以微调。

因此Dialer接口提供了可以微调的两方面选项:

- DeadLine和Timeout选项: 用于不成功拨号的超时设置。
- KeepAlive选项: 管理连接的使用寿命(life span)。 

```go src/net.go
type Conn interface {
	Read(b []byte) (n int, err error)
	Write(b []byte) (n int, err error)
	Close() error
	LocalAddr() Addr
	RemoteAddr() Addr
	SetDeadline(t time.Time) error
	SetReadDeadline(t time.Time) error
	SetWriteDeadline(t time.Time) error
}
```
net.Conn接口是面向流的一般的网络连接。它具有下面这些接口方法:

- Read(): 从连接上读取数据。
- Write(): 向连接上写入数据。
- Close(): 关闭连接。
- LocalAddr(): 返回本地网络地址。
- RemoteAddr(): 返回远程网络地址。
- SetDeadline(): 设置连接相关的读写最后期限。等价于同时调用SetReadDeadline()和SetWriteDeadline()。
- SetReadDeadline(): 设置将来的读调用和当前阻塞的读调用的超时最后期限。
- SetWriteDeadline(): 设置将来写调用以及当前阻塞的写调用的超时最后期限。

Conn接口也有deadline设置; 有对整个连接的(SetDeadLine())，也有特定读写调用的(SetReadDeadLine()和SetWriteDeadLine())。

注意deadline是(wallclock)时间固定点。和timeout不同，它们新活动之后不会重置。因此连接上的每个活动必须设置新的deadline。

下面的样本代码没有使用deadline, 因为它足够简单，我们可以很容易看到什么时候会被卡住。Ctrl-C时我们手动触发deadline的工具。

### 接收端上
接收端步骤如下:
- 对本地端口打开监听。
- 当请求到来时，产生(spawn)goroutine来处理请求。
- 在goroutine中，读取数据。也可以选择性的发送响应。
- 关闭连接。

监听需要指定本地监听的端口号。一般来说，监听应用程序(也叫server)宣布监听的端口号，如果提供标准服务, 那么使用这个服务对应的相关端口。例如，web服务通常监听80来伺服HTTP, 443端口伺服HTTPS请求。 SSH守护默认监听22端口, WHOIS服务使用端口43。

```go src/net.go
type Listener interface {
	// Accept waits for and returns the next connection to the listener.
	Accept() (Conn, error)

	// Close closes the listener.
	// Any blocked Accept operations will be unblocked and return errors.
	Close() error

	// Addr returns the listener's network address.
	Addr() Addr
}
```
```go src/net/dial.go
func Listen(network, address string) (Listener, error) {
	addrs, err := DefaultResolver.resolveAddrList(context.Background(), "listen", network, address, nil)
	if err != nil {
		return nil, &OpError{Op: "listen", Net: network, Source: nil, Addr: nil, Err: err}
	}
	var l Listener
	switch la := addrs.first(isIPv4).(type) {
	case *TCPAddr:
		l, err = ListenTCP(network, la)
	case *UnixAddr:
		l, err = ListenUnix(network, la)
	default:
		return nil, &OpError{Op: "listen", Net: network, Source: nil, Addr: la, Err: &AddrError{Err: "unexpected address type", Addr: address}}
	}
	if err != nil {
		return nil, err // l is non-nil interface containing nil pointer
	}
	return l, nil
}
```
net包实现服务端的核心部分是:

net.Listen()在给定的本地网络地址上来创建新的监听器。如果只传端口号给它，例如":61000", 那么监听器会监听所有可用的网络接口。 这相当方便，因为计算机通常至少提供两个活动接口，回环接口和最少一个真实网卡。 这个函数成功的话返回Listener。

Listener接口有一个Accept()方法用来等待请求进来。然后它接受请求，并给调用者返回新的连接。Accept()一般来说都是在循环中调用，能够同时服务多个连接。每个连接可以由一个单独的goroutine处理，正如下面代码所示的。

## 代码部分

与其让代码来回推送一些字节，我更想要它演示一些更有用的东西。 我想让它能给服务器发送带有不同数据载体的不同命令。服务器应该能标识每个命令和解码命令数据。

我们代码中客户端会发送两种类型的命令: "STRING"和"GOB"。它们都以换行符终止。

"STRING"命令包含一行字符串数据，可以通过bufio中的简单读写操作来处理。

"GOB"命令由结构体组成，这个结构体包含一些字段，包含一个分片和映射，甚至指向自己的指针。 正如你所见，当运行这个代码时，gob包能通过我们的网络连接移动这些数据没有什么稀奇(fuss).

我们这里基本上都是一些即席协议(ad-hoc protocol: 特设的、特定目的的、即席的、专案的), 客户端和服务端都遵循它，命令行后面是换行，然后是数据。对于每个命令来说，服务端必须知道数据的确切格式，知道如何处理它。

要达到这个目的，服务端代码采取两步方式实现。

- 第一步: 当Listen()函数接收到新连接，它会产生一个新的goroutine来调用handleMessage()。 这个函数从连接中读取命令名, 从映射中查询合适的处理器函数，然后调用它。
- 第二步: 选择的处理器函数读取并处理命令行的数据。

```go
package main

import (
	"bufio"
	"encoding/gob"
	"flag"
	"github.com/pkg/errors"
	"io"
	"log"
	"net"
	"strconv"
	"strings"
	"sync"
)

type complexData struct {
	N int
	S string
	M map[string]int
	P []byte
	C *complexData
}

const (
	Port = ":61000"
)
```

### Outcoing connections(发射连接)
使用发射连接是一种快照。net.Conn满足io.Reader和io.Writer接口，因此我们可以将TCP连接和其他任何的Reader和Writer一样看待。

```go
func Open(addr string) (*bufio.ReadWriter, error) {
    log.Println("Dial " + addr)
    conn, err := net.Dial("tcp", addr)

    if err != nil {
        return nil, errors.Wrap(err, "Dialing " + addr + " failed")
    }

    return bufio.NewReadWriter(bufio.NewReader(conn), bufio.NewWriter(conn)), nil
}
```

打开TCP地址的连接。它返回一个带有超时的TCP连接，并将其包装进缓冲的ReadWriter。拨号远程进程。注意本地端口是实时(on the fly)分配的。如果必须指定本地端口号，请使用DialTCP()方法。

### 进入连接

这节有点涉及到对进入数据的准备环节处理。根据我们前面介绍的ad-hoc协议，命令名+换行符+数据+换行符。自然数据是和具体命令相关的。要处理这样的情况，我们创建了一个Endpoint对象，它具有下面的属性:

- 它允许注册一个或多个处理器函数，每个函数可以处理一个特殊的命令。
- 它根据命令名将具体命令调度到相关的处理器函数。

首先我们声明一个HandleFunc类型，该类型为接收一个bufio.ReadWriter指针值的函数类型， 也就是后面我们要为每种不同命令注册的处理器函数。它接收的参数是使用ReadWriter接口包装的net.Conn连接。
```go
type HandleFunc func(*bufio.ReadWriter)
```

然后我们声明一个Endpoint结构体类型，它有三个属性:
- listener: net.Listen()返回的Listener对象。
- handler: 用于保存已注册的处理器函数的映射。
- m: 一个互斥锁，用于解决map的多goroutine不安全的问题。

```go
type Endpoint struct {
    listener net.Listener
    handler map[string]HandleFunc
    m sync.RWMutex        // Maps不是线程安全的，因此需要互斥锁来控制访问。
}

func NewEndpoint() *Endpoint {
    return &Endpoint{
        handler: map[string]HandleFunc{},
    }
}

func (e *Endpoint) AddHandleFunc(name string, f HandleFunc) {
    e.m.Lock()
    e.handler[name] = f
    e.m.Unlock()
}

func (e *Endpoint) Listen() error {
    var err error
	e.listener, err = net.Listen("tcp", Port)
	if err != nil {
		return errors.Wrap(err, "Unable to listen on "+e.listener.Addr().String()+"\n")
	}
	log.Println("Listen on", e.listener.Addr().String())
	for {
		log.Println("Accept a connection request.")
		conn, err := e.listener.Accept()
		if err != nil {
			log.Println("Failed accepting a connection request:", err)
			continue
		}
		log.Println("Handle incoming messages.")
		go e.handleMessages(conn)
	}
}

// handleMessages读取连接到第一个换行符。 基于这个字符串，它会调用恰当的HandleFunc。

func (e *Endpoint) handleMessages(conn net.Conn) {
    // 将连接包装到缓冲reader以便于读取
    rw := bufio.NewReadWrite(bufio.NewReader(conn), bufio.NewWriter(conn))
    defer conn.Close()

    // 从连接读取直到遇到EOF. 期望下一次输入是命令名。调用注册的用于该命令的处理器。

    for {
        log.Print("Receive command '")
        cmd, err := rw.ReadString('\n')
        switch {
        case err == io.EOF:
            log.Println("Reached EOF - close this connection.\n  ---")
            return
        case err != nil:
            log.Println("\nError reading command. Got: '" + cmd + "'\n", err)
        }

        // 修剪请求字符串中的多余回车和空格- ReadString不会去掉任何换行。

        cmd = strings.Trim(cmd, "\n ")
        log.Println(cmd + "'")

        // 从handler映射中获取恰当的处理器函数, 并调用它。

        e.m.Lock()
        handleCommand, ok := e.handler[cmd]
        e.m.Unlock()

        if !ok {
            log.Println("Command '" + cmd + "' is not registered.")
            return
        }

        handleCommand(rw)
    }
}
```
NewEndpoint()函数是Endpoint的工厂函数。它只对handler映射进行了初始化。为了简化问题，假设我们的终端监听的端口好是固定的。

Endpoint类型声明了几个方法:
- AddHandleFunc(): 使用互斥锁为handler属性安全添加处理特定类型命令的处理器函数。
- Listen(): 对终端端口的所有接口启动监听。 在调用Listen之前，至少要通过AddHandleFunc()注册一个handler函数。
- HandleMessages(): 将连接用bufio包装起来，然后分两步读取，首先读取命令加换行，我们得到命令名字。 然后通过handler获取注册的该命令对应的处理器函数, 然后调度这个函数来执行数据读取和解析。

> 注意上面如何使用动态函数的。 根据命令名查找具体函数，然后这个具体函数赋值给handleCommand, 其实这个变量类型为HandleFunc类型, 即前面声明的处理器函数类型。

Endpoint的Listen方法调用之前需要先至少注册一个处理器函数。因此我们下面定义两个类型的处理器函数: handleStrings和handleGob。

handleStrings()函数接收和处理我们即时协议中只发送字符串数据的处理器函数。handleGob()函数是接收并处理发送的gob数据的复杂结构体。handleGob稍微复杂一点，除了读取数据外，我们海需要解码数据。

我们可以看到连续两次使用rw.ReadString('\n'), 读取字符串，遇到换行停止, 将读到的内容保存到字符串中。注意这个字符串是包含末尾换行的。

另外对于普通字符串数据来说，我们直接用bufio包装连接后的ReadString来读取。而对于复杂的gob结构体来说，我们使用gob来解码数据。

```go
func handleStrings(rw *bufio.ReadWriter) {
    log.Print("Receive STRING message:")
    s, err := rw.ReadString('\n')
    if err != nil {
        log.Println("Cannot read from connection.\n", err)
    }

    s = strings.Trim(s, "\n ")
    log.Println(s)

    -, err = rw.WriteString("Thank you.\n")
    if err != nil {
        log.Println("Cannot write to connection.\n", err)
    }

    err = rw.Flush()
    if err != nil {
        log.Println("Flush failed.", err)
    }
}

func handleGob(rw *bufio.ReadWriter) {
    log.Print("Receive GOB data:")
    var data complexData
     
    dec := gob.NewDecoder(rw)
    err := dec.Decode(&data)

    if err != nil {
        log.Println("Error decoding GOB data:", err)
        return
    }

    log.Printf("Outer complexData struct: \n%#v\n", data)
    log.Printf("Inner complexData struct: \n%#v\n", data.C)
}
```

### 客户端和服务端函数
一切就绪，我们可以准备我们的客户端和服务端函数了。

- 客户端函数连接到服务器并发送STRING和GOB请求。
- 服务端开始监听请求并触发恰当的处理器。

```go
// 当应用程序使用-connect=ip地址的时候被调用

func client(ip string) error {
    testStruct := complexData{
        N: 23,
        S: "string data",
        M: map[string]int{"one": 1, "two": 2, "three": 3},
        P: []byte("abc"),
        C: &complexData{
            N: 256,
            S: "Recursive structs? Piece of cake!",
            M: Map[string]int{"01": "10": 2, "11": 3},
        },
    }

    rw, err := Open(ip + Port)
    if err != nil {
        return errors.Wrap(err, "Client: Failed to open connection to " + ip + Port)
    }

    log.Println("Send the string request.")

    n, err := rw.WriteString("STRING\n")
    if err != nil {
        return errors.Wrap(err, "Could not send the STRING request (" + strconv.Itoa(n) + " bytes written)")
    }

    // 发送STRING请求。发送请求名并发送数据。

    log.Println("Send the string request.")

    n, err = rw.WriteString("Additional data.\n")
    if err != nil {
        return errors.Wrap(err, "Could not send additional STRING data (" + strconv.Itoa(n) + " bytes written)")
    }

    log.Println("Flush the buffer.")
    err = rw.Flush()
    if err != nil {
        return errors.Wrap(err, "Flush failed.")
    }

    // 读取响应

    log.Println("Read the reply.")

    response, err := rw.ReadString('\n')
    if err != nil {
        return errors.Wrap(err, "Client: Failed to read the reply: '" + response + "'")
    }

    log.Println("STRING request: got a response:", response)
   
    // 发送GOB请求。 创建一个encoder直接将它转换为rw.Send的请求名。发送GOB

    log.Println("Send a struct as GOB:")
    log.Printf("Outer complexData struct: \n%#v\n", testStruct)
	log.Printf("Inner complexData struct: \n%#v\n", testStruct.C)
    enc := gob.NewDecoder(rw)
    n, err = rw.WriteString("GOB\n")
    if err != nil {
        return errors.Wrap(err, "Could not write GOB data (" + strconv.Itoa(n) + " bytes written)")
    }

    err = enc.Encode(testStruct)
    if err != nil {
        return errors.Wrap(err, "Encode failed for struct: %#v", testStruct)
    }

    err = rw.Flush()
    if err != nil {
        return errors.Wrap(err, "Flush failed.")
    }

    return nil
}
```

客户端函数在执行应用程序时指定connect标志的时候执行，这点后面的代码可以看到。

下面是服务端程序server。服务端监听进来的请求并根据请求命令名将它们调度给注册的具体相关处理器。

```go
func server() error {
    endpoint := NewEndpoint()

    // 添加处理器函数

    endpoint.AddHandleFunc("STRING", handleStrings)
    endpoint.AddHandleFunc("GOB", handleGOB)

    // 开始监听

    return endpoint.Listen()
}
```

### main函数
下面的main函数既可以启动客户端也可以启动服务端， 依赖于是否设置connect标志。 如果没有这个标志，则以服务器启动进程， 监听进来的请求。如果有标志， 启动为客户端，并连接到这个标志指定的主机。

可以使用localhost或127.0.0.1在同一机器上运行这两个进程。
```go
func main() {
    connect := flag.String("connect", "", "IP address of process to join. If empty, go into the listen mode.")
    flag.Parse()

    // 如果设置了connect标志，进入客户端模式

    if *connect != '' {
        err := client(*connect)
        if err != nil {
            log.Println("Error:", errors.WithStack(err))
        }
        log.Println("Client done.")
        return
    }

    // 否则进入服务端模式

    err := server()
    if err != nil {
        log.Println("Error:", errors.WithStack(err))
    }

    log.Println("Server done.")
}

// 设置日志记录的字段标志

func init() {
    log.SetFlags(log.Lshortfile)
}
```

## 如何获取并运行代码

第一步: 获取代码。 注意-d标志自动安装二进制到$GOPATH/bin目录。
```go
go get -d github.com/appliedgo/networking
```

第二步: cd到源代码目录。
cd $GOPATH/src/github.com/appliedgo/networking

第三步: 运行服务端。
go run networking.go

第四步: 打开另外一个shell, 同样进入到源码目录(第二步), 然后运行客户端。
go run networking.go -connect localhost

## Tips
如果你想稍微修改下源代码，下面是一些建议:
* 在不同机器运行客户端和服务端(同一个局域网中).
* 用更多的映射和指针来增强(beef up)complexData, 看看gob如何应对它(cope with it)。
* 同时启动多个客户端，看看服务端是否能处理它们。


> 2017-02-09: map不是线程安全的，因此如果在不同的goroutine中使用同一个map, 应该使用互斥锁来控制map的访问。
> 而上面的代码，map在goroutine启动之前已经添加好了， 因此你可以安全的修改代码，在handleMessages goroutine已经运行的时候调用AddHandleFunc()。

## 本章所学知识点总结
---- 2018-05-04 -----

- bufio的应用。
- gob的应用。
- map在多goroutine间共享时是不安全的。

## 参考链接
* [TCP/IP网络](https://appliedgo.net/networking/)
* [简单的TCP服务端和客户端](https://systembash.com/a-simple-go-tcp-server-and-tcp-client/)
* [gob数据](https://blog.golang.org/gobs-of-data)
