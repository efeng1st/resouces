使用Go语言开发RESTFul JSON API
=================================
本文，我们不仅涵盖如何使用Go语言创建RESTful JSON API, 也会讨论有关RESTful的好设计。如果你过去曾经消费过不遵循良好设计的API, 你会因为去消费差劲API而却步。希望在本文之后，你会更好的了解一个良好的API应该是什么样子。

## JSON API是什么?
在JSON之前，使用的是XML。如果同时使用过JSON和XML, 毫无疑问JSON是胜利者。我不会深入JSON API的概念，因为它已经在jsonapi.org中阐述的很清楚了。
## 基本的Web服务器
一开始从根本上说，RESTful服务首先是Web服务。这里是一个非常基本的Web服务器，它通过简单的输出请求URL来响应请求:
```go
package main

import (
	"fmt"
	"html"
	"log"
	"net/http"
)

func main() {
	http.HandleFunc("/", func(w http.ResponseWriter, r *http.Request) {
		fmt.Fprintf(w, "Hello, %q", html.EscapeString(r.URL.Path))
	})

	log.Fatal(http.ListenAndServe(":8080", nil))
}
```
上面代码运行后，可以直接通过浏览器访问http://localhost:8080来访问。

## 添加路由
虽然标准库包含有router, 但是我发现很多人对它的工作原理感觉很困惑。 我在自己的项目中使用过各种不同的第三方router库。 最值得一提的是Gorilla Web ToolKit的mux router。

另外一个流行的router是来自Julien Schmidt的叫做httprouter的包。
```go
package main

import (
	"fmt"
	"html"
	"log"
	"net/http"

	"github.com/gorilla/mux"
)

func main() {
	router := mux.NewRouter().StrictSlash(true)
	router.HandleFunc("/", Index)

	log.Fatal(http.ListenAndServe(":8080", router))
}

func Index(w http.ResponseWriter, r *http.Request) {
	fmt.Fprintf(w, "Hello, %q", html.EscapeString(r.URL.Path))
}
```
要运行上面的代码，首先使用go get获取mux router的源代码：
```
> go get github.com/gorilla/mux
```
上面代码创建了一个基本的路由器，添加路由/并将Index处理器赋予给它，当终端调用的时候它会运行。 你会看到之前我们直接访问http://localhost:8080/foo能工作，而现在不能了。因为我们之前没有定义路由。 而这里定义了路由，只有http://localhost:8080才有有效响应。

## 创建一些基本的路由
我们现在已经有了路由了，是时候给它添加一些路由了。

假设我们正要创建一个基本的ToDo应用。

```go
package main

import (
	"fmt"
	"log"
	"net/http"

	"github.com/gorilla/mux"
)

func main() {
	router := mux.NewRouter().StrictSlash(true)
	router.HandleFunc("/", Index)
	router.HandleFunc("/todos", TodoIndex)
	router.HandleFunc("/todos/{todoId}", TodoShow)

	log.Fatal(http.ListenAndServe(":8080", router))
}

func Index(w http.ResponseWriter, r *http.Request) {
	fmt.Fprintln(w, "Welcome!")
}

func TodoIndex(w http.ResponseWriter, r *http.Request) {
	fmt.Fprintln(w, "Todo Index!")
}

func TodoShow(w http.ResponseWriter, r *http.Request) {
	vars := mux.Vars(r)
	todoId := vars["todoId"]
	fmt.Fprintln(w, "Todo Show:", todoId)
}
```
我们添加了另外两个路由: todos和todos/{todoId}。

这就是RESTful API设计的开始。

请注意最后一个路由我们给路由后面添加了一个变量叫做todoId。

这样就允许我们传递id给路由，并且能使用具体的记录来响应请求。

## 基本模型
路由现在已经就绪，是时候创建一个Model，可以用于发送和检索数据。 在Go语言中，结构体一般可用于model。 很多其他语言使用类来作为model。
```go
package model

import (
	"time"
)

type Todo struct {
	Name      string
	Completed bool
	Due       time.Time
}

type Todos []Todo
```
上面我们定义了一个Todo结构体，用于表示待做项。 另外我们还定义了一种类型Todos, 它表示待做列表，是一个数组，或者说是一个分片。
稍后你就会看到这样会变得非常有用。

## 返回一些JSON
我们有了基本的模型，那么我们可以模拟一些真实的响应了。我们可以为TodoIndex模拟一些静态的数据列表。

