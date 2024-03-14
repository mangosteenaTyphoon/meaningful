## JMS介绍

JMS即Java Message Service(Java消息服务)，是Java专⻔为消息服务定义的⼀套规范。Activemq充分遵循了JMS规范。在JMS中定义了⼀系列的成员来描述这套规范，⽐如Connection Factory、Connection、Session、Producer、Consumer等等。JMS约定了使⽤消息服务的多⽅，进⾏消息通信的步骤。

## JMS开发的基本步骤

![image-20230220111641233](https://shanzhu-edu.oss-cn-shanghai.aliyuncs.com/blog/202302201116404.png)

- Producer：消息的⽣产者，发送消息的⼀⽅。

- Consumer：消息的消费者，订阅消息的⼀⽅。

- Connection：描述连接的对象，⽣产者/消费者连接消息队列时需要通过Connection对象。

- ConnectionFactory：⽣产连接对象的⼯⼚

- Session：连接后的⼀次会话，⽣产者/消费者与消息队列之间的会话。

- Message：描述消息的内容

- Destination：⽬的地，泛指⽣产者消息发送的⽬的地，和消费者订阅消息的来源地。

  ​	在activemq中，destination有两者具体的实现：

  - queue：队列，某条消息只能被某⼀个消费者订阅
  - topic：主题，某条消息可以同时被多个消费者订阅

在JMS约定的消息服务中，需要由以上⼏个部分相互配合、协作，完成消息服务中消息的发送和接收。

## JMS消息服务的实现(队列模式)

### 消费生产者

#### 引入依赖

```xml
<!--activemq的jar包-->
<dependency>
    <groupId>org.apache.activemq</groupId>
    <artifactId>activemq-all</artifactId>
    <version>5.15.14</version>
</dependency>

<!--activemq和spring的整合包 -->
<dependency>
    <groupId>org.apache.xbean</groupId>
    <artifactId>xbean-spring</artifactId>
    <version>3.16</version>
</dependency>
```

#### 编写生产者程序

```JAVA
public class ProducerDemo {
    public static final String URL="tcp://81.68.201.126:61616";

    public static final String QUEUE_NAME = "shanzhuQueue1";

    public static void main(String[] args) throws JMSException {
        //创建连接到activemq服务器的连接⼯⼚
        ActiveMQConnectionFactory connectionFactory = new ActiveMQConnectionFactory(URL);
        //创建连接对话
        Connection connection = connectionFactory.createConnection();
        //开启连接
        connection.start();
        //从连接对象中获取一个session
        Session session = connection.createSession(false, Session.AUTO_ACKNOWLEDGE);
        //指明Queue目的地，获取queue对象
        Queue queue = session.createQueue(QUEUE_NAME);
        //创建生产者
        MessageProducer producer = session.createProducer(queue);
        //创建消息对象
        TextMessage message = session.createTextMessage("发送消息");
        producer.send(message);
        System.out.println("消息已经发送~");
        //关闭连接
        connection.stop();
    }
}
```

在控制台查看消息情况

![image-20230220165104491](https://shanzhu-edu.oss-cn-shanghai.aliyuncs.com/blog/202302201651663.png)

### 消息消费者

同样是这个两个依赖，进行引入

~~~xml
<!--activemq的jar包-->
<dependency>
    <groupId>org.apache.activemq</groupId>
    <artifactId>activemq-all</artifactId>
    <version>5.15.14</version>
</dependency>

<!--activemq和spring的整合包 -->
<dependency>
    <groupId>org.apache.xbean</groupId>
    <artifactId>xbean-spring</artifactId>
    <version>3.16</version>
</dependency>
~~~

#### 编写消费者程序

```JAVA
public class ConsumerDemo1 {
    public static final String URL="tcp://81.68.201.126:61616";
    public static final String QUEUE_NAME="shanzhuQueue1";

    public static void main(String[] args) throws JMSException {
        ActiveMQConnectionFactory factory = new ActiveMQConnectionFactory(URL);
        Connection connection = factory.createConnection();
        connection.start();
        Session session = connection.createSession(false, Session.AUTO_ACKNOWLEDGE);
        Queue queue = session.createQueue(QUEUE_NAME);
        MessageConsumer consumer = session.createConsumer(queue);
        Message receive = consumer.receive();
        System.out.println("接收到了消息"+receive);
    }

}
```

#### 查看控制台

![image-20230220165502537](https://shanzhu-edu.oss-cn-shanghai.aliyuncs.com/blog/202302201655641.png)

控制台有3台消息被消费

## 消费者的同步阻塞方式

消费者的receive⽅式即为同步阻塞⽅式来接收消息。所谓的同步阻塞就是消费者获得消息后才能执⾏之后的业务逻辑，在接收到消息之前处于阻塞等待的状态。receive重载了两个⽅法：

- 一直阻塞等待

  ~~~java
  while(true){
      TextMessagemessage= (TextMessage)consumer.receive();
  	if(message!=null){
          System.out.println("接收到的消息："+message.getText());
      }else{
          System.out.println("已完成接收");            
      }
  }
  ~~~

  其中consumer.receive()⽅法在收到消息之前，⼀直处于阻塞等待，之后的代码不会执⾏。直到收到消息后，才会执⾏之后的代码。这里可以使用另一个方法，阻塞等待指定时间。

  

- 阻塞等待时间

  ~~~java
  
  while(true){
      TextMessagemessage= (TextMessage)consumer.receive(3000L);
  	if(message!=null){
          System.out.println("接收到的消息："+message.getText());
      }else{
          System.out.println("已完成接收");            
      }
  }
  ~~~

  指定等待时间为3000毫秒，在该时间内收到消息，则离开阻塞状态，执⾏之后的代码。如果等待3000毫秒都没收到消息，则直接离开阻塞状态，执⾏之后的代码。

  

  ## 消费异步阻塞方式

  消费者通过设置监听器的⽅式监听队列是否有消息。当有消息则监听器⼯作，消费者处理消息，这个过程是异步的，消费者不会因为等待消息⽽阻塞。

  ```JAVA
  public class ConsumerDemo2 {
      public static final String URL="tcp://81.68.201.126:61616";
      public static final String QUEUE_NAME="shanzhuQueue1";
  
      public static void main(String[] args) throws JMSException {
          ActiveMQConnectionFactory factory = new ActiveMQConnectionFactory(URL);
          Connection connection = factory.createConnection();
          connection.start();
          Session session = connection.createSession(false, Session.AUTO_ACKNOWLEDGE);
          Queue queue = session.createQueue(QUEUE_NAME);
          MessageConsumer consumer = session.createConsumer(queue);
          consumer.setMessageListener(message -> {
              if(Objects.nonNull(message)&& message instanceof TextMessage){
                  TextMessage textMessage= (TextMessage) message;
                  try {
                      String text = textMessage.getText();
                      System.out.println(text);
                  }catch (JMSException e){
                     e.printStackTrace();
                  }
              }
          });
  
          System.out.println("这里还有别的工作~");
  
      }
  }
  ```

  ## 队列模式的特点

  队列模式的特点是1条消息只能被1个消费者消费，换句话说，已经被消费的消息不会再次被消费。从以下两种场景可以验证该特点：

- 后加⼊的消费者：先启动⽣产者，再启动两个消费者，只会有⼀个消费者能消费到全部消息，因为在当前配置下，消息不会被重复消费。
- 两个消费者消费同⼀个队列：先启动两个消费者，再启动⽣产者发消息，此时多条消息会被两个消费者均摊消费。

## Topic模式

之前的例⼦使⽤的是Queue队列模式。在队列模式中，⼀条消息只能被⼀个消费者消费。同时，这种点对点的发送和消费，运⾏消费者启动后能够消费到消费者没启动之前的，队列中的消息。

topic也称之为主题，即允许多个消费者消费同⼀个主题。也就是说，多个消费者能接收到同⼀个主题中的消息。相当于是，⼀条消息，会被多个消费者消费。下⾯我们来体验这种效果：

### 编写生产者

```JAVA
public class ProducerDemo1 {
    public static final String URL="tcp://81.68.201.126:61616";

    public static final String TOPIC_NAME = "shanzhuTopic";

    public static void main(String[] args) throws JMSException {
        //创建连接到activemq服务器的连接⼯⼚
        ActiveMQConnectionFactory connectionFactory = new ActiveMQConnectionFactory(URL);
        //创建连接对话
        Connection connection = connectionFactory.createConnection();
        //开启连接
        connection.start();
        //从连接对象中获取一个session
        Session session = connection.createSession(false, Session.AUTO_ACKNOWLEDGE);
        //指明Queue目的地，获取queue对象
        Topic topic = session.createTopic(TOPIC_NAME);
        //创建生产者
        MessageProducer producer = session.createProducer(topic);
        //创建消息对象

        for (int i = 0; i < 10; i++) {
            TextMessage message = session.createTextMessage("发送消息"+i);
            producer.send(message);
        }
        System.out.println("消息已经发送~");
        //关闭连接
        connection.stop();
    }

}
```

### 编写消费者

```JAVA
public class ConsumerDemo1 {
    public static final String URL="tcp://81.68.201.126:61616";
    public static final String TOPIC_NAME="shanzhuTopic";


    public static void main(String[] args) throws JMSException {
        ActiveMQConnectionFactory factory = new ActiveMQConnectionFactory(URL);
        Connection connection = factory.createConnection();
        connection.start();
        Session session = connection.createSession(false, Session.AUTO_ACKNOWLEDGE);
        Topic topic = session.createTopic(TOPIC_NAME);
        MessageConsumer consumer = session.createConsumer(topic);
        consumer.setMessageListener(message -> {
            if(Objects.nonNull(message)&& message instanceof TextMessage){
                TextMessage textMessage= (TextMessage) message;
                try {
                    String text = textMessage.getText();
                    System.out.println(text);
                }catch (JMSException e){
                    e.printStackTrace();
                }
            }
        });
    }
}
```

启动多个消费者实例，发现多个消费者都可以收到该主题的消息。同时会发现，topic模式下，消费者没有办法收到“订阅”动作发⽣之前的消息。这就好⽐订阅⼀份报纸，只能在订阅之⽇起开始才会收到每天的报纸。

## JMSMessage的组成部分

### JMSMessage介绍

JMSMessage是描述消息的规范，包含以下⼏个部分：

- 消息头：描述消息的关键信息
- 消息体：承载具体的消息内容消
- 息属性：描述消息的额外信息

### 消息头

JMSMessage中允许消息头包含以下内容：

- JMSDestination

​	在消息中指定消息的⽬的地。除了之前的同⼀设置会话的⽬的地外，也可以为某⼀条消息指定⽬的地。

~~~JAVA
//创建⽬的地：队列名称
Topictopic=session.createTopic(TOPIC_NAME);
//创建发送消息到指定队列的⽣产者
MessageProducerproducer=session.createProducer(topic);
//创建消息对象
TextMessagetextMessage=
session.createTextMessage("hello activemq");
//指明消息的⽬的地
textMessage.setJMSDestination(topic);
~~~

- JMSDeliveryMode

  设置消息的持久化模式。注意，这⾥是消息的持久化模式。在服务器宕机重启后，持久化消息还会被消费，但⾮持久化消息会丢失。

- JMSExpiration

  设置消息的超时时间，如果发送后，在消息过期时间之后还没有被发送到⽬的地，则该消息被清除。默认是消息永不过期。

- JMSPriority

  设置消息的优先级，优先级越⾼的消息不⼀定会优先达到，但是⾼优先级（5-9）的消息⼀定会早于默认优先级（4）的消息优先达到。

- JMSMessageID

  描述消息的唯⼀id，⽤于标识某⼀条消息。

### 消息体

JMSMessage提供了五种Message：

- TextMessage ：⽤字符串描述⼀条消息。
- MapMessage：⽤键值对描述⼀条消息。
- BytesMessage：⽤byte数组描述⼀条消息。
- StreamMessage：⽤Java数据流来操作消息的填充和读取。
- ObjectMessage：⽤可序列化的Java对象描述消息。

注意，这五种消息格式的使⽤，⽣产者和消费者需要⼀致。

### 消息属性

消息属性⽤于描述消息的额外信息。这个额外信息可以理解为⽤于区分消息的信息，⽐如给消息打标签，给消息分类等等，都可以使⽤消息属性来设置。

~~~java
//设置消息属性
 textMessage.setStringProperty("author","Thor");
~~~

通过键值对的形式设置消息属性。⽐如设置该消息的作者的键值对。在接收消息时可以对消息做检验。

~~~java
String author = textMessage.getStringProperty("author");
System.out.println("author:"+author);
~~~

