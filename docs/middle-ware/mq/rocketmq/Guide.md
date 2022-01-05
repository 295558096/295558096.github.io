# RocketMQ 开发指南



## 产品发展史

### 1.x

- ````````````````Metaq```````````````` Metamorphosis，1.x。
- 由开源社区 killme2008 维护，开源社区非常活跃。

### 2.x

- Metaq 2.x。
- 于2012年10月份上线，在淘宝内部被广泛使用。

### 3.x

- 更名为RocketMQ。
- 基于阿里公司内部开源共建原则，RocketMQ项目只维护核心功能，去除了所有其他运行时依赖，核心功能最简化。
- 每个BU的个性化需求都在RocketMQ项目上进行深度定制。

- 在 RocketMQ 项目基础上衍生的项目如下
  - 为淘宝应用提供消息服务，`com.taobao.metaq v3.0` = RocketMQ + 淘宝个性化需求。
  - 为支付宝应用提供消息服务，`com.alipay.zpullmsg v1.0` = RocketMQ + 支付宝个性化需求。
  - 为B2B应用提供消息服务，`com.alibaba.commonmq v1.0` = Notify + RocketMQ + B2B 个性化需求。



## 专业术语

- `Producer` 消息生产者，负责产生消息，一般由业务系统负责产生消息。

- `Consumer` 消息消费者，负责消费消息，一般是后台系统负责异步消费。
- `Push Consumer` Consumer 的一种，应用通常向Consumer对象**注册一个 Listener 接口**，一旦收到消息，Consumer对象**立刻回调** Listener 接口方法。
- `Pull Consumer` Consumer 的一种，应用通常**主动调用 Consumer 的拉消息方法从 Broker 拉消息**，主动权由应用控制。
- `Producer Group` 一类 Producer 的集合名称，这类 Producer 通常发送一类消息，且发送逻辑一致。
- `Consumer Group` 一类 Consumer 的集合名称，这类 Consumer 通常消费一类消息，且消费逻辑一致。
- `Broker` **消息中转角色**，负责**存储消息**，**收发**消息，一般也称为 Server。在 JMS 规范中称为 Provider。
- `广播消费` 
  - 一条消息被多个 Consumer 消费，即使这些 Consumer 属于**同一个** `ConsumerGroup`，消息也会被 `ConsumerGroup` 中的每个 Consumer 都消费一次，广播消费中的 ConsumerGroup 概念可以认为在消息划分方面无意义。
  - 在 CORBA Notification 规范中，消费方式都属于广播消费。
  - 在 JMS 规范中，相当于`JMS publish/subscribe model`。

- `集群消费`
  - 一个 `ConsumerGroup` 中的 Consumer 实例**平均分摊**消费消息。
  - 在 CORBA Notification 规范中，无此消费方式。
  - 在 JMS 规范中，`JMS point-to-point model` 与之类似，但是 RocketMQ 的集群消费功能**大等于** PTP 模型。因为 RocketMQ 单个 ConsumerGroup 内的消费者类似于 PTP，但是一个 Topic/Queue 可以被多个 ConsumerGroup 消费。

- `顺序消息`
  - **消费消息的顺序要同发送消息的顺序一致**。
  - 在 RocketMQ 中，主要指的是**局部顺序**，即一类消息为满足顺序性，必须 Producer 单线程顺序发送，且发送到同一个队列，这样Consumer 就可以按照 Producer 发送的顺序去消费消息。
- `普通顺序消息`
  - 顺序消息的一种，**正常情况下可以保证完全的顺序消息**，但是一旦发生通信异常，Broker 重启，由于队列总数发生发化，**哈希取模**后定位的队列会发化，产生短暂的消息顺序不一致。
  - 如果业务能容忍在集群异常情况（如某个 Broker 宕机或者重启）下，消息短暂的乱序，使用普通顺序方式比较合适。
