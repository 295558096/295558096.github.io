# 简单消息示例

## 可靠的同步传输

### 应用

- 广泛应用于重要通知消息，短信通知，短信营销系统等

### 代码

```java
import org.apache.rocketmq.client.producer.DefaultMQProducer;
import org.apache.rocketmq.client.producer.SendResult;
import org.apache.rocketmq.common.message.Message;
import org.apache.rocketmq.remoting.common.RemotingHelper;

public class SimpleMessageSyncTest {

    public static void main(String[] args) throws Exception {
        // 初始化生产者
        DefaultMQProducer producer = new
            DefaultMQProducer("UNIONPAYSHOP_PRODUCER_GROUP");
        // 设置NameServer
        producer.setNamesrvAddr("10.10.5.238:9876");
        // 启动生产者
        producer.start();
        // 消息体
        String msgBody = ("Hello RocketMQ, IM a Simple MQ");
        // 创建消息
        Message msg = new Message("TopicTest",
            "TagA", msgBody.getBytes(RemotingHelper.DEFAULT_CHARSET));
        // 发送消息
        SendResult sendResult = producer.send(msg);
        // 打印发送结果
        System.out.printf("%s%n", sendResult);
        // 关闭生产者
        producer.shutdown();
    }
}
```



## 可靠的异步传输

### 应用

- 异步传输通常用于响应时间敏感的业务场景。

### 代码

```java
import org.apache.rocketmq.client.producer.DefaultMQProducer;
import org.apache.rocketmq.client.producer.SendCallback;
import org.apache.rocketmq.client.producer.SendResult;
import org.apache.rocketmq.common.message.Message;
import org.apache.rocketmq.remoting.common.RemotingHelper;

public class SimpleMessageASyncTest {

    public static void main(String[] args) throws Exception {
        // 初始化生产者
        DefaultMQProducer producer = new
            DefaultMQProducer("UNIONPAYSHOP_PRODUCER_GROUP");
        // 设置NameServer
        producer.setNamesrvAddr("10.10.5.238:9876");
        // 启动生产者
        producer.start();
        producer.setRetryTimesWhenSendAsyncFailed(0);
        // 消息体
        String msgBody = ("Hello RocketMQ, IM a Simple MQ");
        // 创建消息
        Message msg = new Message("TopicTest",
            "TagA", msgBody.getBytes(RemotingHelper.DEFAULT_CHARSET));
        // 发送消息
        producer.send(msg, new SendCallback() {
            @Override
            public void onSuccess(SendResult sendResult) {
                System.out.println("发送成功");
                System.out.println("SendResult: " + sendResult);
            }

            @Override
            public void onException(Throwable e) {
                System.out.println("发送失败");
                System.out.println("Throwable: " + e);
            }
        });
        //
        System.out.println("异步发送完成");
        // 关闭生产者
        producer.shutdown();
    }
}
```



## 单向传输

### 应用

- 单向传输用于需要中等可靠性的情况，例如日志收集。

### 代码

```java
import org.apache.rocketmq.client.producer.DefaultMQProducer;
import org.apache.rocketmq.common.message.Message;
import org.apache.rocketmq.remoting.common.RemotingHelper;

public class OneWayTest {

    public static void main(String[] args) throws Exception {
        // 初始化生产者
        DefaultMQProducer producer = new
            DefaultMQProducer("UNIONPAYSHOP_PRODUCER_GROUP");
        // 设置NameServer
        producer.setNamesrvAddr("10.10.5.238:9876");
        // 启动生产者
        producer.start();
        // 消息体
        String msgBody = ("Hello RocketMQ, IM a Simple MQ");
        // 创建消息
        Message msg = new Message("TopicTest",
            "TagA", msgBody.getBytes(RemotingHelper.DEFAULT_CHARSET));
        // 发送消息
        producer.sendOneway(msg);
        // 打印发送结果
        System.out.println("发送完成");
        // 关闭生产者
        producer.shutdown();
    }
}
```

