> 接着一之后这篇文章将从fegin开始记录，帮助自己一点点的扫盲springcloud.

## Feign远程调用

先来看我们以前利用RestTemplate发起远程调用的代码：

```java
String url ="http://userservice/user/"+order.getUserId();
User user=restTemplate.getForObject(url,User.class);
```

可以明显看出这样写的问题：

* 代码的可读性，编程体验不统一。
* 参数复杂URL难以维护。

> Feign是一个声明式的http客户端，客户地址：<https://github.com/OpenFeign/feign>  其作用就是帮助我们优雅的实现http请求的发送，解决上面遇到的问题。

![image-20221229165445786](https://shanzhu-edu.oss-cn-shanghai.aliyuncs.com/blog/202212291654993.png)

###  Feign代替ResTemplate

Feign的使用步骤如下：

####  引入依赖

我们在order-service服务的pom文件中引入feign的依赖：

```xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-openfeign</artifactId>
</dependency>

```

####  添加注解

在order-service的启动类添加注解开启Feign的功能：

![image-20221229170315470](https://shanzhu-edu.oss-cn-shanghai.aliyuncs.com/blog/202212291703561.png)

####  编写Feign客户端

在order-service中新建一个接口，内容如下：

```java
package cn.itcast.order.client;

import cn.itcast.order.pojo.User;
import org.springframework.cloud.openfeign.FeignClient;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.PathVariable;

@FeignClient("userservice")
public interface UserClient {
    @GetMapping("/user/{id}")
    User findById(@PathVariable("id") Long id);
}

```

这个客户端主要是基于SpringMVC的注解来声明远程调用的信息，比如：

* 服务名称：userservice
* 请求方式：Get
* 请求路径：/user/{id}
* 请求参数：Long id
* 返回值类型：User

这样，Feign就可以帮助我们发送http请求，无需自己使用RestTemplate来发送了。

####  测试

修改order-service中的OrderService类中的queryOrderById方法，使用Feign客户端代替RestTemplate

~~~java
    @Autowired
    private UserClient userClient;

    public Order queryOrderById(Long orderId) {
        // 1.查询订单
        Order order = orderMapper.findById(orderId);
        // 2.用Feign远程调用
        User user = userClient.findById(order.getUserId());
        // 3.封装user到Order
        order.setUser(user);
        // 4.返回
        return order;
    }
~~~

这样的话就优雅很多了。

#### 总结

使用Feign的步骤：

* 引入依赖
* 添加@EnableFeignClients注解
* 编写FeignClient接口
* 使用FeignClient中定义的方法代替RestTemplate

### 自定义配置

| **类型**            | **作用**         | **说明**                                               |
| ------------------- | ---------------- | ------------------------------------------------------ |
| feign.Logger.Level  | 修改日志级别     | 包含四种不同的级别：NONE、BASIC、HEADERS、FULL         |
| feign.codec.Decoder | 响应结果的解析器 | http远程调用的结果做解析，例如解析json字符串为java对象 |
| feign.codec.Encoder | 请求参数编码     | 将请求参数编码，便于通过http请求发送                   |
| feign. Contract     | 支持的注解格式   | 默认是SpringMVC的注解                                  |
| feign. Retryer      | 失败重试机制     | 请求失败的重试机制，默认是没有，不过会使用Ribbon的重试 |

一般情况下，默认值就能满足我们使用，如果要自定义时，只需要创建自定义的@Bean覆盖默认Bean即可。

下面以日志为例来演示如何自定义配置。

####  配置文件方式

基于配置文件修改feign的日志级别可以针对单个服务：

~~~ yaml
feign:  
  client:
    config: 
      userservice: # 针对某个微服务的配置
        loggerLevel: FULL #  日志级别 

~~~

也可以针对所有服务：

~~~yaml
feign:  
  client:
    config: 
      default: # 这里用default就是全局配置，如果是写服务名称，则是针对某个微服务的配置
        loggerLevel: FULL #  日志级别 

~~~

而日志的级别分为四种：

- NONE：不记录任何日志信息，这是默认值。
- BASIC：仅记录请求的方法，URL以及响应状态码和执行时间
- HEADERS：在BASIC的基础上，额外记录了请求和响应的头信息
- FULL：记录所有请求和响应的明细，包括头信息、请求体、元数据。

#### Java代码方式配置

也可以基于Java代码来修改日志级别，先声明一个类，然后声明一个Logger.Level对象：

~~~java
public class DefaultFeignConfiguration  {
    @Bean
    public Logger.Level feignLogLevel(){
        return Logger.Level.BASIC; // 日志级别为BASIC
    }
}
~~~

如果需要全局生效，将其放到启动类的@EnableFeignClients这个注解中：

~~~java
@EnableFeignClients(defaultConfiguration = DefaultFeignConfiguration .class) 
~~~

如果是局部生效，则把它放到接口对应的@FeignClient这个注解中：

~~~java
@FeignClient(value = "userservice", configuration = DefaultFeignConfiguration .class) 
~~~

###  Feign使用优化

Feign底层发起http请求，依赖于其它的框架。其底层客户端实现包括：

* URLConnection：默认实现，不支持连接池
* Apache HttpClient ：支持连接池
* OKHttp：支持连接池

因此提高Feign的性能主要手段就是使用连接池代替默认的URLConnection。

这里我们用Apache的HttpClient来演示。

#### 引入依赖

在order-service的pom文件中引入Apache的HttpClient依赖：

~~~xml
<!--httpClient的依赖 -->
<dependency>
    <groupId>io.github.openfeign</groupId>
    <artifactId>feign-httpclient</artifactId>
</dependency>
~~~

#### 配置连接池

在order-service的application.yml中添加配置：

~~~yaml
feign:
  client:
    config:
      default: # default全局的配置
        loggerLevel: BASIC # 日志级别，BASIC就是基本的请求和响应信息
  httpclient:
    enabled: true # 开启feign对HttpClient的支持
    max-connections: 200 # 最大的连接数
    max-connections-per-route: 50 # 每个路径的最大连接数
~~~

接下来，在FeignClientFactoryBean中的loadBalance方法中打断点：

![image-20221230091443885](https://shanzhu-edu.oss-cn-shanghai.aliyuncs.com/blog/202212300914007.png)

Debug方式启动order-service服务，可以看到这里的client，底层就是Apache HttpClient：



![image-20221230091630867](https://shanzhu-edu.oss-cn-shanghai.aliyuncs.com/blog/202212300916924.png)

总结，Feign的优化：

* 日志级别尽量用basic/
* 使用HttpClient或OKHttp代替URLConnection
  * 先引入feign-httpClient依赖，再配置文件开启httpClient功能，设置连接池参数。

###  最佳实践

所谓最近实践，就是使用过程中总结的经验，最好的一种使用方式。仔细观察可以发现，Feign的客户端与服务提供者的controller代码非常相似

feign客户端：

~~~java
@FeignClient(value = "userservice")
public interface UserClient {

    @GetMapping("/user/{id}")
    User findById(@PathVariable("id") Long id);
}
~~~

User的控制层：

```Java
@GetMapping("/{id}")
public User queryById(@PathVariable("id") Long id,
                      @RequestHeader(value = "Truth", required = false) String truth) {
    System.out.println("truth: " + truth);
    return userService.queryById(id);
}
```

我们可以寻找一种方式去简化这样的写法

####  继承方式

> 一样的代码可以通过继承来共享

* 定义一个API接口，利用定义方法，并基于SpringMvc注解做声明。

* Feign客户端和Controller都集成改接口。

  ![image-20221230101712550](https://shanzhu-edu.oss-cn-shanghai.aliyuncs.com/blog/202212301017644.png)

优点：

- 简单
- 实现了代码共享

缺点：

- 服务提供方、服务消费方紧耦合
- 参数列表中的注解映射并不会继承，因此Controller中必须再次声明方法、参数列表、注解

#### 抽取方式

将Feign的Client抽取为独立模块，并且把接口有关的POJO、默认的Feign配置都放到这个模块中，提供给所有消费者使用。

例如，将UserClient、User、Feign的默认配置都抽取到一个feign-api包中，所有微服务引用该依赖包，即可直接使用。

![image-20221230112839374](https://shanzhu-edu.oss-cn-shanghai.aliyuncs.com/blog/202212301128502.png)

### 实现基于抽取的最佳实践

#### 抽取

首先创建一个module 命名为feign-api

![image-20221230112959089](https://shanzhu-edu.oss-cn-shanghai.aliyuncs.com/blog/202212301129193.png)

在feign-api中然后引入feign的starter依赖

~~~xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-openfeign</artifactId>
</dependency>
~~~

然后编写UserClient

```java
import cn.itcast.feign.pojo.User;
import org.springframework.cloud.openfeign.FeignClient;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.PathVariable;

@FeignClient(value = "userservice")
public interface UserClient {

    @GetMapping("/user/{id}")
    User findById(@PathVariable("id") Long id);
}
```

DefaultFeignConfiguration

```java
import feign.Logger;
import org.springframework.context.annotation.Bean;

public class DefaultFeignConfiguration {
    @Bean
    public Logger.Level logLevel(){
        return Logger.Level.BASIC;
    }
}
```

User实体类

```Java
import lombok.Data;

@Data
public class User {
    private Long id;
    private String username;
    private String address;
}
```

#### 在order-service中使用feign-api

在order-service的pom文件中中引入feign-api的依赖：

~~~xml
<dependency>
    <groupId>cn.itcast.demo</groupId>
    <artifactId>feign-api</artifactId>
    <version>1.0</version>
</dependency>
~~~

修改order-service中的所有与上述三个组件有关的导包部分，改成导入feign-api中的包

#### 注意问题

我们在启动程序后会报出以下错误，无法找到UserClient

![image-20221230113613962](https://shanzhu-edu.oss-cn-shanghai.aliyuncs.com/blog/202212301136072.png)

这是因为UserClient现在在cn.itcast.feign.clients包下，而order-service的@EnableFeignClients注解是在cn.itcast.order包下，不在同一个包，无法扫描到UserClient。==我们有两种方法解决此问题，==

* 指定Feign应该扫描的包

  ~~~java
  @EnableFeignClients(basePackages = "cn.itcast.feign.clients")
  ~~~

* 指定需要加载的Client接口

  ~~~java
  @EnableFeignClients(clients = {UserClient.class})
  ~~~