- 严格顺序消息
  - 顺序消息的一种，无论正常异常情况都能保证顺序，但是**牺牲了分布式 Failover 特性**，即 Broker 集群中只要有一台机器不可用，则整个集群都不可用，服务可用性大大降低。
  - 如果服务器部署为同步双写模式，此缺陷可通过备机**自动切换**为主避免，不过仍然会存在几分钟的服务不可用。
  - 目前已知的应用只有数据库 binlog 同步强依赖严格顺序消息，其他应用绝大部分都可以容忍短暂乱序，推荐使用普通的顺序消息。
- `Message Queue`
  - 在 RocketMQ 中，所有消息队列都是**持久化**，**长度无限**的数据结构。所谓长度无限是指队列中的每个存储单元都是定长，访问其中的存储单元使用 `Offset` 来访问，`Offset` 为 java long 类型，64 位，理论上在 100 年内不会溢出，所以认为是长度无限。
  - 队列中只保存最近几天的数据，之前的数据会按照过期时间来删除。
  - 也可以讣为 Message Queue 是一个长度无限的数组，offset 就是下标。



## 消息中间件需要解决哪些问题

### Publish / Subscribe

- 发布订阅是消息中间件的最基本功能，也是相对于传统 RPC 通信而言。

### Message Priority

- 规范中描述的优先级是只在一个消息队列中，每条消息都有不同的优先级，一般用**整数**来描述，优先级高的消息先投递，如果消息完全在一个内存队列中，那么在投递前可以按照优先级排序，令优先级高的先投递。
- 由于 RocketMQ 所有消息都是持久化的，所以如果按照优先级来排序，开销会非常大，因此 RocketMQ 没有特意支持消息优先级，但是可以通过变通的方式实现类似功能。
  - 即单独配置一个优先级高的队列，和一个普通优先级的队列， 将不同优先级发送到不同队列即可。
- 优先级问题
  - 只要达到优先级目的即可，不是严格意以上的优先级，通常将优先级划分为高、中、低，或者再多几个级别。**每个优先级可以用不同的 topic 表示**，发消息时，指定不同的 topic 来表示优先级，这种方式可以解决绝大部分的优先级问题，但是对业务的优先级精确性做了妥协。
  - 严格的优先级，优先级用整数表示，例如 0 ~ 65535，这种优先级问题一般使用不同 topic 解决就非常不合适。如果要让 MQ 解决此问题，会对 MQ 的性能造成非常大的影响。

### Message Order

- 消息有序指的是**一类消息消费时，能按照发送的顺序来消费**。
- RocketMQ 可以严格的保证消息有序。

### Message Filter

#### Broker 端消息过滤

- 在 Broker 中，按照 Consumer 的要求做过滤。
- 优点是减少了对于 Consumer 无用消息的网络传输，缺点是增加了 Broker 的负担，实现相对复杂。
- 淘宝 Notify 支持多种过滤方式，包含直接按照消息类型过滤，灵活的语法表达式过滤，几乎可以满足最苛刻的过滤需求。
- 淘宝 RocketMQ 支持按照简单的 `Message Tag` 过滤，也支持挄照 `Message Header`、body 进行过滤。
- CORBA Notification 规范中也支持灵活的语法表达式过滤。

#### Consumer 端消息过滤

- 这种过滤方式可由应用完全自定义实现，但是缺点是很多无用的消息要传输到 Consumer 端。

### Message Persistence

1. 持久化到数据库。
2. 持久化到 KV 存储。
3. 文件记录形式持久化，例如kafka、RocketMQ。
4. 对内存数据做一个持久化镜像。



- 1、2、3三种持久化方式都具有将内存队列 Buffer 进行扩展的能力。
- 4只是一个内存的镜像，作用是当 Broker 挂掉重启后仍然能将之前内存的数据恢复出来。
- JMS 不 CORBA Notification 规范没有明确说明如何持久化，但是**持久化部分的性能直接决定了整个消息中间件的性能**。
- RocketMQ 参考了 Kafka 的持久化方式，充分**利用 Linux 文件系统内存 cache 来提高性能**。

### Message Reliablity

影响消息可靠性的几种情况

1. Broker 正常关闭。
2. Broker 异常 Crash。
3. OS Crash。
4. 机器掉电，但是能立即恢复供电情况。
5. 机器无法开机（可能是 cpu、主板、内存等关键设备损坏）。
6. 磁盘设备损坏。



