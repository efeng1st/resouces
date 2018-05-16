从容器到容器编排
===============

Docker平台以及周边生态系统包含很多工具来管理容器的生命周期。例如，Docker Command Line Interface(CLI)支持下面的容器活动:
- 从注册表中拉取仓库。
- 运行容器并可选的附加一个终端给它。
- 将容器提交新镜像。
- 上传镜像到注册中心。
- 终止运行中的容器。

CLI满足在单个主机上管理容器的需求，但是面对部署在多个主机上的容器时就无所适从了。为了超越单个容器管理，我们必须转向编排工具(业务流程工具)。

编排工具将生命周期管理能力扩展到部署在大量机器集群上部署的复杂的、多容器工作负载。

通过抽象主机基础结构， 编排工具允许用户将整个集群视为单个部署目标。

## 基本特征(Baseline Features) 
编排过程一般涉及应用管理所有方面自动化的工具，例如初始布局、调度和部署到稳态活动例如更新、部署，支持扩展和故障转移的更新和健康监控功能。这些能力已经成为用户希望现代容器编排工具提供的核心特点中的一部分。

## 声明配置
编排工具为DevOps团队提供了一个用于声明应用负载和它的标准方案配置的蓝图选项, 它们使用yaml或json格式。 这些定义还包含支持工作负载的仓库、网络(ports)、存储(volumes)和日志的信息。这种方法允许编排工具多次应用同一配置，并且总是在目标系统上产生相同的结果。 它还允许工具在同一个应用的不同阶段, 例如开发阶段、测试阶段和生产阶段接受不同的配置。

## 规则和限制
工作负载通常对主机放置、性能和高可用具有特殊的策略或要求。例如，在同一主机上提供主备数据库容器是无意义的；它破坏了目的性。类似的，在web服务器同一机器上放置内存缓存可能是个好主意。 编排工具支持定义容易放置的亲和性和约束的机制。

## Provisioning
供应(Provisioning)或调度是处理容器在集群中的放置和启动的。这个过程包括根据配置选择合适的主机。除了容器提供API, 编排工具也涉及针对主机环境的基础设施API。

## 发现
在由运行于多个主机上的容器组成的分布式部署, 容器发现至关重要。Web服务器需要动态发现数据库服务器，负载均衡需要发现并注册web服务器。 编排工具提供或期望有一个分布式key-value存储, 轻量级DNS或一些其他能够发现容器的机制。

## 健康监控
既然编排工具知道系统的期望配置，所以它们能够唯一的监控系统容器和主机的健康情况。在主机故障的情况下，这个工具可以重新定位容器。类似的，当容器崩溃时，编排工具可以启动替换。编排工具确保部署始终处于开发人员或操作人员声明的期望状态。

## 仔细看看三种流行的编排工具
### Docker Swarm
Docker Swarm的目标是使用和核心Docker引擎一起工作同样的Docker API。API端点的目标不是代表一个Docker引擎，Swarm透明的处理与Docker引擎池相关联的端点。这种方法的关键优势在于现有工具和API能够和对待单个实例的相同方式对待集群。Docker的工具/CLI和Compose是开发者如何创建它们应用程序的，因此，它们不得不重新编排来容纳一个编排器。

![](images/Chart_Docker-Swarm-Swap-Plug-and-Play.png)

遵循Docker的"自带电源、可移动"的哲学思想，支持数个发现后端，包括静态文件和IP地址，etcd, Consul和ZooKeeper。调度策略也是可插拔的。

- etcd: CoreOS 团队发起的一个管理配置信息和服务发现的项目。
- Consul: 另外一个提供服务发现的工具，它是分布式的、高可用、横向可扩展的。
- ZooKeeper: 一个分布式的，开放源码的分布式应用程序协调服务，是Google的Chubby一个开源的实现，是Hadoop和Hbase的重要组件。它是一个为分布式应用提供一致性服务的软件，提供的功能包括：配置维护、域名服务、分布式同步、组服务等。

Docker Swarm由几个内置的调度策略组成，给予用户指导容器放置的能力, 以最大化或最小化容器在集群中的扩散。也支持随机放置。

Docker寻址遵循"自带电源，但可移动"的原则，意思就是当前只使用少量的简单调度后端搬运，但是将来通过可插拔接口它能支持额外的后端。基于给定用例的规模和复杂度，开发人员和操作人员可以选择插入恰当的替代后端。

Docker Swarm支持确定容易在特定主机放置的约束和亲和力。

约束(Constraints)定义了需要选择用于调度的节点子集的要求。它们可以基于存储类型、地理位置、环境以及内核版本等属性。
亲和力(Affinity)定义了在主机上分配容器的需求。

