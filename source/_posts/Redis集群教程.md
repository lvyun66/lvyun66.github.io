---
title: Redis集群教程
date: 2019-02-14 19:10:25
tags: redis
categories:
  - redis
  - cluster
comments: true
---

本文主要是翻译redis官网的一篇文章，并且加上作者的一些理解而成。若理解错误，请提出批评。

原文请查看[Redis cluster tutorial](https://redis.io/topics/cluster-tutorial#redis-cluster-101)

--------

# Redis集群教程

本文档是对Redis Cluster的简要介绍，它不适用复杂的方法来理解分布式系统概念。它提供了有关如何设置集群、测试和操作集群的说明，而不涉及Redis集群规范中涵盖的详细信息，而只是从用户的角度描述系统的行为方式。

但是，本教程尝试以最终用户的角度提供有关Redis Cluster的可用性和一致性特征的信息，以简明易懂的方式说明。

**请注意，本教程需要Redis 3.0或更高版本。**

如果你计划运行严格的Redis集群部署，则应该阅读更加正式的规范，即使不是严格的要求。但是，从本文档开始，使用Redis Cluster一段时间，然后才阅读规范是一个好主意。

## Redis Cluster 101

Redis Cluster提供了一种运行Redis安装的方法，其中数据在多个节点之间自动分片。

Redis Cluster还在分区期间提供了一定程度的可用性，实际上是在某些节点发生故障或无法通信时继续运行的能力。但是，如果发生较大的故障（例如，当大多数主设备不可用时），集群将停止运行。

所以实际上，你对Redis Cluster有什么看法？

- 能够在多节点之间自动拆分数据集
- 当节点的子集遇到故障或无法与集群的其他部分通信时，能够继续操作

## Redis Cluster TCP端口

**每个Redis Cluster节点都需要打开两个TCP连接。用于为客户端提供服务的普通Redis TCP端口，例如6379，加上通过想数据端口添加10000获得的端口，因此在示例中为16379。**

第二个高端口用户集群总线，这是使用二进制通信的节点到节点通信信道。节点使用集群总线进行故障检测，配置更新，故障转移授权等。客户端永远不应尝试集群总线端口，但始终使用正常的Redis命令端口，但确保在防火墙中打开这两个端口，否则Redis Cluster节点将无法通信。

命令端口和集群总线端口偏移是固定的，始终为10000。

请注意，要是每个Redis节点正常工作，你需要：

1. 用于与客户端通信的普通客户端端口（通常为6379）对所有需要访问集群的客户端以及所有其他集群节点（使用客户端端口进行密钥迁移）开放。
2. 必须可以从所有其他集群节点访问集群总线客户端（客户端端口+10000）。

如果不打开两个TCP端口，则集群将无法按预期工作。

集群总线使用不同的二进制协议进行节点到节点的数据交换，这更适合于使用很少的带宽和处理时间在节点之间交互信息。

## Redis Cluster和Docker

目前，Redis Cluster不支持NATed环境，也不支持重新映射IP地址或TCP端口的一般环境。

Docker使用一种称为端口映射的技术：与程序认为使用的端口相比，在Docker容器内运行的程序可能会使用不同的端口。这对使用同一服务器中同时使用相同端口运行多个容器非常有用。

为了使Redis Cluster和Docker兼容，你需要使用Docker的Host网络模式。有关更多信息，请查看Docker文件中的-net=host选项。

## Redis Cluster数据分片

Redis Cluster不使用一致的散列，而是使用不同形式的分片，其中每个键在概念上都是我们称之为散列槽（hash slot）的一部分。

Redis Cluster中有16384个散列槽，为了计算给定密钥的散列槽，我们只需要采用密钥模数16384的CRC16。

Redis Cluster中每个节点都负责散列槽的子集，例如，你可能拥有一个3节点的集群，其中：

- Node A包含0-5500的散列槽
- Node B包含5501-11000的散列槽
- Node C包含11001-16383的散列槽
  
这允许轻松添加和删除集群中的节点。例如，我想增加一个Node D，我需要将一些散列槽从Node A、Node B、Node C移动到Node D，如果我想删除Node A，只需要将Node A的节点移动到Node B和Node C中。当Node A为空时，可以完全从集群中删除它。

因为将散列槽从一个节点移动到另一个节点不需要停止操作，添加和删除节点或者更改节点所持有的散列槽的百分比，不需要任何停机时间。

只要涉及单个命令执行（或整个事务或Lua脚本执行）的所有键都属于同一个散列槽，Redis Cluster就支持多个键操作。用户可以通过使用称为哈希标记（hash tag）的概念强制多个密钥称为同一个散列槽的一部分。

哈希标记（Hash Tags）记录在Redis Cluster规范中，但要点是，如果一个键中{}括号之间有一个子串，则只对该字符串的内容进行哈希处理，所以例如这个{foo}密钥和另一个{foo}密钥保证在同一个散列槽中，并且可以在具有多个密钥作为参数的命令中一起使用。

## Redis Cluster主从模型

为了在主节点子集发生故障或无法与大多数节点通信时保持可用，Redis Cluster使用主从模式，其中每个散列槽从1（主机本身）到N个副本（N-1附加从机节点）。

在具有Node A、B、C的示例集群中，如果Node B发生故障，则集群无法继续，因为我们不再能在5501-11000范围内提供服务哈希位置的方法。

但是，当创建集群时（或稍后），我们会向每个主节点添加一个从节点，为了使最终的簇由作为主节点A、B、C和作为从节点的A1、B1、C1组成，如果Node B发生故障，系统能够继续使用。

Node B1复制B，B失败，集群将节点B1升级为新的主节点，并将继续正常使用。

但请注意，如果节点B和B1同时发生故障，Redis Cluster将无法继续运行。

## Redis Cluster一致性保证

Redis Cluster无法保证强一致性。实际上，这意味着在某些条件下，Redis Cluster可能会丢失系统向客户端确认的写入。

Redis Cluster可能丢失写入的第一个原因是它使用异步复制。这意味着在写入期间会发生一下情况：

- 客户端写入主节点B
- 主节点B向客户端回复确认
- 主节点B将写入复制到从节点B1、B2、B3

正如所见，B在回复客户端之前不等待来自B1、B2、B3的确认，因为这是对Redis来说是一个禁止的延迟惩罚，因此，如果客户端写入某些内容，主节点B确认写入，但在能够将写入发送到其他从属节点之前崩溃，其中一个从属服务器（未接收到写入）可以被提升为主节点，那么将会永远丢失写入。

这与大多数配置为每秒将数据刷新到磁盘的数据库非常相似，因此，由于过去不涉及分布式系统的传统数据库系统的经验，这是一个你已经能够推理的场景。同样，可以在回复客户端之前强制数据库刷新磁盘上的数据来提供一致性，但这通常会导致性能过低，在Redis Cluster的情况下，这相当于同步复制。

基本上需要在性能和一致性之间进行权衡。

Redis Cluster在绝对需要支持同步写入，通过WAIT命令实现，这使得丢失写入的可能性大大降低，但请注意，即时使用同步复制，Redis Cluster也不会实现强一致性：在更复杂的故障情况下，始终有可能将无法接收写入的从节点选为主节点。

还有另一个值得注意的地方是，Redis Cluster将丢失写入，这种情况发生在网络分区中，其中客户端与少数实例（只是包括主服务器）隔离。

以六个节点为例，包括Node A、B、C、A1、B1、C1，3个主节点和3个从节点。还有一个客户端Z1。在发生分区之后，可能在分区的一侧有Node A、C、A1、B1、C1，在另一侧有B和Z1。Z1仍然可以写入B，它将接受其写入。如果分区在很短的时间内恢复，集群将继续正常运行。但是，如果分区持续足够的时间是B1在分区的多数侧被提升为主节点，则Z1发送给B的写入将丢失。

请注意，Z1将能够发送到B的写入量存在最大窗口：如果分区的多数方面已经有足够的时间将从节点选为主节点，则少数端的每个主节点都会提供接受写入。

这段时间是Redis Cluster的一个非常重要的配置指令，称为节点超时。节点超时过后，主节点会被视为失败，可以由其中一个从节点替换。类似地，在节点超时已经过去而主节点无法感知大多数其他主节点之后，它进入错误状态并且停止接收写入。

# Redis Cluster参数配置

我们即将创建一个示例集群部署。在继续之前，让我们介绍Redis Cluster在redis.conf文件中引入的配置参数，有些会很明显，有些会在你继续阅读之后会变得更加清晰。

- **cluster-enable <yes/no>**：如果是，则在特定Redis实例中启用Redis集群支持。否则，实例像往常一样作为独立实例运行。
- **cluster-config-file <filename>**：请注意，尽管此选项的名称这不是用户可编辑的配置文件，但每次发生更改时，Redis Cluster节点会自动保持集群配置（基本上是状态）的文件，以便能够在启动时重新读取它。该文件列出了集群中其他的节点、状态、持久变量等内容。由于某些消息接收，通常会将此文件重写并刷新到磁盘上。
- **cluster-node-timeout <milliseconds>**：在超时时间内，集群节点不会被视为无效。如果主节点的可访问时间超过指定时间，则其从属节点将进行故障转移。此参数控制Redis集群中其他重要的事项。值得注意的是，在指定时间内无法访问大多数主节点的每个节点都将停止接受查询。
- **cluster-save-validity-factor <factor>**：如果设置为零，则从站将始终尝试故障转移主站，无论主站和从站之间的链路保持断开连接的时间长短。如果值为正，则计算最大断开时间，因为节点超时值乘以此选项提供的因子，如果节点是从属节点，如果主站断开超过指定时间，它将不会尝试启动故障转移。
- **cluster-migration-barrier <count>**
- **cluster-require-full-coverage <yes/no>**

# 创建并使用Redis Cluster

注意：要手动部署Redis集群，了解它的某些操作方面非常重要。但是，如果要启动集群并尽快运行，请跳过本节和下一节，然后直接使用`create-cluster`脚本创建Redis集群。

要创建集群，我们首先要做的是在集群模式下运行一下空的Redis实例。这基本上意味着，不使用普通的Redis实例创建集群，因为需要配置特殊模式，以便Redis实例启用集群特定的功能和命令。

以下是最小的Redis集群配置文件：

```config
port 7000
cluster-enabled yes
cluster-config-file nodes.conf
cluster-node-timeout 5000
appendonly yes
```

正如你所看到的，启动集群模式的只是启用集群的指令。每个实例还包含存储此节点配置的文件路1径，默认情况下为`nodes.conf`，这个文件永远不会被手动编辑，它只是在Redis Cluster实例启动时生成，并在每次需要是更新。

请注意，按预期工作的最小集群需要包含至少三个主节点。对于你的第一次测试，强烈建议启动具有三个主节点和三个从节点的六节点集群。

为此，创建一个新目录`clusters`，并创建以我们将在任何给定目录中运行的实例的端口好命令的一下目录。

就像是：

```shell
mkdir clusters
cd clusters
mkdir 7000 7001 7002 7003 7004 7005
```

在每个目录下创建一个`redis.conf`，从7000-7005。作为配置文件的模板，只需要使用上面的小例子，但请确保根据目录名称使用正确的端口号。

```shell
./bin/redis-server ./clusters/7000/redis.conf
```

从每个实例的日志中可以看出，由于不存在`nodes.conf`文件，因此每个节点都会为自己分配一个新ID。

此特定实例将永久使用此ID，以使实例在集群的上下文具有唯一名称。每个节点都使用此ID记住每个其他节点，而不是通过IP或者端口记住。IP地址和端口可能会发生变化，但唯一的节点标识符永远不会在节点的整个生命周期内发生变化。我们称这个标识符为`Node ID`。

## 创建集群

现在我们已经运行了许多实例，我们需要通过向节点编写一些有意义的配置来创建我们的集群。

如果你使用的是Redis 5，这很容易实现，因为我们可以使用嵌入到redis-cli的`Redis Cluster`命令来执行应用，可以用来创建新的集群，检查或者重新部署现有集群等。

对于Redis版本3和4，有一个名为`redis-trib.rb`的就工具非常相似。可以在Redis源代码分发的src目录中找到它。你需要安装`redis gem`才能运行`redis-trib`。

```shell
gem install redis
```

第一个例子，即集群创建，将使用redis 5中的`redis-cli`和在redis3、4版本中的`redis-ctib`。但是，所有下面的实例都只是使用`redis-cli`，因为你可以看到语法非常相似，并且可以通过使用`redis-trib.rb`帮助获取有关旧语法的信息，从而轻松地将一个命令行更改为另一个命令行。**重要提示**：请注意，如果你愿意，可以将redis 5`redis-cli`用于redis 4的集群上而不会出现问题。要用`redis-cli`创建集群，只需要键入：

```shell
./bin/redis-cli --cluster create 127.0.0.1:7000 127.0.0.1:7001 127.0.0.1:7002 127.0.0.1:7003 127.0.0.1:7004 127.0.0.1:7005 --cluster-replicas 1
```

在redis 3或者4键入：

```shell
./redis-trib.rb --cluster create 127.0.0.1:7000 127.0.0.1:7001 127.0.0.1:7002 127.0.0.1:7003 127.0.0.1:7004 127.0.0.1:7005
```

这里使用的命令是`create`，因为我们想创建一个新的集群。选项`--cluster-replicas 1`意味着我们想要为每个创建的主服务创建一个slave。其他参数是我要用于创建新集群的实例的地址列表。

显然，我们要求的唯一设置是创建一个包含3个主服务器和三个从服务器的集群。

`redis-cli`将为你提供配置，键入yes接受建议的配置。将配置加入集群，这意味着实例将被引导为彼此通信。最后如果一切顺利，你会看到下面的信息。

```shell
>>> Performing hash slots allocation on 6 nodes...
Master[0] -> Slots 0 - 5460
Master[1] -> Slots 5461 - 10922
Master[2] -> Slots 10923 - 16383
Adding replica 127.0.0.1:7003 to 127.0.0.1:7000
Adding replica 127.0.0.1:7004 to 127.0.0.1:7001
Adding replica 127.0.0.1:7005 to 127.0.0.1:7002
>>> Trying to optimize slaves allocation for anti-affinity
[WARNING] Some slaves are in the same host as their master
M: 126a204fd52df0adede630ad91b3c3434e1ec971 127.0.0.1:7000
   slots:[0-5460] (5461 slots) master
M: 22f9b0bf8dde6afad4c544b04ead5380745af198 127.0.0.1:7001
   slots:[5461-10922] (5462 slots) master
M: 8b92169294ddd61506372d656efc178c6100abd3 127.0.0.1:7002
   slots:[10923-16383] (5461 slots) master
S: cbd1967581bf92333cfac27f832cb8a806c0f57d 127.0.0.1:7003
   replicates 22f9b0bf8dde6afad4c544b04ead5380745af198
S: 3e11a361cccf8e9ebc346feba9cc4e2ea4f1db6f 127.0.0.1:7004
   replicates 8b92169294ddd61506372d656efc178c6100abd3
S: 84525e5e43e8e30ee1399294d9d045711fc73fb8 127.0.0.1:7005
   replicates 126a204fd52df0adede630ad91b3c3434e1ec971
Can I set the above configuration? (type 'yes' to accept): yes
>>> Nodes configuration updated
>>> Assign a different config epoch to each node
>>> Sending CLUSTER MEET messages to join the cluster
Waiting for the cluster to join
.
>>> Performing Cluster Check (using node 127.0.0.1:7000)
M: 126a204fd52df0adede630ad91b3c3434e1ec971 127.0.0.1:7000
   slots:[0-5460] (5461 slots) master
   1 additional replica(s)
M: 22f9b0bf8dde6afad4c544b04ead5380745af198 127.0.0.1:7001
   slots:[5461-10922] (5462 slots) master
   1 additional replica(s)
S: 84525e5e43e8e30ee1399294d9d045711fc73fb8 127.0.0.1:7005
   slots: (0 slots) slave
   replicates 126a204fd52df0adede630ad91b3c3434e1ec971
S: cbd1967581bf92333cfac27f832cb8a806c0f57d 127.0.0.1:7003
   slots: (0 slots) slave
   replicates 22f9b0bf8dde6afad4c544b04ead5380745af198
S: 3e11a361cccf8e9ebc346feba9cc4e2ea4f1db6f 127.0.0.1:7004
   slots: (0 slots) slave
   replicates 8b92169294ddd61506372d656efc178c6100abd3
M: 8b92169294ddd61506372d656efc178c6100abd3 127.0.0.1:7002
   slots:[10923-16383] (5461 slots) master
   1 additional replica(s)
[OK] All nodes agree about slots configuration.
>>> Check for open slots...
>>> Check slots coverage...
[OK] All 16384 slots covered.
```

这意味着至少有一个主实例为16384个可用槽提供服务。

## 使用`create-cluster`脚本创建Redis集群

如果你不想通过如上述手动配置和执行单个实例来创建redis集群，则可以使用更简单的系统。

只需要检查`utils/create-cluster`目录即可。里面有一个名为`create-cluster`的脚本，它是一个简单的bash脚本。要启动具有3个主服务器和3个从服务器的6节点集群，只需要键入一下命令：

```shell
create-cluster start
create-cluster create
```

当`redis-cli`实例程序要求你接受集群布局时，在步骤2中回复yes。

现在你可以与集群交互，默认情况下，第一个节点将从端口30001开始，完成后，使用一下命令停止集群：

```shell
create-cluster stop
```

有关如何运行脚本的更多信息，请阅读此目录下的`README`。

## 畅玩集群

在此阶段，Redis集群的一个问题是缺少客户端库实现，我知道以下实现：

- [redis-rb-cluster](http://github.com/antirez/redis-rb-cluster) 是作者编写的Ruby实现，作为其他的语言的参考。它是原始redis-rb的简单包装器，实现了最小的定义，可以有效的与集群通信。
- [redis-py-cluster](https://github.com/Grokzen/redis-py-cluster) Python的redis集群客户端，支持大多数的功能，正在积极开发中。
- [predis](https://github.com/nrk/predis) predis支持redis集群，最近更新了支持并且正在积极开发中。
- [jredis](https://github.com/xetorthio/jedis) 最常用的Java客户端，最近添加了对Redis集群的支持，请参阅项目README中的jredis集群部分。
- [StackExchange.Redis](https://github.com/StackExchange/StackExchange.Redis) 提供对C#的支持（并且应该适用于大多数.net语言、VB、F#等）
- [thunk-redis](https://github.com/thunks/thunk-redis) 支持Node.js和io.js，它是基于thunk/promise的redis客户端，具有流水线和集群。
- [redis-go-cluster](https://github.com/chasex/redis-go-cluster) 是使用Redigo库客户端作为基本客户端的Go语言Redis集群的实现。通过结果聚合实现MEGT/MSET。
- [redis-cli]()

测试Redis集群的一种简单方法是尝试上述任何客户端或者简单的使用redis-cli命令行实用程序。以下是使用后者的交互示例：

```shell
19:54:06 › ./bin/redis-cli -c -p 7000
127.0.0.1:7000> set foo bar
-> Redirected to slot [12182] located at 127.0.0.1:7002
OK
127.0.0.1:7002> set hello world
-> Redirected to slot [866] located at 127.0.0.1:7000
OK
127.0.0.1:7000> get foo
-> Redirected to slot [12182] located at 127.0.0.1:7002
"bar"
127.0.0.1:7002> get hello
-> Redirected to slot [866] located at 127.0.0.1:7000
"world"
```

注意，如果使用脚本创建集群，则节点可以监听不同的端口，默认情况下从30001开始。

redis-cli集群支持非常基础，因此它始终使用Redis集群节点能够保持客户端重定向到右节点的事实。一个严肃的客户端能够做到很好，并在哈希槽和节点地址之间缓存地图，以直接使用与正确节点的正确连接。仅当集群配置中的某些内容发生变更时（例如，在故障转移之后或系统管理员通过添加或删除节点更改集群后），才会刷新映射。

### 使用redis-rb-cluster编写示例应用程序

### 集群重新分片

现在我们准备尝试重新分片。为此，请保持`example.rb`程序运行，以便你可以查看是否对运行的程序的一些影响。此外，你可能需要注释睡眠时间，以便在重新分片期间有一些更严重的写入负载。

重新分片基本上意味着将散列槽从一组节点移动到另一组节点，并且想集群创建一样，它是使用redis-cli命令完成的。

要重新开始分片，只需输入：

```shell
redis-cli --cluster reshard 127.0.0.1:7000
```

只需指定一个节点，redis-cli将自动找到其他节点。

目前redis-cli只能通过管理员重新加载，你不能只说将5%的插槽从这个节点移动到另一个节点（但这实现起来非常简单）。所以从这个问题开始，首先是你需要做多大的重新slots：

我们可以尝试重新刷新1000和散列槽，如果示例仍在没有睡眠调用的情况下运行，那么它应该已包含非常少量的密钥。

然后redis-cli需要知道重新分片的目标是什么，即接受哈希槽的节点。我将使用第一个节点，即127.0.0.1:7000，但需要指定示例节点的ID，这已由redis-cli打印在列表中，但如果需要，总能使用一下命令找到节点ID：

```shell
redis-cli -p 7000 clusters nodes | grep myself

# 126a204fd52df0adede630ad91b3c3434e1ec971 127.0.0.1:7000@17000 myself,master - 0 1551144977000 1 connected 0-5460
```

目标节点ID为：126a204fd52df0adede630ad91b3c3434e1ec971

现在，你将被问到要从哪些节点获取这些密钥，我只需要键入`all`即可从所有其他主节点获取一些哈希槽。

在最终确认之后，将会看到redis-cli将从一个节点移动到另一个节点的每一个槽的消息，并且将为从一侧移动到另一侧的每个实际键打印一个点。

重新分片正在进行中，你应该可以看到示例程序不受影响的运行。如果需要，你可以在重新分片期间多次停止并重新启动他们。

在重新分片结束时，可以使用以下命令测试集群的运行情况：

```shell
redis-cli --cluster check 127.0.0.1:7000
```

所有的插槽都会像往常一样覆盖，但这次127.0.0.1:7000的主机将有更多的散列槽，大约6461个。

### 编写重新分片脚本

可以自动执行重新分片，而无需以交互手动输入方式。这可以使用如下命令行：

```shell
redis-cli reshard <host>:<port> --cluster-from <node-id> --cluster-to <node-id> --cluster-slots <number of slots> --cluster-yes
```

如果经常需要进行分片操作，可以创建一些自动化脚本，但是目前redis-cli无法自动重新平衡集群，检查集群节点上密钥的分布情况，并根据需要只能地移动插槽。此功能将在未来添加。

### 更有趣的示例

我们早期写的示例程序不是很好，它是以简单的方式写入集群，甚至不检查写入的内容是否正确。

从我们的角度来看，接受写入的集群可能总是将密钥foo写入42到每个操作，我们根本不会注意到。

所以在`redis-rb-cluster`存储库中，有一个更有趣的应用程序叫做`consistency-test.rb`。它使用一组计数器，默认为1000，并发送INCR命令以递增计数器。

然而，改应用程序不仅仅是编写，而是执行另外两项操作：

- 使用INCR更新计数器时，应用程序会记住写入
- 它还在每次写入之前读取一个随机计数器，并检查改值是否与我们预期的值相比，将其与内存中的值进行比较

### 测试故障转移

注意，在此测试期间，应该打开一个终端，并运行一致性测试应用程序。

为了触发故障转移，我们可以做的最简单的事情（也就是在分布式系统中可能出现的语义上最简单的故障）是使单个进程崩溃，在我们的示例中是单个进程。

我们可以使用一下命令识别集群并使其崩溃：

```shell
redis-cli -p 7000 cluster nodes | grep master

# 22f9b0bf8dde6afad4c544b04ead5380745af198 127.0.0.1:7001@17001 master - 0 1551170676000 2 connected 5962-10922
# 126a204fd52df0adede630ad91b3c3434e1ec971 127.0.0.1:7000@17000 myself,master - 0 1551170676000 7 connected 0-5961 10923-11421
# 8b92169294ddd61506372d656efc178c6100abd3 127.0.0.1:7002@17002 master - 0 1551170676578 3 connected 11422-16383
```

让我们用`debug segfault`命令崩溃节点7002：

```shell
redis-cli -p 7002 debug segfault
```

现在我们可以查看一致性测试的输出，看看它报告了什么：

### 添加新节点

添加新节点基本上是添加空节点然后将一些数据移入其中的过程，以防它是新的主节点，或者告诉它设置为已知节点的副本，以防它是从属节点。

我们将展示两者，从添加新的主服务器开始。

在两种情况下，执行的第一步是添加空节点。

这很简单，只需在端口7006中启动一个新节点（我们现在已经有6个节点，端口从7000-7005）其他节点使用相同的配置，端口号除外，那么你应该怎么做才能符合我们以前节点使用的设置：

- 在终端应用程序中创建一个新选项卡
- 进入`cluter`目录
- 创建7006目录
- 在里面创建一个`redis.conf`文件，类似于用于其他节点但使用7006端口的文件
- 最庸使用`./bin/redis-server ./cluster/7006/redis.conf`启动服务器

此时，服务器应该在运行。

现在我们可以想往常一样使用`redis-cli`命令以便将节点添加到我们集群中。

```shell
redis-cli --cluster add-node 127.0.0.1:7006 127.0.0.1:7000
```

如你所见，我使用`add-node`命令将新节点的地址指定为第一个参数，并将集群中随机存在节点的地址指定为第二个参数。

实际上，`redis-cli`在这方面做的好少帮助我们，它只是向节点发送一个`CLUSTER MEET`详细，这也可以手动完成。但是`redis-cli`在运行之前也会检查集群的状态，所以即使你知道内部是如何运行的，也总是通过`redis-cli`执行集群操作是个好主意。

现在我们可以连接到新节点以查看它是否真正加入集群：

```shell

```

请注意，由于此节点已连接到集群，因此它已能够正确地重定向客户端查询，并且通常是集群的一部分。然而，与其他master相比，它有两个特点：

- 它没有数据，因为它没有分配哈希槽
- 因为它是没有分配插槽的主节点，所有当从服务器成为主节点时，它不参与选举过程

现在可以使用`redis-cli`的重新分片功能为此节点分配哈希槽。显示这一点基本没用，就像我们在上一节中所做的那样，没有区别，它只是一个重新分区，具有空节点的目标。

### 添加新节点为副本

添加新副本可以通过两种方式进行。显而易见的是再次使用`redis-cli`，但使用`--cluster-slave`选项，如下所示：

```shell
redis-cli --cluster add-node 127.0.0.1:7006 127.0.0.1:7000 --cluster-slave
```

请注意，此处的命令行与我们用于添加新主服务器的命令完全相同，因此我们不指定要添加副本的主服务器。在这种情况下，会发生的事情是`redis-cli`会将新节点作为随机主副本的副本添加到副本较少的主服务器中。

但是，你可以使用一下命令行准确指定要使用新副本定位的主服务器。

```shell
redis-cli --cluster add-node 127.0.0.1:7006 127.0.0.1:7000 --cluster-slave --cluster-master-id xasf2k45s2341gf9f0hh0fldnylxiu
```

这样我们就可以将新副本分配给特定的主副本。

将副本添加到特定主节点的手动方法是将新节点添加为空主节点，然后使用`cluster replicate`命令将其转换为副本。如果节点作为从属节点添加，但你想将其作为不同节点的副本移动，则此方法也是有效的。

### 删除节点

要删除从属节点，只需要使用`redis-cli`的`del-node`命令：

```shell
redis-cli --cluster del-node 127.0.0.1:7000 <node-id>
```

第一个参数只是集群中的随机节点，第二个参数是要删除节点的ID。

你也可以以相同的方式删除主节点，但是为了删除主节点，它必须为空。如果主服务器不为空，则需要将数据从其重新分配给所有其他主节点。删除主节点的另一种方法是在其中一个节点上手动故障转移，并在节点变为新主节点的从节点后将其删除。显然，当你想减少集群中实际的主数量时，这没有用，在这种情况下，需要重新分片。

### 副本迁移

在Redis集群中，只需要使用一下命令，就可以随时使用不同的主服务器重新配置从属服务器进行复制：

```shell
CLUSTER REPLICATE <master-node-id>
```

但是，有一种特殊情况，你希望副本在没有系统管理员帮助的情况下自动从一个主服务器移动到另一个主服务器。副本的自动重新配置称为服务迁移，并且能够提高Redis集群的可靠性。

注意：你可以在Redis集群规范中阅读副本迁移的详细信息，这里我们仅提供有关一般概念的一些信息以及你应该从中获益的信息。

你可能希望让集群副本在特定情况下从一个主服务器移动到另一个主服务器的原因是，通常Redis集群与附加到给定主服务器的副本数量一样可以抵御故障。