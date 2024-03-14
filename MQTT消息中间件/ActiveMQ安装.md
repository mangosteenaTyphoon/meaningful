### ActiveMQ的介绍

Apache ActiveMQ是最流⾏的开源、多协议、基于 Java 的消息代理。它⽀持⾏业标准协议，因此⽤户可以从各种语⾔和平台的客户端选择中受益。从用JavaScript、C、C++、Python、.Net 等编写的客户端连接。使⽤⽆处不在的AMQP协议集成多平台应⽤程序。使⽤STOMP over websockets在 Web 应⽤程序之间交换消息。使⽤MQTT管理您的 IoT 设备。⽀持您现有的JMS基础架构及其他基础架构。ActiveMQ 提供了⽀持任何消息传递⽤例的能⼒和灵活性。

### ActiveMQ的安装

安装之前我们必须配置好JDK环境，这里不多做解释，先创建一个文件夹activemq

~~~sh
mkdir activemq
~~~

将下载好的安装包进行解压

~~~sh
tar -zxvf apache-activemq-5.15.14-bin.tar.gz
~~~

进入到解压好的文件bin目录中并执行

~~~sh
./activemq start
~~~

![image-20230131233336913](C:\Users\Admin\AppData\Roaming\Typora\typora-user-images\image-20230131233336913.png)

至此，activemq快速启动成功了。

### 两种启动方式

普通启动

~~~sh
./activemq start
~~~

启动时指明文件

~~~sh
./activemq start > ../data/log.log
~~~

启动⽇志将会记录在data路径内的log.log中

### 验证activemq的服务状态

执⾏完activemq的启动命令后，如何得知服务的执⾏情况。

#### 查看服务状态

~~~sh
./activemq status
~~~

#### 查看activemq进程信息

~~~sh
ps -ef | grep activemq
~~~

### 查看activemq控制台

activemq提供了控制台界⾯，访问服务器的8161端⼝即可。8161端⼝为activemq控制台⻚⾯的固定端⼝。默认的登录账号：admin，默认的登录密码：admin。

### 停止activemq服务

~~~sh
./activemq stop
~~~
