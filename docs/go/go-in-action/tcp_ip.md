TCP/IP网络
=================

在TCP/IP层连接两个进程起初看起来可能感觉很可怕，但是在Go语言中可能比你想象的要简单的多。

## 谁需要在TCP/IP层发送数据？
当然，很多情况下，如果不是绝大多数情况，毫无疑问使用高级别的网络协议会更好，因为它可以在设想的API之下隐藏很多技术细节。目前有很多选择，根据不同的需求，消息队列协议,gRPC, protobuf, FlatBuffers, RESTful web APIs, WebSockets等等。

然而在一些特殊的情况下，尤其是小项目，你选择的任何方法都会看起来太大了，更不用说你需要引入的额外依赖包了。

幸运的是，使用标准库net包创建简单的网络通信不会比你看到的困难。

## 简则1: 连接是io流
net.Conn接口实现了io.Reader, io.Writer和io.Closer接口。因此你可以像使用其他io流一样使用TCP连接。
我知道你会认为- "OK，我可以在TCP连接中发送字符串或字节分片。那是非常不错，但是对于复杂数据结构怎么办？例如结构体之类的?"

## 简则2: Go语言知道如何有效的解码复杂的类型
当说到通过网络发送编码的结构化数据，JSON首先映入脑海。 但是请稍等- Go的标准库encoding/gob包提供了一种序列化和反序列化Go数据类型的方式，它无需给结构体添加字符串标签，处理json/go不兼容, 或者等待json.Unmarshal来费力的解析文本为二进制数据。

gob编码解码可以直接操作io流, 这点很完美的匹配第一条简则。

下面我们将这两条简则放在一起实现一个简单的App。

## 这个简单APP的目标
这个app应该做两件事情:
- 发送和接收简单的字符串消息。
- 通过gob发送和接收结构体

第一部分，发送简单字符串将会演示通过TCP/IP网络无需借助高级协议发送数据是多么简单。
第二部分，稍微深入一点，通过网络发送完整的结构体， 使用字符串、分片、映射、甚至到结构体自身的递归指针。

辛亏有gob包，做到这些毫不费力。

## 通过TCP发送字符串数据的基本要素

### 发送端
发送字符串需要三个简单的步骤:
- 打开到接收进程的连接。
- 写字符串。
- 关闭连接。

net包提供了两种实现它的方法。

ResolveTCPAddr()接收一个代表TCP地址的字符串(就像localhost:80, 127.0.0.1:80, 或[::1]:80，都是代表本地机器的80端口), 染灰一个net.TCPAddr(或者因为字符串不能被解析成有效的TCP地址而返回错误)。

DialTCP()接收一个net.TCPAddr参数并连接到这个地址。它将打开的连接作为net.TCPConn对象返回(或者连接尝试失败返回error). 如果我们不需要对Dial设置有过细致的控制，我们可以使用net.Dial()代替。这个函数直接接收一个地址字符串，返回一个一般的net.Conn对象。 这已经足够满足我们的测试用例了。然而，如果你需要只有在TCP连接上可用的功能，可以使用TCP变体(DialTCP, TCPConn, TCPAddr等等).

成功拨号之后，我们可以将新的连接与其他的输入/输出流同等对待，如上面提到的那样。我们甚至可以将连接包装进bufio.ReadWriter，这样可以得益于各种ReadWriter方法，就像ReadString(), ReadBytes, WriteString等等。

> 记住缓冲Writer在写之后需要调用Flush(), 这样所有的数据才会刷到底层网络连接中。

最后，每个连接对象都有一个Close()方法来终止通信。

### 微调(fine tuning)

有两个可用选项可以微调。

Dialer接口提供了这些选项(在所有选项中的):

- DeadLine和Timeout选项: 用于不成功拨号的超时设置。
- KeepAlive选项: 管理连接的使用寿命(life span)。 

Conn接口也有deadline设置; 有对整个连接的(SetDeadLine())，也有特定读写调用的(SetReadDeadLine()和SetWriteDeadLine())。

