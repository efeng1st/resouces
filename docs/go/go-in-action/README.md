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
We have spent over 18 months writing this book, but none of our efforts would have been possible without the support of many people—our families, friends, colleagues, and mentors; the entire Go community; and our publisher, Manning.
When you’re writing a book like this, you need an editor who will not only share the good but help you through the bad and be there for you at all cost. Jennifer Stout, you’re a brilliant, nurturing, and amazing friend. Thank you for everything and for being there when we needed you the most. Thank you for making this book a reality. Thanks also to all the other folks at Manning who worked with us during the develop- ment and production of our book.
You can’t know everything, so it requires a community of people to give their time and knowledge. We thank the Go community and everyone who participated in reviews and provided feedback on the manuscript at various stages of its development, especially Adam McKay, Alex Basile, Alex Jacinto, Alex Vidal, Anjan Bacchu, Benoît Benedetti, Bill Katz, Brian Hetro, Colin Kennedy, Doug Sparling, Jeffrey Lim, Jesse Evans, Kevin Jackson, Mark Fisher, Matt Zulak, Paulo Pires, Peter Krey, Philipp K. Janert, Sam Zaydel, and Thomas O’Rourke. Thanks also to Jimmy Frasché for his care- ful technical review of the final manuscript shortly before it went into production.
There are a few other people who need to be acknowledged in particular.
Kim Shrier was there from the very beginning, providing reviews, and giving of his time to teach. We learned so many things from you and we are grateful. The book is better technically because of you.
Bill Hathaway got involved heavily in the last year of writing the book, shaping each chapter; his thoughts and opinions were invaluable. We must give Bill credit as a coauthor of chapter 9. It would not exist without Bill’s time, talent, and effort.

We would also like to recognize Cory Jacobson, Jeffery Lim, Chetan Conikee, and Nan Xiao, who consistently provided time for reviews, opinions, and guidance. Thanks to Gabriel Aszalos, Fatih Arslan, Kevin Gillette, and Jason Waldrip for help with sample code and reviews. And special thanks to Steve Francia for contributing the foreword and endorsing our work.
We end by sincerely thanking our families and friends. Anything that takes this level of commitment and time always has an effect on the ones you love.
WILLIAM KENNEDY
I would like to thank Lisa, my beautiful wife, and my five children: Brianna, Melissa, Amanda, Jarrod, and Thomas. Lisa, I know you and the kids spent way too many days, nights, and weekends without your husband and father. Thank you for letting me take all the time I needed to work on the book: I love each and every one of you.
I would also like to thank my business partner Ed Gonzalez, creative director Erick Zelaya, and the entire team at Ardan Studios. Ed, thanks for supporting me from the beginning. I could not have done this without you. You are more than just a business partner, you are a friend and brother: thank you. Erick, thanks for everything you do to support me and the company. Not sure what we would do without you.
BRIAN KETELSEN
I would like to thank my family for their patience during this four-year-long process of producing a book. Christine, Nathan, Lauren, and Evelyn: thank you for putting up with me as I wrote chapters in a lounge chair by the pool while you were swimming.
Thank you for believing that this book could and would be published.
ERIK ST. MARTIN
I would like to thank my fiancée Abby, and my three children Halie, Wyatt, and Allie for being so patient and understanding how much time writing a book and organizing conferences demand. I love you all so very much and am lucky to have you.
I would also like to thank Bill Kennedy for the tremendous effort he has poured into this book—we asked him to help us write it, and he steered the ship most of the way due to the demands of our jobs and organizing GopherCon. I also want to thank the community for all their reviews and words of encouragement.

## 关于本书
Go is an open source programming language that makes it easy to build simple, reli- able, and efficient software. Although it borrows ideas from existing languages, it has a unique and simple nature that makes Go programs different in character from pro- grams written in other languages. It balances the capabilities of a low-level systems lan- guage with some high-level features you see in modern languages today. This creates a programming environment that allows you to be incredibly productive, performant, and fully in control; in Go, you can write less code and do so much more.
### Who should read this book?
This book was written for an intermediate-level developer who has some experience with other programming languages and wants to learn Go. Our goal in writing this book is to provide you an intensive, comprehensive, and idiomatic view of the lan- guage. We focus on both the specification and implementation of the language, including topics that range from language syntax, Go’s type system, concurrency, channels, testing, and more. We believe this book is perfect for anyone who wants a jump-start in learning Go as well as for those who want a more thorough understand- ing of the language and its internals.
Roadmap
The book consists of nine chapters, briefly described here:
- Chapter 1 is a quick introduction to what the language is, why it was created, and the problems it solves. It also briefly introduces some of Go’s core concepts such as concurrency.
- Chapter 2 walks you through a complete Go program, teaching you all that Go has to offer as a language along the way.
- Chapter 3 introduces the concept of packaging and how to best set up your Go workspace and development environment. It also shows how to use the Go tool- ing, including fetching and building your code.
- Chapter 4 provides a detailed view of Go’s built-in data types: arrays, slices, and maps. It explains the implementation and mechanics behind these data structures.
- Chapter 5 is a detailed view of Go’s type system, from struct types to named types to interfaces and type embedding. It also covers how all these things come together to allow you to structure and write complex software in a simpler way.
- Chapter 6 dives deeply into how the Go scheduler, concurrency, and channels work. It teaches the mechanics behind this aspect of the language.
- Chapter 7 takes what you learn from chapter 6 and shows more practical code around concurrency patterns. You will learn how to implement goroutine pools to manage work and how to pool reusable resources to be shared.
- Chapter 8 explores the standard library and goes deep into three packages: log, json, and io. The chapter focuses on some of the intricacies of these three packages.
- Chapter 9 closes the book by showing how to use the testing and benchmarking framework. You will learn how to write unit and table tests and benchmarks, and how to add examples to your documentation and use the examples as tests.