- 1、2、3、4四种情况都**属于硬件资源可立即恢复情况**，RocketMQ 在返四种情况下能保证消息不用丢，或者丢失少量数据（依赖刷盘方式是同步还是异步）。
- 5、6属于单点故障，无无法恢复，一旦发生，在此**单点上的消息全部丢失**。RocketMQ 在这两种情况下，通过**异步复制**，可保证 99%的消息不丢，但是仍然会有极少量的消息可能丢失。
- 通过**同步双写技术**可以完全规避单点，**同步双写势必会影响性能，适合对消息可靠性要求极高的场合**。
- RocketMQ 从 3.0 版本开始支持同步双写。

### Low Latency Messaging

- 在消息不堆积情况下，消息到达 Broker 后，能立刻到达 Consumer。
- RocketMQ 使用**长轮询 Pull 方式**，可保证消息非常实时，消息实时性不低于 Push。

### At least Once

- 指每个消息必须投递一次。
- RocketMQ Consumer 先 pull 消息到本地，**消费完成后，才向服务器返回 ack**，如果没有消费一定不会 ack 消息， 所以 RocketMQ 可以很好的支持此特性。

### Exactly Only Once

1. 发送消息阶段，不允许发送重复的消息。

2. 消费消息阶段，不允许消费重复的消息。

   

- 只有以上两个条件都满足情况下，才能认为消息是`Exactly Only Once`，而要实现以上两点，在分布式系统环境下，不可避免要**产生巨大的开销**。
- RocketMQ 为了追求高性能，并**不保证此特性**，要求在**业务上进行去重**， 也就是说消费消息要做到**幂等性**。

- RocketMQ 虽然不能严格保证不重复，但是**正常情况下很少会出现重复发送、消费情况，只有网络异常，Consumer 启停等异常情况下会出现消息重复**。
  - 此问题的本质原因是网络调用存在不确定性，即既不成功也不失败的第三种状态，所以才产生了消息重复性问题。

### Broker 的 Buffer 满了怎么办

Broker 的 Buffer 通常指的是 **Broker 中一个队列的内存 Buffer 大小**，这类 Buffer 通常大小有限，如果 Buffer 满了以后怎么办。

#### CORBA Notification 规范中处理方式

- RejectNewEvents
  - 拒绝新来的消息，向 Producer 返回 RejectNewEvents 错误码。
- 按照特定策略丢弃已有消息
  - `AnyOrder` - Any event may be discarded on overflow. This is the default setting for this property.
  - `FifoOrder` - The first event received will be the first discarded.
  - `LifoOrder` - The last event received will be the first discarded.
  - `PriorityOrder` - Events should be discarded in priority order, such that lower priority events will be discarded before higher priority events.
  - `DeadlineOrder` - Events should be discarded in the order of shortest expiry deadline first.

#### RocketMQ实现

- RocketMQ **没有内存 Buffer 概念**，RocketMQ 的队列**都是持久化磁盘，数据定期清除**。
- 对于此问题的解决思路，RocketMQ 同其他 MQ **有非常显著的区别**，**RocketMQ 的内存 Buffer 抽象成一个无限长度的队列**，不管有多少数据进来都能装得下，这个无限是有前提的，Broker 会定期删除过期的数据。

### 回溯消费

- 回溯消费是指 Consumer 已经消费成功的消息，由于业务上需求需要重新消费，要支持此功能，**Broker 在向 Consumer 投递成功消息后，消息仍然需要保留**。
- 重新消费一般是按照时间维度，例如由于 Consumer 系统故障， 恢复后需要重新消费 1 小时前的数据，那么 Broker 要提供一种机制，可以**按照时间维度来回退消费进度**。
- **RocketMQ 支持按照时间回溯消费，时间维度精确到毫秒，可以向前回溯，也可以向后回溯**。

### 消息堆积

消息中间件的主要功能是异步解耦，迓有个重要功能是挡住前端的数据洪峰，保证后端系统的稳定性，返就要求消息中间件具有一定的消息堆积能力。

