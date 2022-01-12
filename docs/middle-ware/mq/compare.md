# 常见消息中间件对比

## 定位

### 设计定位

#### Kafka

- 系统间数据流管道，实时数据处理。
- 常规的消息系统。
- 网站活性跟踪。
- 监控数据、日志收集、处理等。

#### RocketMQ

- 非日志的可靠消息传输。

#### RabbitMQ

- 可靠的消息传输，与RocketMQ类似。



## 基础对比

### 成熟度

#### Kafka

- 日志领域成熟。

#### RocketMQ

- 成熟。

#### RabbitMQ

- 成熟。

### 所属公司/社区

#### Kafka

- Apache。

#### RocketMQ

- Alibaba开发，已加入到Apache。

#### RabbitMQ

- Mozilla Public License。

### 社区活跃度

#### Kafka

- 高。

#### RocketMQ

- 中。

#### RabbitMQ

- 高。

### API完备性

#### Kafka

- 高。

#### RocketMQ

- 高。

#### RabbitMQ

- 高。

### 文档完备性

#### Kafka

- 高。

#### RocketMQ

- 高。

#### RabbitMQ

- 高。

### 开发语言

#### Kafka

- Scala。

#### RocketMQ

- Java。

#### RabbitMQ

- Erlang。

### 支持协议

#### Kafka

- 一套自行设计的基于 TCP 的二进制协议。

#### RocketMQ

- 自己定义的一套（社区提供 JMS 不成熟）。

#### RabbitMQ

- AMQP。

### 客户端语言

#### Kafka

- C、C++、Python、Go、Erlang、.NET、Ruby、Node.js、PHP等。

#### RocketMQ

- Java。

#### RabbitMQ

- Java、C、C++、Python、PHP、Perl 等。

### 持久化发送

#### Kafka

- 磁盘文件。

#### RocketMQ

- 磁盘文件。

#### RabbitMQ

- 内存、文件。

## 可用性、可靠性比较

### 部署方式

#### Kafka

- 单机。
- 集群。

#### RocketMQ

- 单机。
- 集群。

#### RabbitMQ

- 单机。
- 集群。

### 集群管理

#### Kafka

- zookeeper。

#### RocketMQ

- name server。

#### RabbitMQ

无

### 选主方式

#### Kafka

- 从ISR中自动选举一个leader。

#### RocketMQ

- 不支持自动选主。
- 通过设定BrokerName和BrokerId实现。
- BrokerName相同，BrokerId为0的为master，非0的为slave。

#### RabbitMQ

- 最早加入集群的broker。

### 可用性

#### Kafka

- 非常高。
- 分布式。
- 主从。

#### RocketMQ

- 非常高。
- 分布式。
- 主从。

#### RabbitMQ

- 高。
- 主从。
- 采用镜像模式实现，数据量大时可能产生性能瓶颈。

### 主从切换

#### Kafka

- 自动切换。
- N个副本，允许N-1个失效。
- master失效以后自动从ISR中选择一个主。

#### RocketMQ

- 不支持自动切换。
- Master 失效以后不能向 master 发送信息，consume 大约 30s（默认）可以感知此事件，此后从 slave 消费。
- 如果 master 无法恢复，**异步复制时可能出现部分信息丢失**。

#### RabbitMQ

- 自动切换。
- 最早加入集群的slave会成为master。
- 因为新加入的slaver不会同步master之前的数据，所以可能会出现部分数据丢失。

### 数据可靠性

#### Kafka

- 很好。
- 支持producer单条发送、同步刷盘、同步复制，但这些场景下性能明显下降。

#### RocketMQ

- 很好。
- 支持producer单条发送、异步刷盘、同步双写、异步复制。

#### RabbitMQ

- 好。
- producer支持同步/异步ack。
- 支持队列数据持久化，镜像模式中支持主从同步。

### 消息写入性能

#### Kafka

- 非常好。
- 每条10字节测试，百万条/s。

#### RocketMQ

- 很好。
- 每条10字节测试，单机单broker约7w/s，单机3broker约12w/s。

#### RabbitMQ

- RAM约为RocketMQ的1/2。
- DISK约为RocketMQ的1/3。

### 性能的稳定性

#### Kafka

- 队列/分区多时性能不稳定，明显下降。
- 消息堆积时性能稳定。

#### RocketMQ

- 队列较多、消息堆积时性能稳定。

#### RabbitMQ

- 消息堆积时，性能不稳定、明显下降。

### 单机支持的队列数

#### Kafka

- 单机超过64个队列/分区，Load会发生明显的飙高现象。
- **队列越多，Load 越高，发送消息响应时间变长**。

#### RocketMQ

- 单机支持最高 5 万个队列，Load 不会发生明显变化。

#### RabbitMQ

- 依赖于内存。

### 堆积能力

#### Kafka

- 非常好。
- 消息存储在log 中，每个分区一个log文件。

#### RocketMQ

- 非常好。
- 所有消息存储在同一个 commitlog 中。

#### RabbitMQ

- 一般。
- 生产者、消费者正常时，性能表现稳定。
- 消费者不正常消费时，性能不稳定。

### 复制备份

#### Kafka

- 消息先写入leader的log，followers从leader中pull，pull到数据后先ack leader，然后写入log中。
- ISR中维护与leader同步的列表，落后太多的followers被删除掉。

