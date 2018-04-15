《GO实战》
======================

## 序言

在计算机科学中，当你联想杰出人员时，脑海中会出现一些人。在他们之中有Rob Pike, Robert Griesmier, 以及Ken Thompson, 他们都是负责Unix，Plan 9, B, Java的JVM Hotspot, V8, Strongtalk, Sawzall, Ed, Acme,以及UTF8， 还有其他很多作品。 2007年，他们聚集到一起，用一个非常强大的想法做了个试验，结合他们十多年的经验，创建一种新的系统语言，这个语言由现有语言启发，但是与任何语言都不同。他们发布了他们的创作，开放源码并将其命名为"Go". 如果Go语言继续下去，它必将被证明成为他们显著作品中最具影响力的作品之一。

当人们聚集在一起，纯属为了把世界变得更美好，那么这些人就处于最佳状态。2013年，Brian和Erik成立了地鼠学院(Gopher Academy), 不久后Bill和其他一些类似的志同道合的人加入进来， 团结一致的围绕Go语言构建一个更好的社区。他们第一次注意到社区需要需要一个地方来在线搜集和共享资料, 因此他们搭建了Go讨论版块(slack)和地鼠学院博客。随着时间迁移,社区不断发展壮大，它们确立了世界上第一个全局的Go发布会(Conference) - GopherCon。通过社区深入了解，它们了解到需要一个资源来指导成千上万的程序员进入这个语言，因此它们开始写这本你正在读的书。

这本书是三个在Go社区贡献了很多时间和才能的人爱的贡献。 我见证了Bill, Brian, Erik在过去一年中写作、修正内容的过程，同时他们还维护现有的地鼠学院博客的编辑、会议组织者，在他们日常工作、以及作为父亲和丈夫角色的时间里。对于他们来说，这不是一本书，而是对他们喜欢语言的一种致敬。他们不满足于制作一本好书。他们每页、每个例子、每章内容都不断写作、检查、重写、修正很多次，直到他们出了和他们珍视语言匹配的该书为止。

离开一种舒适、熟悉的语言去尝试一种不仅对你自己新而且对全世界都新的语言来说是需要很大的勇气的。这是一条崎岖不平的道路，布满了只有早期试验者熟悉的缺陷。包括意外错误、参差不齐或缺少文档，以及缺少使用的已建库等。这是开拓者、先锋的道路。如果你正在读这里， 你很可能就是在旅途的开始。

从第一章到最后一章，本书为读者提供简明、全面的探索、学习和使用Go语言的指导。 全世界来说，你不可能有比Bill, Brian, Erik更好的指导了。为你发现Go的美好，期待在网上、Go的聚会和会议上看到你。

----- STEVE FRANCIA 
      GOPHER AND CREATOR OF HUGO,
      COBRA, VIPER, AND SPF13-VIM

## 前言
2013年十月，在写完GoingGo.net博客几个月，我接到Brian Ketelsen和Erik St. Martin的电话。他们正准备写这本书，问我是否愿意加入其中。我就抓住机会加入其中。当时我对Go语言还是新手，因此这是我很好的学习Go语言的机会，和Brian, Erik工作，分享比博客所学更多的知识。

在我们完成前面四章，我们在MEAP下面发布了该书。不久之后，我们收到一个来自该语言组成员的邮件。该人提供了审核，包含了详细的修改、加上丰富的知识、建议、鼓励和支持。从那开始，我们决定重新重写第二章, 并对第四章进行重大修改。我们了解到重写章节不会是例外，而是常态。这一经历也告诉我们，要让世界各界都帮我们写这本书，我们必须立即做到这一点。

从那时起，该书已经成为社区努力的结果。我们试图在研究每章放适量的时间，开发代码样例，以及和社区一起检查、讨论以及编辑材料和代码。我们尽力确保本书技术的正确性，只展示符合习惯的代码，教你社区感觉舒服方式书写和考虑的Go语言。我们的确也有我们自己的想法、实践和指导方针。

我们希望本书能帮你学习Go，以及你能查找现在或多年以后还有用的资源。Brian, Erik和我经常在线，能帮助任何能找到我们的人。如果你购买此书，感谢您，不要害羞说"hi".