```go
package main

import (
	"encoding/json"
	"fmt"
	"log"
	"net/http"

	"github.com/gorilla/mux"
	"model"
)

// ...

func TodoIndex(w http.ResponseWriter, r *http.Request) {
	todos := model.Todos{
		model.Todo{Name: "Write presentation"},
		model.Todo{Name: "Host meetup"},
	}

	json.NewEncoder(w).Encode(todos)
}

// ...
```
现在我们创建了一个静态的Todos分片来响应客户端请求。注意，如果你请求http://localhost:8080/todos, 就会得到下面的响应:
```json
[
    {
        "Name": "Write presentation",
        "Completed": false,
        "Due": "0001-01-01T00:00:00Z"
    },
    {
        "Name": "Host meetup",
        "Completed": false,
        "Due": "0001-01-01T00:00:00Z"
    }
]
```
## 更好的Model
对于经验丰富的老兵来说，你可能已经发现了一个问题。虽然看起来微不足道，但是响应JSON的key首字母大写不是习惯JSON的做法。 那么下面教你如何解决这个问题:
```go
type Todo struct {
    Name      string    `json:"name"`
    Completed bool      `json:"completed"`
    Due       time.Time `json:"due"`
}
```

其实很简单，就是在结构体中添加标签属性， 这样可以完全控制你的结构体如何编排(marshalled)成JSON的。

## 好吧，我们需要把它们拆开!
到这个程度，项目需要稍微重构一下。 我们需要将代码拆分成多个文件。

我们准备创建下面的文件，然后将相应代码移到具体的代码文件中:
- main.go: 程序入口文件。
- handlers.go: 路由相关的处理器。
- routes.go: 路由。
- todo.go: todo相关的代码。

```go handlers.go
package main

import (
    "encoding/json"
    "fmt"
    "net/http"

    "github.com/gorilla/mux"
)

func Index(w http.ResponseWriter, r *http.Request) {
    fmt.Fprintln(w, "Welcome!")
}

func TodoIndex(w http.ResponseWriter, r *http.Request) {
    todos := Todos{
        Todo{Name: "Write presentation"},
        Todo{Name: "Host meetup"},
    }

    if err := json.NewEncoder(w).Encode(todos); err != nil {
        panic(err)
    }
}

func TodoShow(w http.ResponseWriter, r *http.Request) {
    vars := mux.Vars(r)
    todoId := vars["todoId"]
    fmt.Fprintln(w, "Todo show:", todoId)
}
```

```go routes.go
package main

import (
    "net/http"

    "github.com/gorilla/mux"
)

type Route struct {
    Name        string
    Method      string
    Pattern     string
    HandlerFunc http.HandlerFunc
}

type Routes []Route

func NewRouter() *mux.Router {

    router := mux.NewRouter().StrictSlash(true)
    for _, route := range routes {
        router.
            Methods(route.Method).
            Path(route.Pattern).
            Name(route.Name).
            Handler(route.HandlerFunc)
    }

    return router
}

var routes = Routes{
    Route{
        "Index",
        "GET",
        "/",
        Index,
    },
    Route{
        "TodoIndex",
        "GET",
        "/todos",
        TodoIndex,
    },
    Route{
        "TodoShow",
        "GET",
        "/todos/{todoId}",
        TodoShow,
    },
}
```

```go todo.go
package main

import "time"

type Todo struct {
    Name      string    `json:"name"`
    Completed bool      `json:"completed"`
    Due       time.Time `json:"due"`
}

type Todos []Todo
```

```go main.go
package main

import (
    "log"
    "net/http"
)

func main() {

    router := NewRouter()

    log.Fatal(http.ListenAndServe(":8080", router))
}
```

## 更好的Routing
我们重构的过程中，我们创建了一个更多功能的routes文件。 这个新文件利用了一个包含多个关于路由信息的结构体。 特别的，我们可以指定请求的类型， 例如GET, POST, DELETE等等。

## 输出Web日志
在分割路由文件中，我也包含有一个不可告人的动机。 稍后你就会看到， 现在很容易使用另外的函数来修饰我的http处理器。

首先让我们能对web请求输出日志的能力，就像很多流行web服务器那样的。 在Go语言中，标准库里边没有web日志包或功能, 因此我们需要自己创建。
```go logger.go
package logger

import (
	"log"
	"net/http"
	"time"
)

func Logger(inner http.Handler, name string) http.Handler {
	return http.HandlerFunc(func(w http.ResponseWriter, r *http.Request) {
		start := time.Now()
		inner.ServeHTTP(w, r)

		log.Printf(
			"%s\t%s\t%s\t%s",
			r.Method,
			r.RequestURI,
			name,
			time.Since(start),
		)
	})
}
```

