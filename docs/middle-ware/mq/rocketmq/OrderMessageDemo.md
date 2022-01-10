# 顺序消息示例

## 有序消息定义

- RocketMQ使用FIFO队列提供有序消息。

## 发送有序消息

- 一个 Topic 下默认创建4个队列结束消息。

```java
public class OrderedProducer {

    public static void main(String[] args) throws Exception {
        // 设置生产者组信息
        DefaultMQProducer producer = new DefaultMQProducer("unionPayShop");
        // 设置NameServer
        producer.setNamesrvAddr("10.10.5.238:9876");
        producer.start();
        int orderId = 334455;
        // 构建消息
        Message msg = new Message("TopicTestjjj", "TagA", "KEY" + orderId,
            ("Hello RocketMQ " + orderId).getBytes(RemotingHelper.DEFAULT_CHARSET));
        // 指定消息选择器发送消息
        SendResult sendResult = producer.send(msg, (msgSelectors, msgInfo, arg) -> msgSelectors.get(0), orderId);
        // 消息发送结果
        System.out.printf("%s%n", sendResult);
        // 关闭生产者
        producer.shutdown();
    }
}
```

## 订阅有序消息

- 注册顺序监听器。
- 消费顺序和发送消息顺序严格一致。

```java
public class OrderedConsumer {

    public static void main(String[] args) throws Exception {

        // 创建推送模式消费者
        DefaultMQPushConsumer consumer = new DefaultMQPushConsumer("unionPayShopConsumer");

        // 设置NaseServer
        consumer.setNamesrvAddr("10.10.5.238:9876");

        // 消费起始点
        consumer.setConsumeFromWhere(ConsumeFromWhere.CONSUME_FROM_FIRST_OFFSET);

        // 订阅的消息，tag
        consumer.subscribe("TopicTestjjj", "TagA");

        // 注册顺序消息监听器，默认是并发
        consumer.registerMessageListener((MessageListenerOrderly) (msgs, context) -> {
            context.setAutoCommit(false);
            String orderId = new String(msgs.get(0).getBody(), StandardCharsets.UTF_8);
            System.out.printf(Thread.currentThread().getName() + " Receive New Messages: " + orderId + "%n");
            return ConsumeOrderlyStatus.SUCCESS;
        });
        consumer.start();
        System.out.printf("Consumer Started.%n");
    }
}
```