#### Buffer满载

- 消息堆积在内存 Buffer，一旦超过内存 Buffer，可以根据一定的丢弃策略来丢弃消息，如 CORBA Notification 规范中描述。
- 丢弃策略适合能容忍丢弃消息的业务，这种情况消息的**堆积能力主要在于内存 Buffer 大小**，而且**消息堆积后，性能下降不会太大，因为内存中数据多少对于对外提供的访问能力影响有限**。

#### 磁盘IO瓶颈

- 消息堆积到持久化存储系统中，文件记录形式，当消息不能在内存 Cache 命中时，要不可避免的访问磁盘，会产生大量读 IO，**读 IO 的吞吐量直接决定了消息堆积后的访问能力**。

#### 评估消息堆积能力

- 消息能堆积多少条，多少字节？即消息的堆积容量。
- 消息堆积后，发消息的吞吐量大小，是否会受堆积影响？
- 消息堆积后，正常消费的 Consumer 是否会受影响？
- 消息堆积后，访问堆积在磁盘的消息时，吞吐量有多大？

### 分布式事务

- 已知的几个分布式事务规范，如 XA，JTA 等。
  - XA 规范被各大数据库厂商广泛支持，如 Oracle，Mysql 等。 
  - XA 的 TM 实现佼佼者如 Oracle Tuxedo，在金融、电信等领域被广泛应用。
- 分布式事务涉及到**两阶段提交**问题，在数据存储方面的方面必然需要 KV 存储的支持，因为**第二阶段的提交回滚需要修改消息状态**，一定涉及到**根据 Key 去查找 Message 的动作**。
- RocketMQ 在第二阶段**绕过了根据 Key 去查找 Message** 的问题，采用第一阶段发送 `Prepared` 消息时，拿到了消息的 `Offset`，第二阶段**通过 Offset 去访问消息**，并修改状态，**Offset 就是数据的地址**。
- RocketMQ 返种实现事务方式，没有通过 KV 存储做，而是通过 Offset 方式，存在一个显著缺陷，即**通过 Offset 更改数据，会令系统的脏页过多**，需要特别关注。

### 定时消息

- 定时消息是指消息发到 Broker 后，不能立刻被 Consumer 消费，要到特定的时间点或者等待特定的时间后才能被消费。
- 如果要支持任意的时间精度，在 Broker 局面，必须要做**消息排序**，如果再**涉及到持久化，那么消息排序要不可避免的产生巨大性能开销**。
- RocketMQ 支持定时消息，但是**不支持任意时间精度**，**支持特定的 level**。

### 消息重试

Consumer 消费消息失败后，要提供一种重试机制，令消息再消费一次。

Consumer 消费消息失败通常可以认为有以下几种情况。

- 由于消息本身的原因，例如反序列化失败，消息数据本身无法处理等。

  这种错误通常需要跳过这条消息，再消费其他消息，而返条失败的消息即使立刻重试消费，99%也不成功， 所以最好提供一种定时重试机制，即过 10s 秒后再重试。

- 由于依赖的下游应用服务不可用，例如 db 连接不可用，外系统网络不可达等。

  遇到这种错误，即使跳过当前失败的消息，消费其他消息同样也会报错。这种情况建议应用 sleep 30s，再消费下一条消息，这样可以减轻 Broker 重试消息的压力。

## Overview

### RocketMQ是什么

- 是一个队列模型的消息中间件，具有高性能、高可靠、高实时、分布式特点。
- Producer、Consumer、队列都可以分布式。
- Producer 向一些队列**轮流**发送消息，**队列集合称为 Topic**。
- Consumer 如果做**广播消费**，则一个 Consumer 实例消费这个Topic对应的所有队列。
- Consumer 如果做**集群消费**，则多个 Consumer 实例**平均消费**这个Topic对应的队列集合。
- 能够保证严格的消息顺序。
- 提供丰富的消息拉叏模式。
- 高效的订阅者水平扩展能力。
- 实时的消息订阅机制。
- 亿级消息堆积能力。
- 较少的依赖。

