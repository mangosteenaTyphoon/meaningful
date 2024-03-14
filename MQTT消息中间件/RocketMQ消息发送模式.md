> 这篇文章主要来记录一下RocketMQ消息发送模式。

### 消息发送模式

#### 单生产者与单消费者模式

引入依赖

```XML
<dependency>
    <groupId>org.apache.rocketmq</groupId>
    <artifactId>rocketmq-client</artifactId>
    <version>4.8.0</version>
</dependency>
```

编写生产者

```Java
import org.apache.rocketmq.client.exception.MQClientException;
import org.apache.rocketmq.client.producer.DefaultMQProducer;
import org.apache.rocketmq.client.producer.SendResult;
import org.apache.rocketmq.common.message.Message;

public class Producer {
    public static void main(String[] args) throws Exception {
    

        //1.创建一个发送消息的对象Producer
        DefaultMQProducer producer = new DefaultMQProducer("group1");
        //2.设定发送的命名服务器地址
        producer.setNamesrvAddr("localhost:9876");
        //3.1启动发送的服务
        producer.start();
        //4.创建要发送的消息对象,指定topic，指定内容body
        Message msg = new Message("topic1", "hello rocketmq".getBytes("UTF-8"));
        //3.2发送消息
        SendResult result = producer.send(msg);
        System.out.println("返回结果：" + result);
        //5.关闭连接
        producer.shutdown();
    }
}
```

编写消费者

```Java
import org.apache.rocketmq.client.consumer.DefaultMQPushConsumer;
import org.apache.rocketmq.client.consumer.listener.ConsumeConcurrentlyContext;
import org.apache.rocketmq.client.consumer.listener.ConsumeConcurrentlyStatus;
import org.apache.rocketmq.client.consumer.listener.MessageListenerConcurrently;
import org.apache.rocketmq.client.exception.MQClientException;
import org.apache.rocketmq.common.message.MessageExt;

import java.util.List;

public class Consumer {
    public static void main(String[] args) throws Exception {
        //1.创建一个接收消息的对象Consumer
        DefaultMQPushConsumer consumer = new DefaultMQPushConsumer("group1");
        //2.设定接收的命名服务器地址
        consumer.setNamesrvAddr("localhost:9876");
        //3.设置接收消息对应的topic,对应的sub标签为任意
        consumer.subscribe("topic1","*");
        //3.开启监听，用于接收消息
        consumer.registerMessageListener(new MessageListenerConcurrently() {
            @Override
            public ConsumeConcurrentlyStatus consumeMessage(List<MessageExt> list, ConsumeConcurrentlyContext consumeConcurrentlyContext) {
                //遍历消息
                for (MessageExt msg : list) {
                    System.out.println("收到消息："+msg);
                }
                return ConsumeConcurrentlyStatus.CONSUME_SUCCESS;
            }
        });
        //4.启动接收消息的服务
        consumer.start();
        System.out.println("接受消息服务已经开启！");
        //5 不要关闭消费者！
    }
}
```

#### 单生产者多消费者消息发送（OneToMany）

生产者

```Java
  //1.创建一个发送消息的对象Producer
        DefaultMQProducer producer = new DefaultMQProducer("group1");
        //2.设定发送的命名服务器地址
        producer.setNamesrvAddr("localhost:9876");
        //3.1启动发送的服务
        producer.start();
        for (int i = 0; i < 10; i++) {
            //4.创建要发送的消息对象,指定topic，指定内容body
            Message msg = new Message("topic1", ("hello rocketmq"+i).getBytes("UTF-8"));
            //3.2发送消息
            SendResult result = producer.send(msg);
            System.out.println("返回结果：" + result);
        }
        //5.关闭连接
        producer.shutdown();
```