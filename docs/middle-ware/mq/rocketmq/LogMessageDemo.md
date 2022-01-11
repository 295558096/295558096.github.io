# 日志追加消息示例

- RocketMQ 的 `logappender`提供`log4j appender`，`log4j2 appender`和`logback appender`供业务使用。

## Log4j

### 配置

```java
log4j.appender.mq=org.apache.rocketmq.logappender.log4j.RocketmqLog4jAppender
log4j.appender.mq.Tag=yourTag
log4j.appender.mq.Topic=yourLogTopic
log4j.appender.mq.ProducerGroup=yourLogGroup
log4j.appender.mq.NameServerAddress=yourRocketmqNameserverAddress
log4j.appender.mq.layout=org.apache.log4j.PatternLayout
log4j.appender.mq.layout.ConversionPattern=%d{yyyy-MM-dd HH:mm:ss} %-4r [%t] (%F:%L) %-5p - %m%n
```

### 配置Appender

```xml
<appender name="mqAppender1" class="org.apache.rocketmq.logappender.log4j.RocketmqLog4jAppender">
    <param name="Tag" value="yourTag" />
    <param name="Topic" value="yourLogTopic" />
    <param name="ProducerGroup" value="yourLogGroup" />
    <param name="NameServerAddress" value="yourRocketmqNameserverAddress"/>
    <layout class="org.apache.log4j.PatternLayout">
        <param name="ConversionPattern" value="%d{yyyy-MM-dd HH:mm:ss}-%p %t %c - %m%n" />
    </layout>
</appender>

<appender name="mqAsyncAppender1" class="org.apache.log4j.AsyncAppender">
    <param name="BufferSize" value="1024" />
    <param name="Blocking" value="false" />
    <appender-ref ref="mqAppender1"/>
</appender>
```

## log4j2

### 配置Appender

```xml
<appender name="mqAppender1" class="org.apache.rocketmq.logappender.logback.RocketmqLogbackAppender">
    <tag>yourTag</tag>
    <topic>yourLogTopic</topic>
    <producerGroup>yourLogGroup</producerGroup>
    <nameServerAddress>yourRocketmqNameserverAddress</nameServerAddress>
    <layout>
        <pattern>%date %p %t - %m%n</pattern>
    </layout>
</appender>

<appender name="mqAsyncAppender1" class="ch.qos.logback.classic.AsyncAppender">
    <queueSize>1024</queueSize>
    <discardingThreshold>80</discardingThreshold>
    <maxFlushTime>2000</maxFlushTime>
    <neverBlock>true</neverBlock>
    <appender-ref ref="mqAppender1"/>
</appender>
```

## logback

### 配置AsyncAppender

```xml
<appender name="mqAppender1" class="org.apache.rocketmq.logappender.logback.RocketmqLogbackAppender">
    <tag>yourTag</tag>
    <topic>yourLogTopic</topic>
    <producerGroup>yourLogGroup</producerGroup>
    <nameServerAddress>yourRocketmqNameserverAddress</nameServerAddress>
    <layout>
        <pattern>%date %p %t - %m%n</pattern>
    </layout>
</appender>

<appender name="mqAsyncAppender1" class="ch.qos.logback.classic.AsyncAppender">
    <queueSize>1024</queueSize>
    <discardingThreshold>80</discardingThreshold>
    <maxFlushTime>2000</maxFlushTime>
    <neverBlock>true</neverBlock>
    <appender-ref ref="mqAppender1"/>
</appender>
```



