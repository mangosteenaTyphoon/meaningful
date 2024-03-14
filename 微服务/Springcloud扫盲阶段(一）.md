> 随着互联网行业的发展，对服务的要求也越来越高，服务架构从单体架构逐渐演变成了现在流行的微服务架构。以此文记在跟着黑马学习springcloud阶段的扫盲笔记。

## 微服务是什么

微服务就是将单体架构中的各个模块进行拆分，比如说一个电商系统，可以把订单模块和登录模块进行拆分成很多单体架构，这样做的好处就是大大降低了各个模块之间的耦合度，一个模块的宕机并不会影响其他模块的运行，一个模块的更新也不会影响到整个微服务系统的整体运行，而且分布式部署的话，每个服务器只支持一个模块的运行，服务器的稳定性以及性能也会大大提升。他的缺点也显而易见，整体架构比较复杂，部署成本高。

![img](https://shanzhu-edu.oss-cn-shanghai.aliyuncs.com/202212290902424.png)

### 微服务的架构特征

- 单一职责：微服务拆分粒度更小，每一个服务都对应唯一的业务能力，做到单一职责
- 自治：团队独立、技术独立、数据独立，独立部署和交付
- 面向服务：服务提供统一标准的接口，与语言和技术无关
- 隔离性强：服务调用做好隔离、容错、降级，避免出现级联问题

微服务的上述特性其实是在给分布式架构制定一个标准，进一步降低服务之间的耦合度，提供服务的独立性和灵活性。做到高内聚，低耦合。因此，可以认为微服务是一种经过良好架构设计的分布式架构方案 。但方案该怎么落地？选用什么样的技术栈？全球的互联网公司都在积极尝试自己的微服务落地方案。**其中在Java领域最引人注目的就是SpringCloud提供的方案了。**千万不要认为微服务就是Springcloud，这是令人可笑的认知。

## 认识Springcloud

SpringCloud是目前国内使用最广泛的微服务框架。官网地址：https://spring.io/projects/spring-cloud。

SpringCloud集成了各种微服务功能组件，并基于SpringBoot实现了这些组件的自动装配，从而提供了良好的开箱即用体验。

常见组件：

![img](https://shanzhu-edu.oss-cn-shanghai.aliyuncs.com/202212290902415.png)

另外Springcloud底层是依赖于SpringBoot的，并且有版本兼容关系。在运行相关项目时，可以去官网进行查询相关版本。

## 服务拆分和远程调用

任何分布式架构都离不开服务的拆分，微服务也是一样。

### 服务拆分原则

这里我总结了微服务拆分时的几个原则：

- 不同微服务，不要重复开发相同业务
- 微服务数据独立，不要访问其它微服务的数据库
- 微服务可以将自己的业务暴露为接口，供其它微服务调用

![img](https://shanzhu-edu.oss-cn-shanghai.aliyuncs.com/202212290902407.png)

### 服务拆分示例

以此代码为例。我将代码上传到了gitee：https://gitee.com/shanzhu2/springcloud_demo.git

父工程进行管理依赖

- order-service：订单微服务，负责订单相关业务
- user-service：用户微服务，负责用户相关业务

要求：

- 订单微服务和用户微服务都必须有各自的数据库，相互独立
- 订单服务和用户服务都对外暴露Restful的接口
- 订单服务如果需要查询用户信息，只能调用用户服务的Restful接口，不能查询用户数据库

### 关于RestTemplate的小技巧

如果我们要使用SpringBoot处理其他服务的接口时，我们可以使用RestTemplate，他可以帮我们解析接口数据。

使用步骤如下：

- 注册一个RestTemplate的实例到Spring容器
- 修改order-service服务中的OrderService类中的queryOrderById方法，根据Order对象中的userId查询User
- 将查询的User填充到Order对象，一起返回

#### 注册[RestTemplate](https://so.csdn.net/so/search?q=RestTemplate&spm=1001.2101.3001.7020)

首先，我们在order-service服务中的OrderApplication启动类中，注册RestTemplate实例：

```Java
package cn.itcast.order;

import org.mybatis.spring.annotation.MapperScan;
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.context.annotation.Bean;
import org.springframework.web.client.RestTemplate;

@MapperScan("cn.itcast.order.mapper")
@SpringBootApplication
public class OrderApplication {

    public static void main(String[] args) {
        SpringApplication.run(OrderApplication.class, args);
    }

    @Bean
    public RestTemplate restTemplate() {
        return new RestTemplate();
    }
}
```

#### 使用RestTemplate

```Java
  public Order queryOrderById(Long orderId) {
        // 1.查询订单
        Order order = orderMapper.findById(orderId);

        String url="http://localhost:8081/user/"+order.getUserId();
        System.out.println();
        User user = restTemplate.getForObject(url, User.class);
        order.setUser(user);
        // 4.返回
        return order;
    }
```

### 提供者与消费者

在服务调用关系中，会有两个不同的角色：

服务提供者：一次业务中，被其它微服务调用的服务。（提供接口给其它微服务）

服务消费者：一次业务中，调用其它微服务的服务。（调用其它微服务提供的接口）

![img](https://shanzhu-edu.oss-cn-shanghai.aliyuncs.com/202212290902381.png)

但是，服务提供者与服务消费者的角色并不是绝对的，而是相对于业务而言。

如果服务A调用了服务B，而服务B又调用了服务C，服务B的角色是什么？

对于A调用B的业务而言：A是服务消费者，B是服务提供者 对于B调用C的业务而言：B是服务消费者，C是服务提供者 因此，服务B既可以是服务提供者，也可以是服务消费者。

## Eureka注册中心

假如我们的服务提供者user-service部署了多个实例，如图：

![img](https://shanzhu-edu.oss-cn-shanghai.aliyuncs.com/202212290902388.png)

大家思考几个问题：

- order-service在发起远程调用的时候，该如何得知user-service实例的ip地址和端口？
- 有多个user-service实例地址，order-service调用时该如何选择？
- order-service如何得知某个user-service实例是否依然健康，是不是已经宕机？

### Euerka的结构和作用

这些问题都需要利用SpringCloud中的注册中心来解决，其中最广为人知的注册中心就是Eureka，其结构如下：

![img](https://shanzhu-edu.oss-cn-shanghai.aliyuncs.com/202212290902437.png)

回答之前的各个问题。

问题1：order-service如何得知user-service实例地址？

获取地址信息的流程如下：

- user-service服务实例启动后，将自己的信息注册到eureka-server（Eureka服务端）。这个叫服务注册
- eureka-server保存服务名称到服务实例地址列表的映射关系
- order-service根据服务名称，拉取实例地址列表。这个叫服务发现或服务拉取

问题2：order-service如何从多个user-service实例中选择具体的实例？

- order-service从实例列表中利用负载均衡算法选中一个实例地址
- 向该实例地址发起远程调用

问题3：order-service如何得知某个user-service实例是否依然健康，是不是已经宕机？

- user-service会每隔一段时间（默认30秒）向eureka-server发起请求，报告自己状态，称为心跳
- 当超过一定时间没有发送心跳时，eureka-server会认为微服务实例故障，将该实例从服务列表中剔除
- order-service拉取服务时，就能将故障实例排除了

**注意：一个微服务，既可以是服务提供者，又可以是服务消费者，因此eureka将服务注册、服务发现等功能统一封装到了eureka-client端**

因此接下来我们需要自己搭建一个Eureka中心

![img](https://shanzhu-edu.oss-cn-shanghai.aliyuncs.com/202212290902962.png)

### 搭建eureka-server

首先大家注册中心服务端：eureka-server，这必须是一个独立的微服务

在父工程下，创建一个子模块：

#### **引入相关依赖**

```XML
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-netflix-eureka-server</artifactId>
</dependency>
```

#### **编写启动类**

给eureka-server服务编写一个启动类，一定要添加一个@EnableEurekaServer注解，开启eureka的注册中心功能：

```Java
package cn.itcast.eureka;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.cloud.netflix.eureka.server.EnableEurekaServer;

@SpringBootApplication
@EnableEurekaServer
public class EurekaApplication {
    public static void main(String[] args) {
        SpringApplication.run(EurekaApplication.class, args);
    }
}
```

#### **编写配置文件**

编写一个application.yml文件，内容如下：

```YAML
server:
  port: 10086
spring:
  application:
    name: eureka-server
eureka:
  client:
    service-url: 
      defaultZone: http://127.0.0.1:10086/eureka
  instance:
    prefer-ip-address: true
    instance-id: 127.0.0.1:${server.port}    
```

#### **启动服务**

启动微服务，然后在浏览器访问：http://127.0.0.1:10086

![img](https://shanzhu-edu.oss-cn-shanghai.aliyuncs.com/202212290902998.png)

### 服务注册

下面，我们将user-service注册到eureka-server中去。

#### **引入依赖**

在user-service的pom文件中，引入下面的eureka-client依赖：

```XML
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
</dependency>
```

引入依赖后记得在启动类加上注解`@EnableEurekaClient`

#### **配置文件**

在user-service中，修改application.yml文件，添加服务名称、eureka地址：

```YAML
spring:
  application:
    name: userservice
eureka:
  client:
    service-url:
      defaultZone: http://127.0.0.1:10086/eureka
  instance:
    prefer-ip-address: true #以IP地址注册到服务中心，相互注册使用IP地址,如果不配置就是机器的主机名
    instance-id: 127.0.0.1:${server.port}    # instanceID默认值为主机名+服务名+端口
```

查看eureka-server管理页面

![img](https://shanzhu-edu.oss-cn-shanghai.aliyuncs.com/202212290902016.png)

会发现有注册好的服务

### 服务发现

下面，我们将order-service的逻辑修改：向eureka-server拉取user-service的信息，实现服务发现。

#### 引入依赖

之前说过，服务发现、服务注册统一都封装在eureka-client依赖，因此这一步与服务注册时一致。

在order-service的pom文件中，引入下面的eureka-client依赖：

```XML
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
</dependency>
```

#### 配置文件

服务发现也需要知道eureka地址，因此第二步与服务注册一致，都是配置eureka信息：

在order-service中，修改application.yml文件，添加服务名称、eureka地址：

```YAML
spring:
  application:
    name: orderservice
eureka:
  client:
    service-url:
      defaultZone: http://127.0.0.1:10086/eureka
  instance:
    prefer-ip-address: true
    instance-id: 127.0.0.1:${server.port}    
```

#### 服务拉取和负载均衡

最后，我们要去eureka-server中拉取user-service服务的实例列表，并且实现负载均衡。

不过这些动作不用我们去做，只需要添加一些注解即可。

**在order-service的OrderApplication中，给RestTemplate这个Bean添加一个@LoadBalanced注解：**

```Java
public class OrderApplication {

    public static void main(String[] args) {
        SpringApplication.run(OrderApplication.class, args);
    }

    @Bean
    @LoadBalanced
    public RestTemplate restTemplate(){
        return new RestTemplate();
    }
}
```

修改order-service服务中的cn.itcast.order.service包下的OrderService类中的queryOrderById方法。修改访问的url路径，用服务名代替ip、端口：

![img](https://shanzhu-edu.oss-cn-shanghai.aliyuncs.com/202212290902043.png)

spring会自动帮助我们从eureka-server端，根据userservice这个服务名称，获取实例列表，而后完成负载均衡。

## Ribbon负载均衡

我们凭借一个小小的注解@LoadBalanced就可以为服务配置负载均衡，那这背后到底是为什么

### 负载均衡原理

SpringCloud底层其实是利用了一个名为**Ribbon**的组件，来实现负载均衡功能的。

![img](https://shanzhu-edu.oss-cn-shanghai.aliyuncs.com/202212290902063.png)

那么我们发出的请求明明是http://userservice/user/1，怎么变成了http://localhost:8081的呢？

### Ribbon源码跟踪

为什么我们只输入了service名称就可以访问了呢？之前还要获取ip和端口

显然有人帮我们根据service名称，获取到了服务实例的ip和端口。它就是LoadBalancerInterceptor，这个类会在对RestTemplate的请求进行拦截，然后从Eureka根据服务id获取服务列表，随后利用负载均衡算法得到真实的服务地址信息，替换服务id。

我们进行源码跟踪：

#### LoadBalancerIntercepor

![img](https://shanzhu-edu.oss-cn-shanghai.aliyuncs.com/202212290902103.png)

可以看到这里的intercept方法，拦截了用户的HttpRequest请求，然后做了几件事：

- request.getURI()：获取请求uri，本例中就是 http://user-service/user/8
- originalUri.getHost()：获取uri路径的主机名，其实就是服务id，user-service
- this.loadBalancer.execute()：处理服务id，和用户请求。

这里的`this.loadBalancer`是`LoadBalancerClient`类型，我们继续跟入。

#### LoadBalancerClient

继续跟入execute方法：

![img](https://shanzhu-edu.oss-cn-shanghai.aliyuncs.com/202212290902515.png)

getLoadBalancer(serviceId)：根据服务id获取ILoadBalancer，而ILoadBalancer会拿着服务id去eureka中获取服务列表并保存起来。

getServer(loadBalancer)：利用内置的负载均衡算法，从服务列表中选择一个。本例中，可以看到获取了8082端口的服务

放行后，再次访问并跟踪，发现获取的是8081端口。果然实现了负载均衡。

#### 负载均衡策略IRule

在刚才的代码中，可以看到获取服务使通过一个`getServer`方法来做负载均衡:

![img](https://shanzhu-edu.oss-cn-shanghai.aliyuncs.com/202212290902590.png)

我们继续跟入：

![img](https://shanzhu-edu.oss-cn-shanghai.aliyuncs.com/202212290902654.png)

继续跟踪源码chooseServer方法，发现这么一段代码

![img](https://shanzhu-edu.oss-cn-shanghai.aliyuncs.com/202212290902662.png)

我们看看这个rule是什么

![img](https://shanzhu-edu.oss-cn-shanghai.aliyuncs.com/202212290902701.png)

这里的rule默认值是一个RoundRobinRule，看类的介绍

![img](https://shanzhu-edu.oss-cn-shanghai.aliyuncs.com/202212290902756.png)

显然是轮询的意思。

### 总结

SpringCloudRibbon的底层采用了一个[拦截器](https://so.csdn.net/so/search?q=拦截器&spm=1001.2101.3001.7020)，拦截了RestTemplate发出的请求，对地址做了修改。用一幅图来总结一下：

![img](https://shanzhu-edu.oss-cn-shanghai.aliyuncs.com/202212290902121.png)

基本流程如下：

- 拦截我们的RestTemplate请求http://userservice/user/1
- RibbonLoadBalancerClient会从请求url中获取服务名称，也就是user-service
- DynamicServerListLoadBalancer根据user-service到eureka拉取服务列表
- eureka返回列表，localhost:8081、localhost:8082
- IRule利用内置负载均衡规则，从列表中选择一个，例如localhost:8081
- RibbonLoadBalancerClient修改请求地址，用localhost:8081替代userservice，得到http://localhost:8081/user/1，发起真实请求

### 负载均衡策略

负载均衡的规则都定义在IRule接口中，而IRule有很多不同的实现类：

![img](https://shanzhu-edu.oss-cn-shanghai.aliyuncs.com/202212290902193.png)

不同规则的含义如下：

| 内置负载均衡规则类        | 规则描述                                                     |
| ------------------------- | ------------------------------------------------------------ |
| RoundRobinRule            | 简单轮询服务列表来选择服务器。                               |
| AvailabilityFilteringRule | 对以下两种服务器进行忽略： （1）在默认情况下，这台服务器如果3次连接失败，这台服务器就会被设置为“短路”状态。短路状态将持续30秒，如果再次连接失败，短路的持续时间就会几何级地增加。 （2）并发数过高的服务器。如果一个服务器的并发连接数过高，配置了AvailabilityFilteringRule规则的客户端也会将其忽略。并发连接数的上限，可以由客户端的…ActiveConnectionsLimit属性进行配置<br><br> |
| WeightedResponseTimeRule  | 为每一个服务器赋予一个权重值。服务器响应时间越长，这个服务器的权重就越小。这个规则会随机选择服务器，这个权重值会影响服务器的选择。 |
| ZoneAvoidanceRule         | 以区域可用的服务器为基础进行服务器的选择。使用Zone对服务器进行分类，这个Zone可以理解为一个机房、一个机架等。而后再对Zone内的多个服务做轮询。 |
| BestAvailableRule         | 忽略那些短路的服务器，并选择并发数较低的服务器。             |
| RandomRule                | 随机选择一个可用的服务器。                                   |
| RetryRule                 | 重试机制的选择逻辑                                           |
|                           |                                                              |

### 自定义负载均衡策略

通过定义IRule实现可以修改负载均衡规则，有两种方式：

- 代码方式：在order-service中的OrderApplication类中，定义一个新的IRule：

```Java
@Bean
public IRule randomRule(){
    return new RandomRule();
}
```

- 配置文件方式：在order-service的application.yml文件中，添加新的配置也可以修改规则

```YAML
userservice: # 给某个微服务配置负载均衡规则，这里是userservice服务
  ribbon:
    NFLoadBalancerRuleClassName: com.netflix.loadbalancer.RandomRule # 负载均衡规则 
```

**注意**，一般用默认的负载均衡规则，不做修改。

### 饥饿加载

Ribbon默认是采用懒加载，即第一次访问时才会去创建LoadBalanceClient，请求时间会很长

而饥饿加载则会在项目启动时创建，降低第一次访问的耗时，通过下面配置开启饥饿加载：

```YAML
ribbon:
  eager-load:
    enabled: true # 开启饥饿加载
    clients:
      - userservice # 指定饥饿加载的服务名称
      - xxxxservice # 如果需要指定多个，需要这么写
```

## Nacos注册中心

国内公司一般都推崇阿里巴巴的技术，比如注册中心，SpringCloudAlibaba也推出了一个名为Nacos的注册中心。

### 认识和安装Nacos

[Nacos](https://nacos.io/)是阿里巴巴的产品，现在是[SpringCloud](https://spring.io/projects/spring-cloud)中的一个组件。相比[Eureka](https://github.com/Netflix/eureka)功能更加丰富，在国内受欢迎程度较高。

#### 启动

- bin 启动脚本
- conf 配置文件

Nacos的默认端口是8848，如果你电脑上的其它进程占用了8848端口，请先尝试关闭该进程。

**如果无法关闭占用8848端口的进程**，也可以进入nacos的conf目录，修改配置文件(application.properties)中的端口：

![img](https://shanzhu-edu.oss-cn-shanghai.aliyuncs.com/202212290902225.png)

**启动**，进入bin目录下：

![img](https://shanzhu-edu.oss-cn-shanghai.aliyuncs.com/202212290902269.png)

windows下执行命令：

```Bash
startup.cmd -m standalone
```

linux下启动命令：

```Bash
startup.sh -m standalone
```

### 服务注册

Nacos是SpringCloudAlibaba的组件，而SpringCloudAlibaba也遵循SpringCloud中定义的服务注册、服务发现规范。因此使用Nacos和使用Eureka对于微服务来说，并没有太大区别。

主要差异在于：

- 依赖不同
- 服务地址不同

#### 引入依赖

在cloud-demo父工程的pom文件中的`<dependencyManagement>`中引入SpringCloudAlibaba的依赖：

```XML
<dependency>
    <groupId>com.alibaba.cloud</groupId>
    <artifactId>spring-cloud-alibaba-dependencies</artifactId>
    <version>2.2.6.RELEASE</version>
    <type>pom</type>
    <scope>import</scope>
</dependency>
```

然后在user-service和order-service中的pom文件中引入nacos-discovery依赖：

```XML
<dependency>
    <groupId>com.alibaba.cloud</groupId>
    <artifactId>spring-cloud-starter-alibaba-nacos-discovery</artifactId>
</dependency>
```

#### 配置nacos地址

在user-service和order-service的application.yml中添加nacos地址：

```YAML
spring:
  cloud:
    nacos:
      server-addr: localhost:8848
```

#### 重启

重启微服务后，登录nacos管理页面，可以看到微服务信息

![img](https://shanzhu-edu.oss-cn-shanghai.aliyuncs.com/202212290902281.png)

### 服务分级存储模型

一个服务可以有多个实例，例如我们的user-service，可以有:

- 127.0.0.1:8081
- 127.0.0.1:8082
- 127.0.0.1:8083

假如这些实例分布于全国各地的不同机房，例如：

- 127.0.0.1:8081，在上海机房

- 127.0.0.1:8082，在上海机房

- 127.0.0.1:8083，在杭州机房

  Nacos就将同一机房内的实例 划分为一个**集群**。

也就是说，user-service是服务，一个服务可以包含多个**集群**，如杭州、上海，每个集群下可以有多个实例，形成分级模型，如图：

![img](https://shanzhu-edu.oss-cn-shanghai.aliyuncs.com/202212290902345.png)

微服务互相访问时，应该尽可能访问同集群实例，因为本地访问速度更快。当本集群内不可用时，才访问其它集群。例如：

![img](https://shanzhu-edu.oss-cn-shanghai.aliyuncs.com/202212290902725.png)

杭州机房内的order-service应该优先访问同机房的user-service。

#### 给user-service配置集群

修改user-service的application.yml文件，添加集群配置：

```YAML
spring:
  cloud:
    nacos:
      server-addr: localhost:8848
      discovery:
        cluster-name: HZ # 集群名称
```

重启两个user-service实例后，我们可以在nacos控制台看到下面结果：

![img](https://shanzhu-edu.oss-cn-shanghai.aliyuncs.com/202212290902774.png)

我们再次复制一个user-service启动配置，添加属性：

```YAML
-Dserver.port=8083 -Dspring.cloud.nacos.discovery.cluster-name=SH
```

#### 同集群优先的负载均衡

默认的ZoneAvoidanceRule并不能实现根据同集群优先来实现负载均衡。

**因此Nacos中提供了一个NacosRule的实现，可以优先从同集群中挑选实例**。

1）给order-service配置集群信息

修改order-service的application.yml文件，添加集群配置：

```YAML
spring:
  cloud:
    nacos:
      server-addr: localhost:8848
      discovery:
        cluster-name: HZ # 集群名称
```

2）修改负载均衡规则

修改order-service的application.yml文件，修改负载均衡规则

```YAML
userservice:
  ribbon:
    NFLoadBalancerRuleClassName: com.alibaba.cloud.nacos.ribbon.NacosRule # 负载均衡规则 
```

### 权重配置

实际部署中会出现这样的场景：

服务器设备性能有差异，部分实例所在机器性能较好，另一些较差，我们希望性能好的机器承担更多的用户请求。

但默认情况下NacosRule是同集群内随机挑选，不会考虑机器的性能问题。

因此，Nacos提供了权重配置来控制访问频率，权重越大则访问频率越高。

在nacos控制台，找到user-service的实例列表，点击编辑，即可修改权重：

![img](https://shanzhu-edu.oss-cn-shanghai.aliyuncs.com/202212290902822.png)

在弹出的编辑窗口，修改权重：

![img](https://shanzhu-edu.oss-cn-shanghai.aliyuncs.com/202212290902885.png)

**注意**：如果权重修改为0，则该实例永远不会被访问

### 环境隔离

Nacos提供了namespace来实现环境隔离功能。

- nacos中可以有多个namespace
- namespace下可以有group、service等
- 不同namespace之间相互隔离，例如不同namespace的服务互相不可见

#### 创建namespace

默认情况下，所有service、data、group都在同一个namespace，名为public：

![img](https://shanzhu-edu.oss-cn-shanghai.aliyuncs.com/202212290902912.png)

我们可以点击页面新增按钮，添加一个namespace：

![img](https://shanzhu-edu.oss-cn-shanghai.aliyuncs.com/202212290902965.png)

然后，填写表单：

![img](https://shanzhu-edu.oss-cn-shanghai.aliyuncs.com/202212290902313.png)

就能在页面看到一个新的namespace：

![img](https://shanzhu-edu.oss-cn-shanghai.aliyuncs.com/202212290902352.png)

#### 给微服务配置namespace

给微服务配置namespace只能通过修改配置来实现。

例如，修改order-service的application.yml文件：

```YAML
spring:
  cloud:
    nacos:
      server-addr: localhost:8848
      discovery:
        cluster-name: HZ
        namespace: 492a7d5d-237b-46a1-a99a-fa8e98e4b0f9 # 命名空间，填ID
```

重启order-service后，访问控制台，可以看到下面的结果：

![img](https://shanzhu-edu.oss-cn-shanghai.aliyuncs.com/202212290902386.png)

![img](https://shanzhu-edu.oss-cn-shanghai.aliyuncs.com/202212290902417.png)

此时访问order-service，因为namespace不同，会导致找不到userservice，控制台会报错：

![img](https://shanzhu-edu.oss-cn-shanghai.aliyuncs.com/202212290902489.png)

### Nacos与Eureka的区别

Nacos的服务实例分为两种l类型：

- 临时实例：如果实例宕机超过一定时间，会从服务列表剔除，默认的类型。
- 非临时实例：如果实例宕机，不会从服务列表剔除，也可以叫永久实例。

配置一个服务实例为永久实例：

```YAML
spring:
  cloud:
    nacos:
      discovery:
        ephemeral: false # 设置为非临时实例
```

Nacos和Eureka整体结构类似，服务注册、服务拉取、心跳等待，但是也存在一些差异：

![img](https://shanzhu-edu.oss-cn-shanghai.aliyuncs.com/202212290902500.png)

- Nacos与eureka的共同点

  都支持服务注册和服务拉取

  都支持服务提供者心跳方式做健康检测

- Nacos与Eureka的区别

  Nacos支持服务端主动检测提供者状态：临时实例采用心跳模式，非临时实例采用主动检测模式 临时实例心跳不正常会被剔除，非临时实例则不会被剔除 Nacos支持服务列表变更的消息推送模式，服务列表更新更及时 Nacos集群默认采用AP方式，当集群中存在非临时实例时，采用CP模式；Eureka采用AP方式

## Nacos配置管理

Nacos除了可以做注册中心，同样可以做配置管理来使用。

### 统一配置管理

当[微服务](https://so.csdn.net/so/search?q=微服务&spm=1001.2101.3001.7020)部署的实例越来越多，达到数十、数百时，逐个修改微服务配置就会让人抓狂，而且很容易出错。我们需要一种统一配置管理方案，可以集中管理所有实例的配置。

![img](https://shanzhu-edu.oss-cn-shanghai.aliyuncs.com/202212290902776.png)

Nacos一方面可以将配置集中管理，另一方可以在配置变更时，及时通知微服务，实现配置的热更新。

#### 在nacos中添加配置文件

如何在nacos中管理配置呢？

![img](https://shanzhu-edu.oss-cn-shanghai.aliyuncs.com/202212290902838.png)

然后在弹出的表单中，填写配置信息：

![img](https://shanzhu-edu.oss-cn-shanghai.aliyuncs.com/202212290902873.png)

注意：项目的核心配置，需要热更新的配置才有放到nacos管理的必要。基本不会变更的一些配置还是保存在微服务本地比较好。

#### 从微服务拉取配置

微服务要拉取nacos中管理的配置，并且与本地的application.yml配置合并，才能完成项目启动。

但如果尚未读取application.yml，又如何得知nacos地址呢？

因此spring引入了一种新的配置文件：bootstrap.yaml文件，会在application.yml之前被读取，流程如下：

![img](https://shanzhu-edu.oss-cn-shanghai.aliyuncs.com/202212290902910.png)

#### 引入nacos-config依赖

首先，在user-service服务中，引入nacos-config的客户端依赖：

```XML
<!--nacos配置管理依赖-->
<dependency>
    <groupId>com.alibaba.cloud</groupId>
    <artifactId>spring-cloud-starter-alibaba-nacos-config</artifactId>
</dependency>
```

然后，在user-service中添加一个bootstrap.yaml文件，内容如下

```YAML
spring:
  application:
    name: userservice # 服务名称
  profiles:
    active: dev #开发环境，这里是dev 
  cloud:
    nacos:
      server-addr: localhost:8848 # Nacos地址
      config:
        file-extension: yaml # 文件后缀名
```

这里会根据spring.cloud.nacos.server-addr获取nacos地址，再根据

`spring.application.name−{`[`spring.application.name`](http://spring.application.name)`}-spring.application.name−{spring.profiles.active}.${spring.cloud.nacos.config.file-extension}`作为文件id，来读取配置。

本例中，就是去读取userservice-dev.yaml：

![img](https://shanzhu-edu.oss-cn-shanghai.aliyuncs.com/202212290902088.png)

#### 读取nacos配置

在user-service中的UserController中添加业务逻辑，读取pattern.dateformat配置：

![img](https://shanzhu-edu.oss-cn-shanghai.aliyuncs.com/202212290902143.png)

### 配置热更新

我们最终的目的，是修改nacos中的配置后，微服务中无需重启即可让配置生效，也就是**配置热更新**。

要实现配置热更新，可以使用两种方式：

#### 方式一

在@Value注入的变量所在类上添加注解@RefreshScope

![img](https://shanzhu-edu.oss-cn-shanghai.aliyuncs.com/202212290902342.png)

#### 方式二

使用@ConfigurationProperties注解代替@Value注解。

在user-service服务中，添加一个类，读取patterrn.dateformat属性:

```Java
package cn.itcast.user.config;

import lombok.Data;
import org.springframework.boot.context.properties.ConfigurationProperties;
import org.springframework.stereotype.Component;

@Component
@Data
@ConfigurationProperties(prefix = "pattern")
public class PatternProperties {
    private String dateformat;
}
```

在UserController中使用这个类代替@Value

```Java
package cn.itcast.user.web;

import cn.itcast.user.config.PatternProperties;
import cn.itcast.user.pojo.User;
import cn.itcast.user.service.UserService;
import lombok.extern.slf4j.Slf4j;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.PathVariable;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;

import java.time.LocalDateTime;
import java.time.format.DateTimeFormatter;

@Slf4j
@RestController
@RequestMapping("/user")
public class UserController {

    @Autowired
    private UserService userService;

    @Autowired
    private PatternProperties patternProperties;

    @GetMapping("now")
    public String now(){
        return LocalDateTime.now().format(DateTimeFormatter.ofPattern(patternProperties.getDateformat()));
    }

    // 略
}
```

### 配置共享

其实微服务启动时，会去nacos读取多个配置文件，例如：

- [[spring.application.name](http://spring.application.name)]-[spring.profiles.active].yaml，例如：userservice-dev.yaml
- [[spring.application.name](http://spring.application.name)].yaml，例如：userservice.yaml

而[[spring.application.name](http://spring.application.name)].yaml不包含环境，因此可以被多个环境共享。

#### 添加一个环境共享变量

我们在nacos中添加一个userservice.yaml文件

![img](https://shanzhu-edu.oss-cn-shanghai.aliyuncs.com/202212290902384.png)

#### 在user-service中读取共享配置

在user-service服务中，修改PatternProperties类，读取新添加的属性：

![img](https://shanzhu-edu.oss-cn-shanghai.aliyuncs.com/202212290902492.png)

在user-service服务中，修改UserController，添加一个方法：

![img](https://shanzhu-edu.oss-cn-shanghai.aliyuncs.com/202212290902543.png)

### 配置共享的优先级

当nacos、服务本地同时出现相同属性时，优先级有高低之分：

![img](https://shanzhu-edu.oss-cn-shanghai.aliyuncs.com/202212290902641.png)

## 搭建Nacos集群

Nacos生产环境下一定要部署为集群状态，部署方式参考课前资料中的文档：

### 集群结构图

官方给出的Nacos集群图

![img](https://shanzhu-edu.oss-cn-shanghai.aliyuncs.com/202212290902717.png)

其中包含3个nacos节点，然后一个[负载均衡](https://so.csdn.net/so/search?q=负载均衡&spm=1001.2101.3001.7020)器代理3个Nacos。这里负载均衡器可以使用nginx。

我们计划的集群结构：

![img](https://shanzhu-edu.oss-cn-shanghai.aliyuncs.com/202212290902920.png)

三个nacos节点的地址：

| 节点   | ip            | port |
| ------ | ------------- | ---- |
| nacos1 | 192.168.150.1 | 8845 |
| nacos2 | 192.168.150.1 | 8846 |
| nacos3 | 192.168.150.1 | 8847 |

### 搭建集群

搭建集群的基本步骤：

- 搭建数据库，初始化数据库表结构

  导入sql，新建数据库。

- 下载nacos安装包

  nacos在GitHub上有下载地址：`https://github.com/alibaba/nacos/tags`，可以选择任意版本下载。

- 配置nacos

  进入nacos的conf目录，修改配置文件cluster.conf.example，重命名为cluster.conf，然后添加内容：

```Java
127.0.0.1:8845
127.0.0.1.8846
127.0.0.1.8847
然后修改application.properties文件，添加数据库配置
spring.datasource.platform=mysql

db.num=1

db.url.0=jdbc:mysql://127.0.0.1:3306/nacos?characterEncoding=utf8&connectTimeout=1000&socketTimeout=3000&autoReconnect=true&useUnicode=true&useSSL=false&serverTimezone=UTC
db.user.0=root
db.password.0=123
```

- 启动nacos集群

  将nacos文件夹复制三份，分别命名为：nacos1、nacos2、nacos3

  然后分别修改三个文件夹中的application.properties，修改端口号为8845 8846 8847保存并分别启动。

- nginx反向代理

  修改nginx/nginx.conf文件，配置如下

```XML
upstream nacos-cluster {
    server 127.0.0.1:8845;
  server 127.0.0.1:8846;
  server 127.0.0.1:8847;
}

server {
    listen       80;
    server_name  localhost;

    location /nacos {
        proxy_pass http://nacos-cluster;
    }
}
```

而后在浏览器访问：http://localhost/nacos即可。

代码中application.yml文件配置如下：

```YAML
spring:
  cloud:
    nacos:
      server-addr: localhost:80 # Nacos地址
```

### 优化

- 实际部署时，需要给做反向代理的nginx服务器设置一个域名，这样后续如果有服务器迁移nacos的客户端也无需更改配置.
- Nacos的各个节点应该部署到多个不同服务器，做好容灾和隔离