### 关于本书源码
All source code in the book is presented in amono-spacedtypefacelikethis, which sets it off from the surrounding text. In many listings, the code is annotated to point out key concepts, and numbered bullets are sometimes used in the text to provide additional information about the code.
Source code for the examples in the book is available for download from the pub- lisher’s website at www.manning.com/books/go-in-action and from GitHub at https:// github.com/goinaction/code.

### Author Online
Purchase of Go in Action includes free access to a private web forum run by Manning Publications where you can make comments about the book, ask technical questions, and receive help from the authors and from other users. To access the forum and sub- scribe to it, point your web browser to www.manning.com/books/go-in-action. This page provides information on how to get on the forum once you’re registered, what kind of help is available, and the rules of conduct on the forum.
 Licensed to Mark Watson <nordickan@gmail.com>
 xviii
 ABOUT THIS BOOK
 Manning’s commitment to our readers is to provide a venue where a meaningful dialog between individual readers and between readers and the authors can take place. It is not a commitment to any specific amount of participation on the part of the authors, whose contributions to the AO remain voluntary (and unpaid). We sug- gest you ask the authors challenging questions, lest their interest stray.
 The Author Online forum and the archives of previous discussions will be accessi- ble from the publisher’s website as long as the book is in print.
 
#### 关于作者
WILLIAM KENNEDY (@goinggodotnet) is a managing partner at Ardan Studio in Miami, Florida, a mobile, web, and systems development company. He is also the author of the blog GoingGo.Net, and the organizer for the Go meetup in Miami. Bill is focused on Go education through his training company, Ardan Labs. He can often be found talking at conferences and giving workshops both locally and over hangouts. He always finds time to work with individuals and groups who want to take their Go knowledge, blogging, and coding skills to the next level.
BRIAN KETELSEN (@bketelsen) is the CIO and cofounder of XOR Data Exchange. Brian is a co-organizer of GopherCon, the annual Go conference, and the founder of GopherAcademy—a community-focused organization created for the promotion of the Go language and the education of Go developers. He’s been using Go since 2010.
ERIK ST. MARTIN (@erikstmartin) is the Director of Software Development at XOR Data Exchange, a big data and analytics company located in Austin, Texas, but resides in Tampa, Florida. Erik is a long-time contributor to open source and its communities. He’s an organizer for GopherCon, an annual Go conference, and the organizer of the Go Tampa meetup group. He’s very passionate about Go and the community and eager to find new ways to foster its growth.

## 关于封面插图
The figure on the cover of Go in Action is captioned “Man from the East Indies.” The illustration is taken from Thomas Jefferys’ A Collection of the Dresses of Different Nations, Ancient and Modern (four volumes), London, published between 1757 and 1772. The title page states that these are hand-colored copperplate engravings, heightened with gum arabic. Thomas Jefferys (1719–1771) was called “Geographer to King George III.” He was an English cartographer who was the leading map supplier of his day. He engraved and printed maps for government and other official bodies and produced a wide range of commercial maps and atlases, especially of North America. His work as a map maker sparked an interest in local dress customs of the lands he surveyed and mapped, and which are brilliantly displayed in this collection.
Fascination with faraway lands and travel for pleasure were relatively new phenom- ena in the late eighteenth century, and collections such as this one were popular, introducing both the tourist as well as the armchair traveler to the inhabitants of other countries. The diversity of the drawings in Jefferys’ volumes speaks vividly of the uniqueness and individuality of the world’s nations some 200 years ago. Dress codes have changed since then, and the diversity by region and country, so rich at the time, has faded away. It is now often hard to tell the inhabitants of one continent from another. Perhaps, trying to view it optimistically, we have traded a cultural and visual diversity for a more varied personal life—or a more varied and interesting intellectual and technical life.
At a time when it is hard to tell one computer book from another, Manning cele- brates the inventiveness and initiative of the computer business with book covers based on the rich diversity of regional life of two centuries ago, brought back to life by Jeffreys’ pictures.

## 章节目录

- 第一章: Go语言介绍
    - 1.1 [使用Go来解决现代编程挑战](chapters/1.1.md)
    - 1.2 Hello, Go
    - 1.3 Summary
- 第二章: Go语言快速入门
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
