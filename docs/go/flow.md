基于流程的编程
==========================

## Get into the flow
在基于流程的编程中，程序在独立的进程单元间被建模为数据流。谁不会认为通道和goroutine就是自然界的类比呢?

微不足道的是所有软件都是关于数据处理的。 然而当你查看用传统编程语言所写的代码时，实际数据流不很容易看到。相反，你主要看到的是控制结构。真正的数据只在运行时出现，因为它们来自控制结构。

基于流程的编程将代码和数据的视图逆转。这里，你首先要看的是数据流；它是定义你的应用程序结构的主要原则。很多情况下都是在终端数据管道之间的小节点上进行数据处理。

```
                                               +----------+
                                   +----->-----|   节点2  |------>-------+
                                   |           +----------+              |
+-------------+    +-------+       |                                     |             +--------+
| data stream |--->| 节点1 |--->---|                                     +----->-------|  节点4 |
+-------------+    +-------+       |                                     |             +--------+
                                   |           +----------+              |
                                   +----->-----|   节点3  |------>-------+
                                               +----------+
```

在这个层面上来看，进程节点就是位于流程图中的方框内。实际的代码在这些方框之内。

### 基于流的编程和并发
看了FBP图我们立即会引发两个问题。

- 数据流模型是继承了并发的。 数据流互相之间是独立的，节点也是如此。 看起来就像关注点分离(separation of concerns)的优化体现。
- 数据流看起来与通道和goroutine异曲同工。

这里是否有自然的巧合? 看起来就是直接在Go的内置并发概念上构建的FBP模型。 实际上，这已经实现了。

### Go FBP类库
在github上快速搜索一下，就可以找到大量Go语言的FBP项目， 下面我把它们列到一起，并简单的描述下每个项目的介绍。

- [trustmaster/goflow](https://github.com/trustmaster/goflow): 这是一个相当简单的实现, 它是基于流程编程、并且几个目标为设计应用程序为组件图的Go编程语言中的其他并发模型。