![image-20220105161926757](Guide-image/image-20220105161926757.png)

### 物理部署结构

#### 网络部署特点

- Name Server 是一个几乎无状态节点，可集群部署，节点之间无任何信息同步。
- Broker 部署相对复杂，Broker 分为 Master 与 Slave。
  - 一个 Master 可以对应多个 Slave，但是一个 Slave 只能对应一个 Master。
  - Master 与 Slave 的对应关系通过指定相同的 BrokerName，不同的 BrokerId 来定义。
  - BrokerId 为 0 表示 Master，非 0 表示 Slave。
  - Master 也可以部署多个。每个 Broker 与 Name Server 集群中的所有节点建立长连接，定时注册 Topic 信息到所有 Name Server。
- Producer 与 Name Server 集群中的其中一个节点（随机选择）建立长连接，定期从 Name Server 取 Topic 路由信息，并向提供 Topic 服务的 Master 建立长连接，且定时向 Master 发送心跳。Producer 完全无状态，可集群部署。
- Consumer 与 Name Server 集群中的其中一个节点（随机选择）建立长连接，定期从 Name Server 取 Topic 路由信息，并向提供 Topic 服务的 Master、Slave 建立长连接，且定时向 Master、Slave 发送心跳。**Consumer 既可以从 Master 订阅消息，也可以从 Slave 订阅消息，订阅规则由 Broker 配置决定。**



![image-20220105163053066](Guide-image/image-20220105163053066.png)

### 逻辑部署结构

#### Producer Group

- 用来表示一个发送消息应用。
- 一个 Producer Group 下包含多个 Producer 实例，可以是多台机器，也可以是一台机器的多个进程，或者一个进程的多个 Producer 对象。
- 一个 Producer Group 可以发送多个 Topic 消息。
- Producer Group 作用
  - 标识一类 Producer。
  - 可以通过运维工具查询这个发送消息应用下有多个 Producer 实例。
  - **发送分布式事务消息时，如果 Producer 中途意外宕机，Broker 会主动回调 Producer Group 内的任意一台机器来确认事务状态。**

#### Consumer Group

- 用来表示一个消费消息应用。
- 一个 Consumer Group 下包含多个 Consumer 实例，可以是多台机器，也可以是多个进程，或者是一个进程的多个 Consumer 对象。
- 一个 Consumer Group 下的多个 Consumer 以**均摊方式消费消息**，如果设置为广播方式，那举这个 Consumer Group 下的每个实例都消费全量数据。

![image-20220105163520638](Guide-image/image-20220105163520638.png)

## 存储特点

### 零拷贝原理

> Consumer 消费消息过程，使用了零拷贝。

#### 使用 mmap + write 方式

**优点**

- 即使频繁调用，使用小块文件传输，效率也很高。

**缺点**

- 不能很好的利用 DMA 方式，会比 sendfile **多消耗 CPU**，**内存安全性控制复杂，需要避免 JVM Crash 问题**。

#### 使用 sendfile 方式

**优点**

- 可以利用 DMA 方式，**消耗 CPU 较少**，**大块文件传输效率高，无内存安全新问题**。

**缺点**

- 小块文件效率低于 mmap 方式，**只能是 BIO 方式传输，不能使用 NIO**。



RocketMQ 选择了第一种方式，**mmap+write** 方式，因为**有小块数据传输的需求**，效果会比 sendfile 更好。

### 文件系统

RocketMQ 选择 Linux Ext4 文件系统。

- Ext4 文件系统**删除** 1G 大小的文件通常耗时小于 50ms，而 **Ext3 文件系统耗时约 1s 左右，且删除文件时，磁盘 IO 压力极大，会导致 IO 写入超时**。
- 文件系统层面需要做调优措施。
  - 文件系统 IO 调度算法需要调整为 `deadline`，因为 deadline 算法在**随机读情况下，可以**合并读请求为顺序跳跃方式，从而提高读 IO 吞吐量**。
- Ext4 文件系统有 Bug，请注意。

### 数据存储结构

