# 过滤消息示例

- 在大多数情况下，通过TAG进行消息过滤。
- 一个消息只能指定一个TAG。
- 消息消费者可以使用SQL表达式筛选出消息。

## 原理

- SQL功能可以通过您在发送消息时放入的属性进行一些计算。

## 语法

- RocketMQ定义了一些基本的语法来支持这个功能。 
- 可以自定义扩展语法。

### 基础语法

- 数字比较

  - `>`
  - `>=`
  - `<`
  - `<=`
  - `BETWEEN`
  - `=`

- 字符比较

  - `=` 

  - `<>`

  - `IN`

- `IS NULL` 或者 `IS NOT NULL`

- 逻辑运算 

  - `AND`
  - `OR`
  - `NOT`



### 常量类型

- 数字。
- 字符串，必须使用单引号。
- NULL，特殊常数。
- 布尔常量，`TRUE`和`FALSE`。

## 使用限制

- 只有消费者可以通过`SQL92`选择消息。

## 发送消息

- 构建消息的时候，添加用户自定义属性。

```java
import org.apache.rocketmq.client.producer.DefaultMQProducer;
import org.apache.rocketmq.client.producer.SendResult;
import org.apache.rocketmq.common.message.Message;
import org.apache.rocketmq.remoting.common.RemotingHelper;

public class FilterMessageProducer {

    public static void main(String[] args) throws Exception {
        // 设置生产者组信息
        DefaultMQProducer producer = new DefaultMQProducer("FilterMessageProducerGroup");
        // 设置NameServer
        producer.setNamesrvAddr("10.10.5.238:9876");
        producer.start();
        for (int i = 0; i < 10; i++) {
            // 构建消息
            Message msg = new Message("TopicTest", "TagC", "KEY" + i,
                String.valueOf(i).getBytes(RemotingHelper.DEFAULT_CHARSET));
            msg.putUserProperty("a", i + "");
            SendResult sendResult = producer.send(msg);
            // 消息发送结果
            System.out.printf("%s%n", sendResult);
        }
        // 关闭生产者
        producer.shutdown();
    }
}

```

## 订阅消息

- RocketMQ需要开启支持过滤属性的配置 `enablePropertyFilter=true`。
- 订阅的时候指定规则。

```java
import java.nio.charset.StandardCharsets;
import org.apache.rocketmq.client.consumer.DefaultMQPushConsumer;
import org.apache.rocketmq.client.consumer.MessageSelector;
import org.apache.rocketmq.client.consumer.listener.ConsumeConcurrentlyStatus;
import org.apache.rocketmq.client.consumer.listener.MessageListenerConcurrently;
import org.apache.rocketmq.client.exception.MQClientException;

public class FilterMessageConsumer {

    public static void main(String[] args) throws MQClientException {
        DefaultMQPushConsumer consumer = new DefaultMQPushConsumer("FilterMessageConsumerGroup");
        // 设置NameServer
        consumer.setNamesrvAddr("10.10.5.238:9876");
        // 设置订阅主题和SQL过滤规则
        consumer.subscribe("TopicTest", MessageSelector.bySql("a > 5"));
        consumer.registerMessageListener(
            (MessageListenerConcurrently) (msgs, context) -> {
                String msg = new String(msgs.get(0).getBody(), StandardCharsets.UTF_8);
                System.out.printf(Thread.currentThread().getName() + " Receive New Messages: " + msg + "%n");
                return ConsumeConcurrentlyStatus.CONSUME_SUCCESS;
            });
        consumer.start();
    }
}

```