----- WILLIAM KENNEDY

## 致谢(acknowledgments)

我们花了18个月来写这本书，但是如果没有很多人的支持我们的付出努力是不可能的，我们的家庭、朋友、同事和导师；整个Go社区；我们的出版商，Manning。

当您正在写类似这样的书的时候，你需要一个编辑，他不仅能分享你的有点，而且能不惜一切代价帮你度过困境。Jennifer Stout，你是个聪明的，有教养的，了不起的朋友。感谢你所做的一切，感谢你在我们最需要你的时候陪伴着你。谢谢你让这本书成为现实。也感谢所有在本书的开发和制作过程中与我们一起工作的曼宁的同事们。

你不可能无所不知，因此需要社区的人贡献他们的时间和知识。我们感谢Go社区以及在各个阶段中所有参与评论和提供反馈的手稿, 特别是Adam McKay, Alex Basile, Alex Jacinto, Alex Vidal, Anjan Bacchu, Benoît Benedetti, Bill Katz, Brian Hetro, Colin Kennedy, Doug Sparling, Jeffrey Lim, Jesse Evans, Kevin Jackson, Mark Fisher, Matt Zulak, Paulo Pires, Peter Krey, Philipp K. Janert, Sam Zaydel, 以及Thomas O’Rourke。也非常感谢Jimmy Frasché在即将上线时候的细心技术评论手稿。

还有一些人需要特别鸣谢。

Kim Shrier是最早开始提供评论，并给予时间去教。我们从你身上学到很多东西，我们非常感激。这本书在技术上更好都是因为你。

Bill Hathaway在写这本书的最后一年投入了大量的精力，塑造了每一章；他的思想和见解是无价的。我们必须给Bill信用卡作为第九章的合著者。如果没有Bill的时间、才干和努力，它就不会存在。

我们还要感谢Cory Jacobson, Jeffery Lim, Chetan Conikee和Nan xiao, 他们一直提供评论、意见和指导。感谢Gabriel Aszalos, Fatih Arslan, Kevin Gillette和Jason Waldrip帮助示例代码和评论。还要特别感谢Steve Francia对序言的贡献以及对我们工作的支持。

最后我们还要感谢我们的家人和朋友。任何承担这种承诺和时间的事情都会对所爱的人产生影响。

    WILLIAM KENNEDY
    感谢Lisa, 我漂亮的妻子和我五个孩子:Brianna, Melissa, Amanda, Jarrod, 和Thomas。Lisa, 我知道你和孩子们度过了太多的没丈夫和父亲的日日夜夜。感谢你让我花了所有时间来写这本书: 我爱你们。

    我还要感谢我的生意伙伴Ed Gonzalez, 创意总监Erick Zelaya, 以及Ardan工作室的所有成员。Ed，感谢你一开始就支持我。没有你我是做不到的。你不仅仅是一个商业伙伴，你还是一个朋友和兄弟: 谢谢你。Erick, 感谢你支持我和公司所做的一切。不知道没有你我们会怎样。

    BRIAN KETELSEN
    我要感谢我的家庭，对他们出这本书长达四年的耐心: Christine, Nathan, Lauren, 和Evelyn: thank you for putting up with me as I wrote chapters in a lounge chair by the pool while you were swimming.
    Thank you for believing that this book could and would be published.

    ERIK ST. MARTIN

    I would like to thank my fiancée Abby, and my three children Halie, Wyatt, and Allie for being so patient and understanding how much time writing a book and organizing conferences demand. I love you all so very much and am lucky to have you.

    I would also like to thank Bill Kennedy for the tremendous effort he has poured into this book—we asked him to help us write it, and he steered the ship most of the way due to the demands of our jobs and organizing GopherCon. I also want to thank the community for all their reviews and words of encouragement.

## 关于本书
Go是一个开源程序语言，它能很容易构建简单、可信赖、有效的软件。虽然它是从其他语言借用的想法，但是它也有自己的独特性和简单性，使得Go程序和其他语言写的程序不同。它权衡低级语言和现今语言的一些高级特性。这创造了一个编程环境，可以提升生产效率，性能，以及控制能力。在Go中，写更少的代码，实现更多的功能。