#### RocketMQ

- 同步双写。
- 异步双写，slave启动线程从master中拉取数据。

#### RabbitMQ

- 普通模式下不复制。
- 镜像模式下，消息先到master，然后写到slave上。
- 加入集群之前的消息不会被复制到新的slaver上。

### 消息投递实时性

#### Kafka

- 毫秒级。
- 具体由 consumer 轮询间隔时间决定。

#### RocketMQ

- 毫秒级。
- 支持pull、push两种模式。

#### RabbitMQ

- 毫秒级。

  

## 功能对比

### 顺序消费

#### Kafka

- 支持顺序消费。
- 但是一台Broker宕机后，就会产生消息乱序。

#### RocketMQ

- 支持顺序消费。
- 在顺序消费场景下，消费失败时消息队列将会暂停。

#### RabbitMQ

- 支持顺序消费。
- 如果一个消费失败，该消息的顺序会被打乱。

### 定时消息

#### Kafka

- 不支持。

#### RocketMQ

- 开源版本仅支持定时Level。

#### RabbitMQ

- 不支持。

### 事务消息

#### Kafka

- 不支持。

#### RocketMQ

- 支持。

#### RabbitMQ

- 不支持。

### Broker端消息过滤

#### Kafka

- 不支持。

#### RocketMQ

- 支持。
- 通过Tag过滤，类似于子Topic。

#### RabbitMQ

- 不支持。

### 消息查询

#### Kafka

- 不支持。

#### RocketMQ

- 支持。
- 根据MessageId查询。
- 支持根据MessageKey查询消息。

#### RabbitMQ

- 不支持。

### 消息重新消费

#### Kafka

- 支持通过修改offset来重新消费。

#### RocketMQ

- 支持按照时间来重新消费消息。

#### RabbitMQ

- 暂不支持。

### 发送端负载均衡

#### Kafka

- 可自由指定。

#### RocketMQ

- 可自由指定。

#### RabbitMQ

- 需要单独loadbalancer支持。

### 消费并行度

#### Kafka

- 消费并读取和分区数一致。

#### RocketMQ

- 顺序消费，消费并读取和分区数一致。
- 乱序消费，消费服务器的消费线程数之和。

#### RabbitMQ

- 镜像模式下其实也是从master消费。

### 消费方式

#### Kafka

- consumer pull。

#### RocketMQ

- consumer pull。
- broker push。

#### RabbitMQ

- broker push。

### 批量发送

#### Kafka

- 支持。
- 默认producer缓存、压缩，然后批量发送。

#### RocketMQ

- 支持。
- 必须发送到相同主题，相同Tag下。

#### RabbitMQ

- 不支持。

### 消息清理

#### Kafka

- 指定文件保存时间，过期删除。

#### RocketMQ

- 指定文件保存时间，过期删除。

#### RabbitMQ

- 可用内存少于40%，触发GC，GC时候找到相邻的两个文件，合并right文件到left。

### 访问控制权限

#### Kafka

- 无。

#### RocketMQ

- 无。

#### RabbitMQ

- 类似数据库一样，需要配置用户和密码。

## 运维

### 系统维护

#### Kafka

- Scala语言开发，维护成本高。

#### RocketMQ

- Java语言开发，维护成本低。

#### RabbitMQ

- Erlang语言开发，维护成本高。

### 部署依赖

#### Kafka

- zookeeper。

#### RocketMQ

- nameserver。

#### RabbitMQ

- Erlang环境。

### 管理后台

#### Kafka

- 官网不提供，第三方开源管理工具可供使用。
- 不用重新开发。

#### RocketMQ

- 官方提供，rocketmq-console。

#### RabbitMQ

- 官方提供，rabbitmqadmin。

## 总结

### 优点

#### Kafka

- 在高吞吐、低延迟、集群热扩展、集群容错上有非常好的表现。
- producer端提供缓存、压缩功能，可节省性能、提高效率。
- 提供顺序消费能力。
- 提供多种客户端语言。
- 生态完善，在大数据处理方面有大量配套的设施。

#### RocketMQ

- 在高吞吐、低延迟、高可用上有非常好的表现，消息堆积时，性能也很好。
- api、系统设计都更加适合在业务处理的场景。
- 支持多种消费方式。
- 支持broker消息过滤。
- 支持事务。
- 提供消息顺序消费能力，consumer可以水平扩展，消费能力很强。
- 集群规模在50台左右，单日处理消息上百亿。
- 经历过大数据量的考验，比较稳定可靠。

#### RabbitMQ

- 在高吞吐量、高可用上较前两者有所不如。
- 支持多种客户端语言，AMQP协议。
- 由于erlang语言的特性，性能比较好。
- 使用RAM模式时，性能很好。
- 管理界面较丰富。

### 缺点

#### Kafka

- 消费集群数目受到分区数目的限制。
- 单机topic多时，性能明显下降。
- 不支持事务。

#### RocketMQ

- 相对于Kafka，使用者较少，生态不够完善。
- 消息堆积、吞吐率上也不如Kafka。
- 不支持主从自动切换，master失效后，消费者需要过一定时间才能感知。
- 客户端只支持java。

#### RabbitMQ

- erlang语言难度较大，集群不支持动态扩展。
- 不支持事务、消息吞吐能力有限。
- 消息堆积时，性能会明显降低。