![image-20220105164952174](Guide-image/image-20220105164952174.png)

### 存储目录结构

![image-20220105165557314](Guide-image/image-20220105165557314.png)

### 数据可靠性

略

## 关键特性

#### 单机支持1万以上持久化队列

- 所有数据单独存储到一个 `Commit Log`，**完全顺序写，随机读**。
- 对最终用户展现的**队列实际只存储消息在 Commit Log 的位置信息**，并且串行方式刷盘。

**优点**

- 队列轻量化，单个队列数据量非常少。
- 对磁盘的访问串行化，避免磁盘竟争，**不会因为队列增加导致 IOWAIT 增高**。

**缺点**

- **写虽然完全是顺序写，但是读却变成了完全的随机读**。
- 读一条消息，会先读 Consume Queue，再读 Commit Log，**增加了开销**。
- 要保证 Commit Log 与 Consume Queue 完全的一致，增加了编程的复杂度。

**缺点的解决方案**

- 随机读，**尽可能让读命中 PAGECACHE，减少 IO 读操作，所以内存越大越好**。如果系统中堆积的消息过多， 读数据要访问磁盘不会由于随机读导致系统性能急剧下降。
  - 访问 PAGECACHE 时，即使只访问 1k 的消息，系统也会提前预读出更多数据，在下次读时，就可能命中内存。
  - 随机访问 Commit Log 磁盘数据，**系统 IO 调度算法设置为 NOOP 方式**，会在**一定程度上将完全的随机读变成顺序跳跃方式**，而**顺序跳跃方式读较完全的随机读性能会高 5 倍以上**。
  - 4k 的消息在完全随机访问情况下，仍然可以达到 8K 次每秒以上的读性能。
- 由于 **Consume Queue 存储数据量极少**，而且是**顺序读**，在 PAGECACHE 预读作用下，即使堆积情况下，**Consume Queue 的读性能几乎与内存一致**。所以可以为 `Consume Queue` 完全不会阻碍读性能。
- Commit Log 中**存储了所有的元信息**，包含消息体，类似于 Mysql、Oracle 的 redolog，所以只要有 Commit Log 在，Consume Queue 即使数据丢失，仍然可以恢复出来。

![image-20220105165825360](Guide-image/image-20220105165825360.png)

### 刷盘策略

RocketMQ 的所有消息都是持久化的，**先写入系统 PAGECACHE，然后刷盘**，可以保证内存与磁盘都有一份数据， 访问时，直接从内存读取。

#### 异步刷盘

<img src="Guide-image/image-20220105172116671.png" alt="image-20220105172116671" style="zoom:50%;" />

在有 RAID 卡，SAS 15000 转磁盘测试顺序写文件，速度可以达到 300M 每秒左右，而线上的网卡一般都为千兆网卡，写磁盘速度明显快于数据网络入口速度，那么是否可以做到写完内存就向用户返回，由后台线程刷盘呢。

- 由于磁盘速度大于网卡速度，那么刷盘的速度肯定可以跟上消息的写入速度。
- 即使由于此时系统压力过大，堆积消息，除了写入 IO，还有读取 IO，出现磁盘读取落后情况， 也不会导致系统内存溢出。
  - 写入消息到 PAGECACHE 时，如果内存不足，则尝试丢弃干净的 PAGE，腾出内存供新消息使用，策略 是 LRU 方式。
  - 如果干净页不足，此时**写入 PAGECACHE 会被阻塞**，系统尝试刷盘部分数据，大约每次尝试 32 个 PAGE，来找出更多干净 PAGE。

#### 同步刷盘

>同步刷盘与异步刷盘的唯一区别是**异步刷盘写完 PAGECACHE 直接返回，而同步刷盘需要等待刷盘完成才返回**。

<img src="Guide-image/image-20220105172634577.png" alt="image-20220105172634577" style="zoom:50%;" />

同步刷盘流程。

- 写入 PAGECACHE 后，线程等待，通知刷盘线程刷盘。
- 刷盘线程刷盘后，唤醒前端等待线程，可能是一批线程。
- 前端等待线程向用户返回成功。