为了在每个主机上发现容器，Swarm使用了可插拔后端架构，它与一个简单的托管发现服务、静态文件、IP列表、etcd、Consul和ZooKeeper一起工作。

Swarm支持基本的将康监控，这样可以防止在故障主机上提供容器。

## Kubernetes
来自Google - 声称每天处理20亿容器的公司 - 的Kubernetes享有独特的声誉。

![](images/Chart_Kubernetes-Building-on-Architectural-Roots.png)

Kubernetes的架构是基于一个带有很多奴仆的主服务器。命令行工具叫做kubecfg, 连接主服务器端点API来管理和编排奴仆节点。下面是运行在Kubernetes环境里边的每个组件的定义:

- Master: 运行Kubernetes管理进程的服务器，包括API服务，替换控制器和调度器。
- Minion: 运行kubelet服务和Docker引擎的主机。Minions从master接收命令。
- Kubelet: Kubernetes中的节点级别的管理器; 运行在minion上的。
- Pod: 部署在同一个minion上的容器集合。
- Replication controller: 定义需要运行的pods或容器数量。
- Service: 定义允许发现由每个容器以及使用通信的外部代理发布的服务/ports。
- Kubecfg: 和master沟通用于管理Kubernetes部署的命令行接口。

服务定义以及规则和约束是用JSON文件描述的。对于服务发现，Kubernetes提供了一个稳定的IP地址和相对于动态的pods集合的DNS名。当运行在Kubernetes pod上的容器连接到这个地址，连接被一个本地代理(叫做kube-proxy)转发到一个具体的后端容器。

Kubernetes支持用户实现的应用程序健康检查。这些检查由运行在每个minion上的kubelet执行, 以确保应用程序正确操作。当前，Kubernetes支持三种类型的健康检查：

- HTTP健康检查: kubelet将调用web端点。如果响应码是200到399之间，它就认为是成功的。
- Container exec: kubelet将在容器中执行一个命令。如果返回OK, 认为就是成功的。
- TCP socket: kubelet将尝试打开一个到容器的socket, 并确立一个连接。如果连接产生的话，就认为是健康的。

## Apache Mesos
Apache Mesos是一个开源集群管理器，简化了在共享服务器池上运行任务的复杂度。最初设计用于支持高性能计算负载, Mesos在0.20.0发布版本添加了Docker的支持。

![](images/Chart_Apache-Mesos-Built-for-High-Performance-Workloads.png)

一般的Mesos集群由一个或多个运行mesos-master服务器和运行了mesos-slave组件的服务器组成。每个slave注册到master来提供资源。 master和部署框架交互来给slaves委派任务。 下面是Mesos的架构:

- Master daemon: The mesos-master service runs on a master node and manages slave daemons.
- Slave daemon: The mesos-slave service runs on each slave node to run tasks that belong to a framework.
- Framework: An application definition consisting of a scheduler that registers with the master to receive resource offers, along with one or more executors to launch tasks on the slaves.
- Offer: The list of a slave node’s resources. Each slave node sends offers to the master, and the master provides offers to registered application frameworks.
- Task: The unit of work scheduled by a framework to be executed on a slave node.
- Apache ZooKeeper: The software used to coordinate the collection of master nodes.

Unlike other tools, Mesos ensures high availability of the master nodes using Apache ZooKeeper, which replicates the masters to form a quorum. A high availability deployment requires at least three master nodes. All nodes in the system, including masters and slaves, communicate with ZooKeeper to determine which master is the current leading master. The leader performs health checks on all the slaves and proactively deactivates any that fail.

When Mesos is used in conjunction with Marathon, service discovery can be enabled based on the HAProxy TCP/HTTP load balancer, along with an assistant script that uses Marathon’s REST API to regenerate a HAProxy configuration file periodically. Alternatively, Mesos-DNS, a DNS-based service discovery mechanism, has recently been released in beta.

## 总结
容器编排正快速发展。从主要的基础设施公司到PAAS供应商到早起的创业公司以及无服务器计算中，每个人都在叫嚣着在生态系统中占有一席之地。在容器编排工具上有很多贡献者，因为这些对于部署真实世界的应用程序是必不可少的，从而推动了Docker和容器的采用。 我们视图强调一些构建配置工具的关键贡献者，但它不仅仅是明显的编排工具，更重要的是，要查看构建、部署、CI/CD、Paas和编排器交互的其他工具，这是我们在自动化和编排目录涵盖大量篇幅的。

## 参考链接
- https://thenewstack.io/containers-container-orchestration/