### 谁应该读本书?
该书是写给中级程序员，他们有一些其他编程语言的经验并且希望学习Go语言的。我们写本书的目标是提供一个密集的、全面的、地道的语言视图。我们聚焦语言的规范和实现，包括从语言语法，Go的类型系统、并发、通道、测试等多个主题。我们相信本书对那些想要跳转学习Go语言以及那些想要透彻理解语言和它内部机制的人都是完美的。

### Roadmap(路标)

本书由九个章节组成，简略描述如下:

- 第一章: 快速介绍Go语言是什么，为什么要创建它，以及它能解决的问题。也简单的介绍了一些Go的核心概念，例如并发。
- 第二章: 带你完成一个完整的Go程序，顺便教你Go作为一个语言需要提供的所有东西。
- 第三章: 介绍包的概念，以及如何设置Go工作空间和开发环境。也展示了如何使用Go工具，包括获取和构建代码。
- 第四章: 提供了go内建数据类型的详情: 数组, 分片和map。介绍了这些数据结构后面的实现和机制。
- 第五章: 详细介绍了God的类型系统，从结构体类型到命名类型到接口到类型嵌入。也涵盖了如何以简单的方式利用这些一起构建实现复杂的软件。
- 第六章: 深入探讨Go调度器、并发、通道如何工作。它教会了这方面语言背后的机制。
- 第七章: 带着第六章学习到的东西，展示更多并发模式的实际代码。这里将学到如何实现goroutine池来管理work，以及如何复用池来实现资源共享。
- 第八章: 探讨标准库，深入三个包: log, json和io。本章重点讨论了这三个包的复杂型。
- 第九章: 通过展示如何使用测试和基准测试框架来结束本书。你将学到如何书写单元测试和基准测试，以及如何在文档中添加例子，以及如何使用例子来测试。