上面我们定义了一个Logger函数，可以给handler进行包装修饰。

这是Go语言中非常标准的惯用方式。非常有效，我们只需要将Handler传入该函数， 然后它会将传入的handler包装一下，添加web日志和耗时统计功能。

## 应用Logger修饰器
要应用Logger修饰符， 我们可以创建router， 我们只需要简单的将我们所有的当前路由都包到其中， NewRouter函数修改如下:
```go
func NewRouter() *mux.Router {

    router := mux.NewRouter().StrictSlash(true)
    for _, route := range routes {
        var handler http.Handler

        handler = route.HandlerFunc
        handler = Logger(handler, route.Name)

        router.
            Methods(route.Method).
            Path(route.Pattern).
            Name(route.Name).
            Handler(handler)
    }

    return router
}
```

现在再次运行我们的程序，我们就可以看到日志大概如下:
```
2014/11/19 12:41:39 GET /todos  TodoIndex       148.324us
```

## 这个路由文件太疯狂...让我们重构它吧
路由routes文件现在已经变得稍微大了些， 下面我们将它分解成多个文件:
- routes.go
- router.go

```go routes.go
package main

import "net/http"

type Route struct {
    Name        string
    Method      string
    Pattern     string
    HandlerFunc http.HandlerFunc
}

type Routes []Route

var routes = Routes{
    Route{
        "Index",
        "GET",
        "/",
        Index,
    },
    Route{
        "TodoIndex",
        "GET",
        "/todos",
        TodoIndex,
    },
    Route{
        "TodoShow",
        "GET",
        "/todos/{todoId}",
        TodoShow,
    },
}
```

```go router.go
package main

import (
    "net/http"

    "github.com/gorilla/mux"
)

func NewRouter() *mux.Router {
    router := mux.NewRouter().StrictSlash(true)
    for _, route := range routes {
        var handler http.Handler
        handler = route.HandlerFunc
        handler = Logger(handler, route.Name)

        router.
            Methods(route.Method).
            Path(route.Pattern).
            Name(route.Name).
            Handler(handler)

    }
    return router
}
```

## 承担一定的责任
到目前为止，我们已经有了一些相当好的样板代码(boilerplate), 是时候重新审视我们的处理器了。我们需要稍微多的责任。 首先修改TodoIndex，添加下面两行代码:
```go
func TodoIndex(w http.ResponseWriter, r *http.Request) {
    todos := Todos{
        Todo{Name: "Write presentation"},
        Todo{Name: "Host meetup"},
    }
    w.Header().Set("Content-Type", "application/json; charset=UTF-8")
    w.WriteHeader(http.StatusOK)
    if err := json.NewEncoder(w).Encode(todos); err != nil {
        panic(err)
    }
}
```
这里发生了两件事。 首先，我们设置了响应类型并告诉客户端期望接受JSON。第二，我们明确的设置了响应状态码。

Go语言的net/http服务器会尝试为我们猜测输出内容类型(然而并不是每次都准确的), 但是既然我们已经确切的知道响应类型，我们总是应该自己设置它。

## 稍等片刻，我们的数据库在哪里?
很明显，如果我们要创建RESTful API, 我们需要一些用于存储和检索数据的地方。然而，这个是不是本文的范围之内， 因此我们将简单的创建一个非常简陋的模拟数据库(非线程安全的)。

我们创建一个repo.go文件，内容如下：
```go repo.go
package main

import "fmt"

var currentId int

var todos Todos

// Give us some seed data
func init() {
    RepoCreateTodo(Todo{Name: "Write presentation"})
    RepoCreateTodo(Todo{Name: "Host meetup"})
}

func RepoFindTodo(id int) Todo {
    for _, t := range todos {
        if t.Id == id {
            return t
        }
    }
    // return empty Todo if not found
    return Todo{}
}

func RepoCreateTodo(t Todo) Todo {
    currentId += 1
    t.Id = currentId
    todos = append(todos, t)
    return t
}
func RepoDestroyTodo(id int) error {
    for i, t := range todos {
        if t.Id == id {
            todos = append(todos[:i], todos[i+1:]...)
            return nil
        }
    }
    return fmt.Errorf("Could not find Todo with id of %d to delete", id)
}
```

## 给Todo添加ID
我们创建了模拟数据库，我们使用并赋予id, 因此我们相应的也需要更新我们的Todo结构体。
```go 
package main

import "time"

type Todo struct {
    Id        int       `json:"id"`
    Name      string    `json:"name"`
    Completed bool      `json:"completed"`
    Due       time.Time `json:"due"`
}

type Todos []Todo
```

