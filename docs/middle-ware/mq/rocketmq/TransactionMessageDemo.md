# 事务消息示例

## 概念

- 事务消息可以被认为是一个两阶段的提交消息实现，以确保分布式系统的**最终一致性**。
- 事务性消息确保本地事务的执行和消息的发送可以原子地执行。

## 使用限制

- 事务性消息没有定时和批处理支持。
- 为避免单条消息被检查次数过多，导致消息堆积到一半，我们将单条消息的检查次数默认限制为15次，但用户可以通过更改`transactionCheckMax`来更改。
- 如果一条消息的检查次数超过`transactionCheckMax`次，broker默认会丢弃这条消息，同时打印错误日志。
- 用户可以通过重写`AbstractTransactionCheckListener`类来改变这种行为。

- 事务消息将在一定时间后检查，该时间由代理配置中的参数`transactionTimeout`确定。
- 用户可以在发送事务消息时通过设置用户属性`CHECK_IMMUNITY_TIME_IN_SECONDS`来改变这个限制，这个参数优先于`transactionMsgTimeout`参数。
- **一个事务性消息可能会被检查或消费不止一次。**
-  `Committed` 给用户目标Topic的消息reput可能会失败。
- 事务消息的高可用是由 RocketMQ 本身的高可用机制来保证的。如果要保证事务消息不丢失，保证事务完整性，推荐使用同步双写机制。
- 事务性消息的生产者 ID 不能与其他类型消息的生产者 ID 共享。
- 与其他类型的消息不同，事务性消息允许向后查询。MQ 服务器通过其生产者 ID 查询客户端。



## 事务状态

- `TransactionStatus.CommitTransaction` 

  提交事务，表示允许消费者消费该消息。

- `TransactionStatus.RollbackTransaction`

  回滚事务，表示该消息将被删除，不允许消费。

- `TransactionStatus.Unknown`

  中间状态，表示需要MQ回查才能确定状态。

## 发送事务信息

- 使用`TransactionMQProducer`类创建生产者客户端，并指定唯一的`producerGroup`。
- 可以设置**自定义线程池**来处理检查请求。
- 本地事务执行后，需要根据执行结果回复MQ。
- 如果本地事务直接提交消息，则不进行回查。

```java
import java.io.UnsupportedEncodingException;
import java.util.concurrent.ArrayBlockingQueue;
import java.util.concurrent.ExecutorService;
import java.util.concurrent.ThreadPoolExecutor;
import java.util.concurrent.TimeUnit;
import org.apache.rocketmq.client.exception.MQClientException;
import org.apache.rocketmq.client.producer.LocalTransactionState;
import org.apache.rocketmq.client.producer.SendResult;
import org.apache.rocketmq.client.producer.TransactionListener;
import org.apache.rocketmq.client.producer.TransactionMQProducer;
import org.apache.rocketmq.common.message.Message;
import org.apache.rocketmq.common.message.MessageExt;
import org.apache.rocketmq.remoting.common.RemotingHelper;

public class TransactionProducer {

    public static void main(String[] args)
        throws MQClientException, InterruptedException, UnsupportedEncodingException {
        TransactionListener transactionListener = new TransactionListener() {
            @Override
            public LocalTransactionState executeLocalTransaction(Message msg, Object arg) {
                System.out.println(System.currentTimeMillis());
                System.out.println(new String(msg.getBody()));
                System.out.println("executeLocalTransaction..");
                return LocalTransactionState.UNKNOW;
            }
            @Override
            public LocalTransactionState checkLocalTransaction(MessageExt msg) {
                System.out.println(System.currentTimeMillis());
                System.out.println(new String(msg.getBody()));
                System.out.println("checkLocalTransaction..");
                return LocalTransactionState.COMMIT_MESSAGE;
            }
        };
        TransactionMQProducer producer = new TransactionMQProducer("TransactionProducerGroup");
        ExecutorService executorService = new ThreadPoolExecutor(2, 5, 100, TimeUnit.SECONDS,
            new ArrayBlockingQueue<>(2000), r -> {
            Thread thread = new Thread(r);
            thread.setName("client-transaction-msg-check-thread");
            return thread;
        });
        producer.setExecutorService(executorService);
        producer.setTransactionListener(transactionListener);
        producer.setNamesrvAddr("10.10.5.238:9876");
        producer.start();
        Message msg =
            new Message("TopicTest", "TagE", "KEY",
                ("Hello Transaction RocketMQ 23333").getBytes(RemotingHelper.DEFAULT_CHARSET));
        SendResult sendResult = producer.sendMessageInTransaction(msg, null);
        System.out.printf("%s%n", sendResult);
        Thread.sleep(1000000);
        producer.shutdown();
    }
}
```