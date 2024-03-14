> MQ（Message Queue）消息队列，是一种用来保存消息数据的队列.消息队列作为高并发系统的核心组件之一，能够帮助业务系统解构提升开发效率和系统稳定性

### MQ的主要优势

我们先和普通的项目工程之间的通信做下对比，普通工程之间，通过直接请求来进行系统间的通信，这种普通的通信常常会因为网络或者一方出现问题后导致服务差错。

![img](https://shanzhu-edu.oss-cn-shanghai.aliyuncs.com/image.png)

如果使用MQ作为中间消息传送件，那么将消息生产者和消费者进行隔离了，发送请求和接受请求方并不是主次关系，这样的传送消息的方式有以下特点：

- **削峰填谷：** 主要解决瞬时写压力大于应用服务能力导致消息丢失、系统奔溃等问题
- **系统解耦：** 解决不同重要程度、不同能力级别系统之间依赖导致一死全死
- **提升性能：** 当存在一对多调用时，可以发一条消息给消息系统，让消息系统通知相关系统
- **蓄流压测：** 线上有些链路不好压测，可以通过堆积一定量消息再放开来压测

![img](https://shanzhu-edu.oss-cn-shanghai.aliyuncs.com/image.png)

### MQ的缺点

- **系统可用性降低**：系统引入的外部依赖越多，系统稳定性越差。一旦MQ宕机，就会对业务造成影响。
- **系统复杂性提高**：MQ的加入大大增加了系统的复杂度，以前系统间是同步的远程调用,现在是通过MQ进行异步调用。
- **一致性问题**：A系统处理完业务，通过MQ给B，C，D三个系统发送消息数据，如果B系统，C系统处理成功，D系统处理失败。

### MQ产品介绍

- **ActiveMQ**

  java语言实现，万级数据吞吐量，处理速度ms级，主从架构，**成熟度高**

- **RabbitMQ**

  **erlang语言实现**，万级数据吞吐量，**处理速度us级**，主从架构，

- **RocketMQ**

  java语言实现，**十万级**数据吞吐量，处理速度ms级，分布式架构，功能强大，扩展性强

- **kafka**

  scala语言实现，**十万级**数据吞吐量，处理速度ms级，分布式架构，功能较少，应用于大数据较多

### RocketMQ

RocketMQ是阿里开源的一款非常优秀中间件产品，脱胎于阿里的另一款队列技术MetaQ，后捐赠给**Apache**基金会 作为一款孵化技术，仅仅经历了一年多的时间就成为Apache基金会的顶级项目。并且它现在已经在阿里内部被广泛 的应用，并且经受住了多次双十一的这种极致场景的压力（2017年的双十一，**RocketMQ**流转的消息量达到了万亿 级，峰值TPS达到5600万）

### RocketMQ环境搭建

RockeqMQ

#### linux安装过程

步骤1：上传压缩包（zip）

```Bash
yum -y install lrzsz 
rz
```

步骤2：解压

```Bash
unzip rocketmq-all-4.5.2-bin-release.zip
```

步骤3：修改目录名称

```Bash
mv rocketmq-all-4.5.2-bin-release rocketmq
```

**启动服务器**

步骤1：启动命名服务器（bin目录下）

```Bash
sh mqnamesrv
```

步骤2：启动消息服务器（bin目录下）

```Bash
sh mqbroker -n localhost:9876
```

修改runbroker.sh文件中有关内存的配置（调整的与当前虚拟机内存匹配即可，推荐256m-128m）

#### window安装

**配置环境变量**

系统变量中新增：

变量名：ROCKETMQ_HOME

变量值：MQ解压路径\MQ文件夹名

**启动**

1.启动NAMESERVER

Cmd命令框执行进入至‘MQ文件夹\bin’下 端口9876

```Bash
start mqnamesrv.cmd
```

2.启动BROKER

```Bash
start mqbroker.cmd -n 127.0.0.1:9876 autoCreateTopicEnable=true
```

