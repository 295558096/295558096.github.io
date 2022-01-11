# 批量消息示例



## 优势

- 批量发送消息可提高单次发送消息的性能。

## 使用限制

- 相同批次的消息应具有相同的主题，相同的等待消息处理成功。
- 批量消息不支持定时处理。
- 一个批量的消息的总大小不要超过1MB。



## 发送批量消息

```java
import java.util.ArrayList;
import java.util.List;
import org.apache.rocketmq.client.producer.DefaultMQProducer;
import org.apache.rocketmq.client.producer.SendResult;
import org.apache.rocketmq.common.message.Message;
import org.apache.rocketmq.remoting.common.RemotingHelper;

public class BatchMessageProducer {

    public static void main(String[] args) throws Exception {
        DefaultMQProducer producer = new DefaultMQProducer("ProducerGroupName");
        // 设置NaseServer
        producer.setNamesrvAddr("10.10.5.238:9876");
        producer.start();
        List<Message> msgs = new ArrayList<>();
        for (int i = 0; i < 10; i++) {
            Message msg = new Message("TopicTest",
                "TagA",
                "OrderID188" + i,
                "Hello world".getBytes(RemotingHelper.DEFAULT_CHARSET));
            msgs.add(msg);
        }
        SendResult sendResult = producer.send(msgs);
        System.out.printf("%s%n", sendResult);
        producer.shutdown();
    }
}
```

## 分割消息

- 避免批量消息大于1M。

### 分割器

```java
import java.util.Iterator;
import java.util.List;
import java.util.Map;
import org.apache.rocketmq.common.message.Message;

public class ListSplitter implements Iterator<List<Message>> {

    private final int SIZE_LIMIT = 1000 * 1000;
    private final List<Message> messages;
    private int currIndex;

    public ListSplitter(List<Message> messages) {
        this.messages = messages;
    }

    @Override
    public boolean hasNext() {
        return currIndex < messages.size();
    }

    @Override
    public List<Message> next() {
        int nextIndex = currIndex;
        int totalSize = 0;
        for (; nextIndex < messages.size(); nextIndex++) {
            Message message = messages.get(nextIndex);
            int tmpSize = message.getTopic().length() + message.getBody().length;
            Map<String, String> properties = message.getProperties();
            for (Map.Entry<String, String> entry : properties.entrySet()) {
                tmpSize += entry.getKey().length() + entry.getValue().length();
            }
            tmpSize = tmpSize + 20;
            if (tmpSize > SIZE_LIMIT) {
                if (nextIndex - currIndex == 0) {
                    nextIndex++;
                }
                break;
            }
            if (tmpSize + totalSize > SIZE_LIMIT) {
                break;
            } else {
                totalSize += tmpSize;
            }
        }
        List<Message> subList = messages.subList(currIndex, nextIndex);
        currIndex = nextIndex;
        return subList;
    }
}
```

### 客户端调用

```java
public static void main(String[] args) throws UnsupportedEncodingException {
    List<Message> msgs = new ArrayList<>();
    for (int i = 0; i < 10; i++) {
        Message msg = new Message("TopicTest",
            "TagA",
            "OrderID188" + i,
            "Hello world".getBytes(RemotingHelper.DEFAULT_CHARSET));
        msgs.add(msg);
    }
    DefaultMQProducer producer = new DefaultMQProducer("BatchProducerGroup");
    producer.setNamesrvAddr("10.10.5.238:9876");
    ListSplitter splitter = new ListSplitter(msgs);
    while (splitter.hasNext()) {
        try {
            List<Message> listItem = splitter.next();
            producer.send(listItem);
        } catch (Exception e) {
            e.printStackTrace();
        }
    }
}
```

