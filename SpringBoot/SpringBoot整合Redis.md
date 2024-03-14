> *本篇文章将记录在springboot中整合redis并使用SpringDataRedis的一些api方法。由于jedis使用简单，并不在此进行记录。*

### SpringDataRedis介绍

SpringData是Spring中数据操作的模块，包含对各种数据库的集成，例如MySQL等，其中对Redis的集成模块就叫做`SpringDataRedis`

SpringDataRedist特点如下：

- 提供了对不同Redis客户端的整合（`Lettuce`和`Jedis`）
- 提供了`RedisTemplate`统一API来操作Redis
- 支持Redis的发布订阅模型
- 支持Redis哨兵和Redis集群
- 支持基于Lettuce的响应式编程
- 支持基于JDK、JSON、字符串、Spring对象的数据序列化及反序列化
- 支持基于Redis的JDKCollection实现

SpringDataRedis中提供了RedisTemplate工具类，其中封装了各种对Redis的操作。并且将不同数据类型的操作API封装到了不同的类型中：

![img](https://shanzhu-edu.oss-cn-shanghai.aliyuncs.com/image-20220525140217446.png)

### 使用SpringDataRedis

#### 引入依赖

连接池依赖：

```XML
<dependency>
    <groupId>org.apache.commons</groupId>
    <artifactId>commons-pool2</artifactId>
</dependency>
```

redis依赖：

```XML
    <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-data-redis</artifactId>
    </dependency>
```

#### 编写配置文件

```YAML
spring:
  redis:
    host: 127.0.0.1 #指定redis所在的host
    port: 6379  #指定redis的端口
    password: 132537  #设置redis密码
    lettuce:
      pool:
        max-active: 8 #最大连接数
        max-idle: 8 #最大空闲数
        min-idle: 0 #最小空闲数
        max-wait: 100ms #连接等待时间
```

#### 开始测试

这个时候就可以直接开码了，先简单的使用redis为数据库添加数据：

```Java
@SpringBootTest
class RedisDemoApplicationTests {

  @Resource
  private RedisTemplate redisTemplate;

  @Test
  void testString() {
    // 1.通过RedisTemplate获取操作String类型的ValueOperations对象
    ValueOperations ops = redisTemplate.opsForValue();
    // 2.插入一条数据
    ops.set("Name","shanzhu");
    
    // 3.获取数据
    String name = (String) ops.get("Name");
    System.out.println("name = " + name);
  }
}
```

虽然添加查询成功了，但不难发现在redis中，出现了一串`\xac\xed\x00\x05t\x00\x04name`这样的字符串,因为RedisTemplate可以接收任意Object作为值写入Redis，只不过写入前会把Object序列化为字节形式，`默认是采用JDK序列化`，所以会得到这样的结果。

#### 自定义RedisTemplate序列化

**那么以上的问题如何解决**，我们可以通过自定义`RedisTemplate`序列化的方式来解决。

引入依赖：

```XML
  <dependency>
            <groupId>com.fasterxml.jackson.core</groupId>
            <artifactId>jackson-databind</artifactId>
  </dependency>
```

编写一个配置类RedisConfig:

```Java
@Configuration
public class RedisConfig {

    @Bean
    public RedisTemplate<String,Object> redisTemplate(RedisConnectionFactory factory){
        // 1.创建RedisTemplate对象
        RedisTemplate<String ,Object> redisTemplate = new RedisTemplate<>();
        // 2.设置连接工厂
        redisTemplate.setConnectionFactory(factory);

        // 3.创建序列化对象
        StringRedisSerializer stringRedisSerializer = new StringRedisSerializer();
        GenericJackson2JsonRedisSerializer genericJackson2JsonRedisSerializer = new GenericJackson2JsonRedisSerializer();

        // 4.设置key和hashKey采用String的序列化方式
        redisTemplate.setKeySerializer(stringRedisSerializer);
        redisTemplate.setHashKeySerializer(stringRedisSerializer);

        // 5.设置value和hashValue采用json的序列化方式
        redisTemplate.setValueSerializer(genericJackson2JsonRedisSerializer);
        redisTemplate.setHashValueSerializer(genericJackson2JsonRedisSerializer);

        return redisTemplate;
    }
}
```

由于我们设置的value序列化方式是json的，因此我们可以直接向redis中插入一个对象

```Java
@Test
void testSaveUser() {
    redisTemplate.opsForValue().set("user:100", new User("Vz", 21));
    User user = (User) redisTemplate.opsForValue().get("user:100");
    System.out.println("User = " + user);
}
```

但我们又发现了一个问题，在redis中，不仅有了我们想要的序列化后的数据，还多出了一条类的class类型数据`com.shanzhu.redis.model.User`，他的内存占比甚至比我们要存储的数据还大。

#### 使用StringRedisTemplate

为了解决上述问题，我们使用StringRedisTemplate来执行操作。

```Java
@SpringBootTest
class RedisStringTemplateTest {

  @Resource
  private StringRedisTemplate stringRedisTemplate;

  @Test
  void testSaveUser() throws JsonProcessingException {
    // 1.创建一个Json序列化对象
    ObjectMapper objectMapper = new ObjectMapper();
    // 2.将要存入的对象通过Json序列化对象转换为字符串
    String userJson1 = objectMapper.writeValueAsString(new User("shanzhu", 21));
    // 3.通过StringRedisTemplate将数据存入redis
    stringRedisTemplate.opsForValue().set("user:100",userJson1);
    // 4.通过key取出value
    String userJson2 = stringRedisTemplate.opsForValue().get("user:100");
    // 5.由于取出的值是String类型的Json字符串，因此我们需要通过Json序列化对象来转换为java对象
    User user = objectMapper.readValue(userJson2, User.class);
    // 6.打印结果
    System.out.println("user = " + user);
  }

}
```

因为使用StringRedisTemplate不能自动序列化，因此我们需要自己将实体类进行序列化，然后存储到redis中，然后拿回值的时候，进行反序列化拿回实体类的值，效果如意。

#### 总结两种序列化方法

RedisTemplate的两种序列化实践方案，两种方案各有各的优缺点，可以根据实际情况选择使用。

**方案一：**

1. 自定义RedisTemplate
2. 修改RedisTemplate的序列化器为GenericJackson2JsonRedisSerializer

**方案二：**

1. 使用StringRedisTemplate
2. 写入Redis时，手动把对象序列化为JSON
3. 读取Redis时，手动把读取到的JSON反序列化为对象