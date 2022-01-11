# 定时消息示例

> 定时消息与正常消息的不同之处在于，它是在指定的时间后执行。

## 发送定时消息

- 发送消息的时候设置消息的延迟等级。

```java
import org.apache.rocketmq.client.producer.DefaultMQProducer;
import org.apache.rocketmq.client.producer.SendResult;
import org.apache.rocketmq.common.message.Message;

public class ScheduledMessageProducer {

    public static void main(String[] args) throws Exception {
        DefaultMQProducer producer = new DefaultMQProducer("ScheduledProducerGroup");
        producer.setNamesrvAddr("10.10.5.238:9876");
        producer.start();
        Message message = new Message("TestTopic", "Hello scheduled message".getBytes());
        // 设置延迟等级
        message.setDelayTimeLevel(3);
        SendResult sendResult = producer.send(message);
        System.out.println(sendResult);
        producer.shutdown();
    }

}
```




## 订阅定时消息

```java
import org.apache.rocketmq.client.consumer.DefaultMQPushConsumer;
import org.apache.rocketmq.client.consumer.listener.ConsumeConcurrentlyStatus;
import org.apache.rocketmq.client.consumer.listener.MessageListenerConcurrently;
import org.apache.rocketmq.common.message.MessageExt;

public class ScheduledMessageConsumer {

    public static void main(String[] args) throws Exception {
        DefaultMQPushConsumer consumer = new DefaultMQPushConsumer("ExampleConsumer");
        consumer.subscribe("TestTopic", "*");
        consumer.setNamesrvAddr("10.10.5.238:9876");
        consumer.registerMessageListener((MessageListenerConcurrently) (messages, context) -> {
            for (MessageExt message : messages) {
                System.out.println("Receive message[msgId=" + message.getMsgId() + "] "
                    + (System.currentTimeMillis() - message.getStoreTimestamp()) + "ms later");
            }
            return ConsumeConcurrentlyStatus.CONSUME_SUCCESS;
        });
        consumer.start();
    }
}
```