## 更新我们的TodoIndex
要使用数据库，我们需要在TodoIndex中检索数据。修改代码如下:
```go
func TodoIndex(w http.ResponseWriter, r *http.Request) {
    w.Header().Set("Content-Type", "application/json; charset=UTF-8")
    w.WriteHeader(http.StatusOK)
    if err := json.NewEncoder(w).Encode(todos); err != nil {
        panic(err)
    }
}
```
## POST JSON
到目前为止，我们只是输出JSON, 现在是时候进入存储一些JSON了。

在routes.go文件中添加如下路由：
```go
Route{
    "TodoCreate",
    "POST",
    "/todos",
    TodoCreate,
},
```

## Create路由
```go
func TodoCreate(w http.ResponseWriter, r *http.Request) {
    var todo Todo
    body, err := ioutil.ReadAll(io.LimitReader(r.Body, 1048576))
    if err != nil {
        panic(err)
    }
    if err := r.Body.Close(); err != nil {
        panic(err)
    }
    if err := json.Unmarshal(body, &todo); err != nil {
        w.Header().Set("Content-Type", "application/json; charset=UTF-8")
        w.WriteHeader(422) // unprocessable entity
        if err := json.NewEncoder(w).Encode(err); err != nil {
            panic(err)
        }
    }

    t := RepoCreateTodo(todo)
    w.Header().Set("Content-Type", "application/json; charset=UTF-8")
    w.WriteHeader(http.StatusCreated)
    if err := json.NewEncoder(w).Encode(t); err != nil {
        panic(err)
    }
}
```
首先我们打开请求的body。 注意我们使用io.LimitReader。这样是保护服务器免受恶意攻击的好方法。假设如果有人想要给你服务器发送500GB的JSON怎么办？

我们读取body以后，我们解构Todo结构体。 如果失败，我们作出正确的响应，使用恰当的响应码， 422， 但是我们依然使用json响应回去。 这样可以允许客户端理解有错发生了， 而且有办法知道到底发生了什么错误。

最后，如果所有都通过了，我们就响应201状态码，表示请求创建的实体已经成功创建了。 我们同样还是响应回代表我们创建的实体的json， 它会包含一个id, 客户端可能接下来需要用到它。

## POST一些JSON
我们现在有了伪repo, 也有了create路由，那么我们需要post一些数据。 我们使用curl通过下面的命令来达到这个目的:
```go
curl -H "Content-Type: application/json" -d '{"name": "New Todo"}' http://localhost:8080/todos
```

如果你再次通过http://localhost:8080/todos访问，大概会得到下面的响应:
```json
[
    {
        "id": 1,
        "name": "Write presentation",
        "completed": false,
        "due": "0001-01-01T00:00:00Z"
    },
    {
        "id": 2,
        "name": "Host meetup",
        "completed": false,
        "due": "0001-01-01T00:00:00Z"
    },
    {
        "id": 3,
        "name": "New Todo",
        "completed": false,
        "due": "0001-01-01T00:00:00Z"
    }
]
```

## 我们还没有做的事情
虽然我们已经有了很好的开端，但是还有很多事情没有做：
- 版本控制: 如果我们需要修改API, 结果完全改变了怎么办? 可能我们需要在我们的路由开头加上/v1/prefix?
- 授权: 除非这些都是公开/免费API, 我们可能还需要授权。 建议学习JSON web tokens的东西。

eTag - 如果你正在构建一些需要扩展的东西，你可能需要实现eTag。

## 还有什么?
对于所有项目来说，开始都很小，但是很快就变得失控了。但是如果我们想要将它带到另外一个层次， 让他生产就绪， 还有一些额外的事情需要做:
- 大量重构(refactoring).
- 为这些文件创建几个包，例如一些JSON助手、修饰符、处理器等等。
- 测试， 使得，你不能忘记这点。这里我们没有做任何测试。对于生产系统来说，测试是必须的。

## 源代码
https://github.com/corylanou/tns-restful-json-api

## 总结
对我来说，最重要的，需要记住的是我们要建立一个负责任的API。 发送适当的状态码，header等，这些是API广泛采用的关键。我希望本文能让你尽快开始自己的API。
## 参考链接
- [Go语言RESTful JSON API实现](https://thenewstack.io/make-a-restful-json-api-go/)
- [JSON API](http://jsonapi.org)
- [Gorilla Web Toolkit](http://www.gorillatoolkit.org/)
- [httprouter](https://github.com/julienschmidt/httprouter)
- [JSON Web Tokens](http://jwt.io/)
- [eTag](http://en.wikipedia.org/wiki/HTTP_ETag)