### 关于本书源码
- [曼宁网站](https://www.manning.com/books/go-in-action)
- [github](https:// github.com/goinaction/code)

### 作者在线

Purchase of Go in Action includes free access to a private web forum run by Manning Publications where you can make comments about the book, ask technical questions, and receive help from the authors and from other users. To access the forum and subscribe to it, point your web browser to www.manning.com/books/go-in-action. This page provides information on how to get on the forum once you’re registered, what kind of help is available, and the rules of conduct on the forum.
 Licensed to Mark Watson <nordickan@gmail.com>
 xviii
 ABOUT THIS BOOK
 Manning’s commitment to our readers is to provide a venue where a meaningful dialog between individual readers and between readers and the authors can take place. It is not a commitment to any specific amount of participation on the part of the authors, whose contributions to the AO remain voluntary (and unpaid). We suggest you ask the authors challenging questions, lest their interest stray.
 The Author Online forum and the archives of previous discussions will be accessible from the publisher’s website as long as the book is in print.
 
#### 关于作者
WILLIAM KENNEDY (@goinggodotnet) is a managing partner at Ardan Studio in Miami, Florida, a mobile, web, and systems development company. He is also the author of the blog GoingGo.Net, and the organizer for the Go meetup in Miami. Bill is focused on Go education through his training company, Ardan Labs. He can often be found talking at conferences and giving workshops both locally and over hangouts. He always finds time to work with individuals and groups who want to take their Go knowledge, blogging, and coding skills to the next level.
BRIAN KETELSEN (@bketelsen) is the CIO and cofounder of XOR Data Exchange. Brian is a co-organizer of GopherCon, the annual Go conference, and the founder of GopherAcademy—a community-focused organization created for the promotion of the Go language and the education of Go developers. He’s been using Go since 2010.
ERIK ST. MARTIN (@erikstmartin) is the Director of Software Development at XOR Data Exchange, a big data and analytics company located in Austin, Texas, but resides in Tampa, Florida. Erik is a long-time contributor to open source and its communities. He’s an organizer for GopherCon, an annual Go conference, and the organizer of the Go Tampa meetup group. He’s very passionate about Go and the community and eager to find new ways to foster its growth.

## 关于封面插图
本书封面插图标题是"东印度群岛的人"。改图从Thomas Jefferys1757年到1772年出版的伦敦不同民族古代和现代服饰合集(四卷)中的取的。

The title page states that these are hand-colored copperplate engravings, heightened with gum arabic. Thomas Jefferys (1719–1771) was called “Geographer to King George III.” He was an English cartographer who was the leading map supplier of his day. He engraved and printed maps for government and other official bodies and produced a wide range of commercial maps and atlases, especially of North America. His work as a map maker sparked an interest in local dress customs of the lands he surveyed and mapped, and which are brilliantly displayed in this collection.
Fascination with faraway lands and travel for pleasure were relatively new phenomena in the late eighteenth century, and collections such as this one were popular, introducing both the tourist as well as the armchair traveler to the inhabitants of other countries. The diversity of the drawings in Jefferys’ volumes speaks vividly of the uniqueness and individuality of the world’s nations some 200 years ago. Dress codes have changed since then, and the diversity by region and country, so rich at the time, has faded away. It is now often hard to tell the inhabitants of one continent from another. Perhaps, trying to view it optimistically, we have traded a cultural and visual diversity for a more varied personal life—or a more varied and interesting intellectual and technical life.
At a time when it is hard to tell one computer book from another, Manning celebrates the inventiveness and initiative of the computer business with book covers based on the rich diversity of regional life of two centuries ago, brought back to life by Jeffreys’ pictures.

## 章节目录

- 第一章: [Go语言介绍](chapters/1.1.md)
    - 1.1 [使用Go来解决现代编程挑战](chapters/1.1.md)
    - 1.2 Hello, Go
    - 1.3 Summary
- 第二章: [Go语言快速入门](chapters/2.1.md)
    - 2.1 Program architecture
    - 2.2 Main package
    - 2.3 Search package
    - 2.4 RSS matcher
    - 2.5 Summary
- 第三章: 包和工具
    - 3.1 Packages
    - 3.2 Imports
    - 3.3 init
    - 3.4 Using Go tools
    - 3.5 Going farther with Go developer tools
    - 3.6 Collaborating with other Go developers
    - 3.7 Dependency management
    - 3.8 Summary
- 第四章: 数组、分片及映射(array, slice, map)
    - 4.1 Array internals and fundamentals
    - 4.2 Slice internals and fundamentals
    - 4.3 Map internals and fundamentals
    - Summary
- 第五章: Go语言类型系统
    - 5.1 User-defined types
    - 5.2 Methods
    - 5.3 The nature of types
    - 5.4 Interfaces
    - 5.5 Type embedding
    - 5.6 Exporting and unexporting identifiers
    - 5.7 Summary
- 第六章: 并发
    - 6.1 Concurrency versus parallelism
    - 6.2 Goroutines
    - 6.3 Race conditions
    - 6.4 Locking shared resources
    - 6.5 Channels
    - 6.6 Summary
- 第七章: 并发模式
    - 7.1 Runner
    - 7.2 Pooling
    - 7.3 Work
    - 7.4 Summary
- 第八章: 标准库
    - 8.1 Documentation and source code
    - 8.2 Logging
    - 8.3 Encoding/Decoding
    - 8.4 Input and output
    - 8.5 Summary
- 第九章: 测试和基准测试
    - 9.1 Unit testing
    - 9.2 Examples
    - 9.3 Benchmarking
    - 9.4 Summary

## 链接
1. [地鼠学院博客](https://www.gopheracademy.com/)
2. [GopherCon Github](https://github.com/gophercon)
3. [Hugo](https://gohugo.io/): 世界最快的构建网站的框架。
4. [GoingGo博客](https://www.ardanlabs.com/blog/)
5. [slack](https://www.slack.com)
6. [MEAP曼宁早期访问计划](https://www.manning.com/meap-program)
7. [曼宁网站](https://www.manning.com/books/go-in-action)
8. [go in action code github](https:// github.com/goinaction/code)
9. [Go Playground](http://play.golang.org)
10. [Go资源搜索](http://go-search.org/)