注意deadline是(wallclock)时间固定点。和timeout不同，它们对每个活动不会重置。因此连接上的每个活动必须设置新的deadline。

下面的样本代码没有使用deadline, 因为它足够简单，我们可以很容易看到什么时候会被卡住。Ctrl-C时我们手动触发deadline的工具。

### 接收端
接收端步骤如下:
- 在本地端口打开监听。
- 当请求到来时，产生(spawn)goroutine来处理请求。
- 在goroutine中，读取数据。 可选的发送响应。
- 关闭连接。

监听需要指定本地监听的端口号。典型来说，监听应用程序(也叫server)宣布监听的端口号， 或者如果提供标准服务, 那么使用这个服务对应相关的端口。例如，web服务通常监听80来伺服HTTP, 443端口伺服HTTPS请求。 SSH守护默认监听22端口, WHOIS服务使用端口43。

net包实现服务端的核心部分是:

net.Listen()在给定的本地网络地址上来创建新的监听器。如果只传端口号给它，例如":61000", 那么监听器会监听所有可用的网络接口。 这相当方便，因为计算机通常至少提供两个活动接口，回环接口和最少一个网卡。

监听器的Accept()方法等待请求进来。 然后它接受请求，并给调用者返回新的连接。Accept()一般来说都是在循环中调用，能够同时服务多个连接。每个连接可以由一个单独的goroutine处理，正如下面代码所示的。

### 代码
与其让代码来回推送一些字节，我更想要它演示一些更有用的东西。 我想让它能给服务器发送带有不同数据载体的不同命令。服务器应能标志每个命令和解码命令数据。

因此代码中的客户端发送了两个测试命令: "STRING"和"GOB". 它们都以换行符终止。

"STRING"命令包含一行字符串数据， 可以通过bufio中的简单读写操作来处理。

"GOB"命令由结构体组成，这个结构体包含一些字段，包含一个分片和映射，甚至指向自己的指针。 正如你所见，当运行这个代码时，gob包通过我们的网络连接移动这些数据没有什么稀奇(fuss).

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

## Outcoing connections(发射连接)
使用传出连接是一种快照。net.Conn满足io.Reader和io.Writer接口，因此我们可以将TCP连接当作任何其他的Reader和Writer看待。

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
打开TCP地址的连接。 它返回一个带有超时的TCP连接，并将其包装进缓冲的ReadWriter。拨号远程进程。 注意本地端口是实时(on the fly)选择的。如果本地端口好必须是指定的， 使用DialTCP()方法。

## 进入连接
有点涉及到进入数据的准备。 根据我们的ad-hoc协议，我们接到命令名以换行结束， 后面跟着数据。 自然数据依赖具体的命令。要处理这样的情况， 我们创建了一个Endpoint对象，带有下面的属性:
- 它允许注册一个或多个处理器函数，每个函数可以处理一个特殊的命令。
- 它根据命令名将具体命令调度到相关的处理器。

```go
// HandleFunc是一个函数， 它处理进入的命令。他接受打开的用ReadWriter接口包装的连接。
type HandleFunc func(*bufio.ReadWriter)

// Endpoint为其他进程提供可以发送数据的终端。
type Endpoint struct {
    listener net.Listener
    handler map[string]HandleFunc

    m sync.RWMutex        // Maps不是线程安全的，因此需要互斥锁来控制访问。
}

// NewEndpoint创建新的终端. 要让事情简化，终端监听固定的端口数。
func NewEndpoint() *Endpoint {
    // 创建新的Endpoint使用空处理器函数列表。
    return &Endpoint{
        handler: map[string]HandleFunc{},
    }
}

// AddHandleFunc为处理进来数据添加新的函数
func (e *Endpoint) AddHandleFunc(name string, f HandleFunc) {
    e.m.Lock()
    e.handler[name] = f
    e.m.Unlock()
}

// Listen启动监听终端所有接口端口. 在这之前至少要有一个处理器通过AddHandlerFunc()加入进去。
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

    // 
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
    }
}
```

## 参考链接
* [TCP/IP网络](https://appliedgo.net/networking/)
