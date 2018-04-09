《GO实战》
======================

## 序言
In computer science, when you think of exceptional people, a few names come to mind. Among them are Rob Pike, Robert Griesmier, and Ken Thompson, who are responsible for UNIX, Plan 9, B, Java’s JVM HotSpot, V8, Strongtalk, Sawzall, Ed, Acme, and UTF8, among many other creations. In 2007, they came together to experi- ment with a very powerful idea, combining their decades of experience to create a new systems language inspired by existing languages but truly unlike anything that came before. They released their creation as open source and named it “Go.” If Go continues on the course it is now on, it may indeed prove to be the most impactful of their many notable creations.

Humanity is at its best when people join together with the pure intention of mak- ing the world a better place. In 2013, Brian and Erik formed the Gopher Academy and were soon joined by Bill and a few other similar-minded people, united in the pursuit of building a better community around the Go language. They first noticed that the community needed a place to gather and share material online so they set up the Go discussion board (slack) and the Gopher Academy blog. As time went on and the community continued to grow, they established the world’s first global Go confer- ence, GopherCon. Through their deep experience with the community, they knew that a resource was needed to guide the many thousands of programmers into this new language, so they began to write the book that you now hold in your hands.
This book is a labor of love from three individuals who have given so much of their time and talents to the Go community. I have been alongside Bill, Brian, and Erik to witness them writing and revising over the past year as they maintained their existing responsibilities as editors of the Gopher Academy blog, as conference organizers, in their day jobs, and in their roles as fathers and husbands. To them this is not a book, but a tribute to the language they love. They weren’t content with producing a “good” book. They wrote and reviewed, rewrote and revised many drafts of each page, exam- ple, and chapter until they had a book worthy of the language they hold so dear.
It takes courage to leave a language of comfort and familiarity and try a language that is not only new to you but new to the world. This road less traveled is a bumpy one, lined with bugs that only early adopters are familiar with. It includes unexpected errors, spotty or missing documentation, and a lack of established libraries to use. This is the path of a trailblazer, a pioneer. If you are reading this now, you are likely on the beginning of this journey.
From the first chapter to the last, this book is crafted to provide you, the reader, a concise and comprehensive guide to exploring, learning, and using Go. In all the world, you couldn’t hope to have better guides than Bill, Brian, and Erik. I’m excited for you to discover all the goodness that is Go and look forward to seeing you online and at the Go meetups and conferences.

----- STEVE FRANCIA GOPHER AND CREATOR OF HUGO, COBRA, VIPER, AND SPF13-VIM

## 前言
Back in October 2013 after writing the GoingGo.net blog for a few months, I received a call from Brian Ketelsen and Erik St. Martin. They were in the process of writing this book and asked if I would be a part of it. I jumped at the opportunity and started writ- ing. I was still very new to Go at the time, so this was a great chance to learn more about the language, work with Brian and Erik and share what I learned at a greater scale than the blog.
After we finished the first four chapters, we released the book under the Manning Early Access Program (MEAP). Soon after, we received an email from a member of the language team. This person provided a review that contained a detailed set of changes plus a wealth of knowledge, advice, encouragement, and support. From there, we decided to rewrite chapter 2 from scratch and performed a major overhaul of chapter 4. We learned that rewriting chapters was not going to be the exception but the norm. That experience also taught us that it was going to take the help of the community to write this book, and we needed to make that happen immediately.
Ever since then, this book has been a community effort. We have tried to put a proper amount of time in researching each chapter, developing code samples, and working with the community to review, discuss, and edit the material and code. We have done our best to make sure this book is technically correct, shows only idiomatic code, and teaches you Go the way the community feels it should be written and thought about. We do have some of our own thoughts, practices, and guidelines sprinkled in as well.
We hope this book helps you learn Go and you find it a useful resource today and for many years to come. Brian, Erik, and I are always online and available to help any- one who reaches out to us. If you purchased the book, thank you, and don’t be shy about saying “hi.”

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
