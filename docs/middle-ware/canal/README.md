# Canal

## 简介

- **canal** ，译意为水道/管道/沟渠，是一个**基于 MySQL 二进制日志的高性能数据同步系统，提供可靠的低延迟增量数据管道**。

- 基于日志增量订阅和消费的业务包括

  - 数据库镜像。

  - 数据库实时备份。

  - 索引构建和实时维护(拆分异构索引、倒排索引等)。

  - 业务 cache 刷新。

  - 带业务逻辑的增量数据处理。

- 当前的 canal 支持源端 MySQL 版本包括`5.1.x`、`5.5.x`、`5.6.x`、`5.7.x`、`8.0.x`。

## 特点

1. 支持所有平台。
2. 支持细粒度的系统监控，由[Prometheus](https://prometheus.io/)提供支持。
3. 支持通过GTID等不同方式解析和订阅MySQL binlog。
4. 支持高性能、实时的数据同步。
5. Canal Server 和 Canal Client 都支持 HA/Scalability，由 Apache ZooKeeper提供支持。
6. Docker 支持。

## 工作原理

### MySQL主备复制原理

1. MySQL master 将数据变更写入二进制日志( binary log，其中记录叫做二进制日志事件binary log events，可以通过 show binlog events 进行查看)。
2. MySQL slave 将 master 的 binary log events 拷贝到它的**中继日志**(relay log)。
3. MySQL slave 重放 relay log 中事件，将数据变更反映它自己的数据。

![468c1a14-e7ad-3290-9d3d-44ac501a7227](README-image/468c1a14-e7ad-3290-9d3d-44ac501a7227.jpg)

### canal 工作原理

- canal 模拟 MySQL slave 的交互协议，伪装自己为 MySQL slave ，向 MySQL master 发送 dump 协议。

- MySQL master 收到 dump 请求，开始推送 binary log 给 slave (即 canal )。

- canal 解析 binary log 对象(原始为 byte 流)。

![20191104101735947](README-image/20191104101735947.png)

## 多语言支持

- canal 特别设计了 client-server 模式，交互协议使用 protobuf 3.0，client 端可采用不同语言实现不同的消费逻辑。
- 支持 java、c#、go、php、python、rust 等语言。
- canal 作为 MySQL binlog 增量获取和解析工具，可将变更记录投递到 MQ 系统中，比如 Kafka/RocketMQ，可以借助于 MQ 的多语言能力。

## 解析日志投递MQ

- canal 1.1.1版本之后，默认支持将 canal server 接收到的 binlog 数据直接投递到MQ，目前默认支持的MQ系统有`kafka`和`RocketMQ`。

### 安装步骤

1. 安装 Zookeeper。

2. 安装 MQ。

3. 安装 canal.server。

   - 下载压缩包。

   - 解压。

   - 配置修改参数，修改instance 配置文件 `vi conf/example/instance.properties`。

     ```properties
     # 按需修改成自己的数据库信息
     #################################################
     ...
     canal.instance.master.address=192.168.1.20:3306
     # username/password,数据库的用户名和密码
     ...
     canal.instance.dbUsername = canal
     canal.instance.dbPassword = canal
     ...
     # mq config
     canal.mq.topic=example
     # 针对库名或者表名发送动态topic
     #canal.mq.dynamicTopic=mytest,.*,mytest.user,mytest\\..*,.*\\..*
     canal.mq.partition=0
     # hash partition config
     # canal.mq.partitionsNum=3
     # 库名.表名: 唯一主键，多个表之间用逗号分隔
     # canal.mq.partitionHash=mytest.person:id,mytest.role:id
     #################################################
     ```

   - 对应ip 地址的MySQL 数据库需进行相关初始化与设置。

   - 修改canal 配置文件`vi /usr/local/canal/conf/canal.properties`。

     ```properties
     # ...
     # 可选项: tcp(默认), kafka, RocketMQ
     canal.serverMode = kafka
     # ...
     # kafka/rocketmq 集群配置: 192.168.1.117:9092,192.168.1.118:9092,192.168.1.119:9092 
     canal.mq.servers = 127.0.0.1:6667
     canal.mq.retries = 0
     # flagMessage模式下可以调大该值, 但不要超过MQ消息体大小上限
     canal.mq.batchSize = 16384
     canal.mq.maxRequestSize = 1048576
     # flatMessage模式下请将该值改大, 建议50-200
     canal.mq.lingerMs = 1
     canal.mq.bufferMemory = 33554432
     # Canal的batch size, 默认50K, 由于kafka最大消息体限制请勿超过1M(900K以下)
     canal.mq.canalBatchSize = 50
     # Canal get数据的超时时间, 单位: 毫秒, 空为不限超时
     canal.mq.canalGetTimeout = 100
     # 是否为flat json格式对象
     canal.mq.flatMessage = false
     canal.mq.compressionType = none
     canal.mq.acks = all
     # kafka消息投递是否使用事务
     canal.mq.transaction = false
     ```

### canal.mq.dynamicTopic 表达式说明

> canal 1.1.3 版本之后，支持配置格式：`schema` 或 `schema.table`，多个配置之间使用逗号或分号分隔。

- `例子1` `test\\.test` 指定匹配的单表，发送到以test_test为名字的topic上。
- `例子2` `.*\\..*` 匹配所有表，则每个表都会发送到各自表名的topic上。
- `例子3` `test` 指定匹配对应的库，一个库的所有表都会发送到库名的topic上
- `例子4` `test\\..*` 指定匹配的表达式，针对匹配的表会发送到各自表名的topic上
- `例子5` `test,test1\\.test1` 指定多个表达式，会将test库的表都发送到test的topic上，test1\\.test1的表发送到对应的test1_test1 topic上，其余的表发送到默认的canal.mq.topic值。

### canal.mq.partitionHash 表达式说明

xx