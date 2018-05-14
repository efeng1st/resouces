使用Go语言搭建微服务
==================
下面这些文章章节中，我们会使用Go编程语言来构建微服务， 我们一片一片的将必要集成添加进来，让它们能在Spring Cloud/Netflix OSS中的集群模式的Docker上良好运行。

如果你还不知道微服务是什么， 我们建议先读读Martin Fowler写的关于微服务的文章(见参考连接)。 关于更多微服务的操作模型，也可参考微服务操作模型(参考连接部分), 里边很好的介绍了一些关键概念。

这一系列文章不是Go语言初学者的编程指南，尽管我们会在文章中写一些代码，还会解释一些Go语言的关键概念。 我们会看到很多代码，尤其是第一部分，这里涵盖单元测试和其他核心主题的基本功能。

第一部分将介绍关键概念和探索基于Go语言的微服务实现可能性的原因。

> 本文中提到"Docker Swarm", 我的意思是以swarm模型运行Docker 1.12或以上版本。在Docker 1.12发布版本后终止了“Docker Swarm”的独立概念。

## 目录

- [微服务介绍](intro.md)
- [Go微服务理论基础介绍](1.md)
- [构建第一个Go微服务](2.md)
- [嵌入数据存储并提供JSON服务](3.md)
- [使用GoConvey对HTTP服务进行单元测试](4.md)
- [在Docker Swarm上发布](5.md)
- [添加健康检查](6.md)
- [服务发现和负载均衡](7.md)
- [使用Spring Clound配置和Viper进行集中式配置](8.md)
- [AMQP消息队列](9.md)
- [使用Logrus和Docker的日志驱动器在Laas上记录日志](10.md)
- [使用Netflix Hystrix实现断路器和快速恢复能力](11.md)
- [使用Zipkin进行分布式跟踪](12.md)
- [使用CockroachDB和GORM进行分布式持久化](13.md)

## 参考连接
- [微服务](https://martinfowler.com/articles/microservices.html#Battle-testedStandardsAndEnforcedStandards): Fowler的微服务概念介绍的文章。
- [微服务操作模型](http://callistaenterprise.se/blogg/teknik/2015/03/25/an-operations-model-for-microservices/): 微服务操作模型。
- https://ewanvalentine.io/microservices-in-golang-part-1/
- http://callistaenterprise.se/blogg/teknik/2017/02/17/go-blog-series-part1/
- https://ewanvalentine.io/tag/go/
