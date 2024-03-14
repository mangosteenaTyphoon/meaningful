## 1.对常用组件和配置的封装

这里自定义注入了`redisTemplate`和基于spring cache的自定义配置，使用json序列化value

```java
  /**
     *  注入一个redisTemplate bean，使用json序列化value
     * @param factory
     * @return
     */
    @Bean
    @ConditionalOnMissingBean({RedisTemplate.class})
    public  RedisTemplate<Object, Object> redisTemplate(RedisConnectionFactory factory) {
        RedisTemplate<Object, Object> template = new RedisTemplate<Object, Object>();
        template.setConnectionFactory(factory);
        template.setKeySerializer(new StringRedisSerializer());
        template.setHashKeySerializer(new StringRedisSerializer());
        template.setDefaultSerializer(new Jackson2JsonRedisSerializer<>(Object.class));
        template.setHashValueSerializer(new Jackson2JsonRedisSerializer<>(Object.class));
        return template;
    }

    @Bean
    @ConditionalOnMissingBean({StringRedisTemplate.class})
    public StringRedisTemplate stringRedisTemplate(RedisConnectionFactory redisConnectionFactory) {
        StringRedisTemplate template = new StringRedisTemplate();
        template.setConnectionFactory(redisConnectionFactory);
        return template;
    }

    @Bean
    public RedisCacheConfiguration redisCacheConfiguration(CacheProperties cacheProperties) {
        RedisCacheConfiguration config = RedisCacheConfiguration.defaultCacheConfig();
        // config = config.entryTtl();
        config = config.serializeKeysWith(RedisSerializationContext.SerializationPair.fromSerializer(new StringRedisSerializer()));
        config = config.serializeValuesWith(RedisSerializationContext.SerializationPair.fromSerializer(new GenericJackson2JsonRedisSerializer()));

        CacheProperties.Redis redisProperties = cacheProperties.getRedis();
        //将配置文件中所有的配置都生效
        if (redisProperties.getTimeToLive() != null) {
            config = config.entryTtl(redisProperties.getTimeToLive());
        }
        if (redisProperties.getKeyPrefix() != null) {
            config = config.prefixCacheNameWith(redisProperties.getKeyPrefix());
        }
        if (!redisProperties.isCacheNullValues()) {
            config = config.disableCachingNullValues();
        }
        if (!redisProperties.isUseKeyPrefix()) {
            config = config.disableKeyPrefix();
        }
        return config;
    }
```

## 2.基于redis实现分布式限流

这里是基于Redis + Lua 的偏业务应用的分布式固定时间窗口算法限流，通过`自定义注解`、`aop`、`Redis + Lua` 实现，使得项目拥有分布式限流能力变得很简单。

**限流类型枚举类**：[LimitType](https://github.com/plasticene/plasticene-boot-starter-parent/blob/main/plasticene-boot-starter-redis/src/main/java/com/plasticene/boot/redis/core/enums/LimitType.java)支持根据自定义key或者请求IP限流

```java
public enum LimitType {

    /**
     * 自定义key
     */
    CUSTOMER,

    /**
     * 请求者IP
     */
    IP;
}
```

**自定义注解**

我们自定义个`@RateLimit`注解，注解类型为`ElementType.METHOD`即作用于方法上。

`period`表示请求限制时间段，`count`表示在`period`这个时间段内允许放行请求的次数。`limitType`代表限流的类型，可以根据`请求的IP`、`自定义key`，如果不传`limitType`属性则默认用方法名作为默认key。

```java
@Target({ElementType.METHOD, ElementType.TYPE})
@Retention(RetentionPolicy.RUNTIME)
@Inherited
@Documented
public @interface RateLimit {

    /**
     * 名字
     */
    String name() default "";

    /**
     * key
     */
    String key() default "";

    /**
     * Key的前缀
     */
    String prefix() default "";

    /**
     * 给定的时间范围 单位(秒)
     */
    int period();

    /**
     * 一定时间内最多访问次数
     */
    int count();

    /**
     * 限流的类型(用户自定义key 或者 请求ip，默认为自定义key)
     */
    LimitType limitType() default LimitType.CUSTOMER;
}
```

**切面代码实现**

```java
@Aspect
@Order(OrderConstant.AOP_RATE_LIMIT)
public class RateLimitAspect {

    private static final Logger logger = LoggerFactory.getLogger(RateLimitAspect.class);

    private static final String UNKNOWN = "unknown";

    @Resource
    private StringRedisTemplate stringRedisTemplate;


    /**
     * @param pjp
     * @description 切面
     */
    @Around("execution(public * *(..)) && @annotation(com.plasticene.boot.redis.core.anno.RateLimit)")
    public Object interceptor(ProceedingJoinPoint pjp) throws Throwable {
        MethodSignature signature = (MethodSignature) pjp.getSignature();
        Method method = signature.getMethod();
        RateLimit limitAnnotation = method.getAnnotation(RateLimit.class);
        LimitType limitType = limitAnnotation.limitType();
        String name = limitAnnotation.name();
        String key;
        int limitPeriod = limitAnnotation.period();
        int limitCount = limitAnnotation.count();

        /**
         * 根据限流类型获取不同的key ,如果不传我们会以方法名作为key
         */
        switch (limitType) {
            case IP:
                key = getIpAddress();
                break;
            case CUSTOMER:
                key = limitAnnotation.key();
                if (StrUtil.isBlank(key)) {
                    key = signature.getDeclaringTypeName() + "." + signature.getName();
                }
                break;
            default:
                key = signature.getDeclaringTypeName() + "." + signature.getName();
        }
        List<String> keys = ListUtil.of(StrUtil.join(limitAnnotation.prefix(), key));
        String luaScript = buildLuaScript();
        RedisScript<Number> redisScript = new DefaultRedisScript<>(luaScript, Number.class);
        Number count = stringRedisTemplate.execute(redisScript, keys, String.valueOf(limitCount), String.valueOf(limitPeriod));
        logger.info("Access try count is {} for name={} and key = {}", count, name, key);
        if (count != null && count.intValue() <= limitCount) {
            return pjp.proceed();
        } else {
            throw new BizException("Too Many Requests");
        }

    }

    /**
     * @description 编写 redis Lua 限流脚本
     */
    public String buildLuaScript() {
        StringBuilder lua = new StringBuilder();
        lua.append("local c");
        lua.append("\nc = redis.call('get',KEYS[1])");
        // 当前调用累计超过最大值，则直接返回，拒绝请求
        lua.append("\nif c and tonumber(c) > tonumber(ARGV[1]) then");
        lua.append("\nreturn c;");
        lua.append("\nend");
        // 执行计算器自加
        lua.append("\nc = redis.call('incr',KEYS[1])");
        lua.append("\nif tonumber(c) == 1 then");
        // 从第一次调用开始限流，设置对应键值的过期
        lua.append("\nredis.call('expire',KEYS[1],ARGV[2])");
        lua.append("\nend");
        lua.append("\nreturn c;");
        return lua.toString();
    }


    public String getIpAddress() {
        HttpServletRequest request = ((ServletRequestAttributes) RequestContextHolder.getRequestAttributes()).getRequest();
        String ip = request.getHeader("x-forwarded-for");
        if (ip == null || ip.length() == 0 || UNKNOWN.equalsIgnoreCase(ip)) {
            ip = request.getHeader("Proxy-Client-IP");
        }
        if (ip == null || ip.length() == 0 || UNKNOWN.equalsIgnoreCase(ip)) {
            ip = request.getHeader("WL-Proxy-Client-IP");
        }
        if (ip == null || ip.length() == 0 || UNKNOWN.equalsIgnoreCase(ip)) {
            ip = request.getRemoteAddr();
        }
        return ip;
    }
}
```

**使用示例**

```java
    @RateLimit( period = 10, count = 3)
    @GetMapping("/test1")
    public int testLimiter1() {
        return 1;
    }

    @RateLimit(key = "customer_limit_test", period = 10, count = 3, limitType = LimitType.CUSTOMER)
    @GetMapping("/test2")
    public int testLimiter2() {
        return 1;
    }

    @RateLimit(key = "ip_limit_test", period = 10, count = 3, limitType = LimitType.IP)
    @GetMapping("/test3")
    public int testLimiter3() {
        return 1;
    }
```

## 3.基于redisson实现分布式锁

目前有很多项目还在使用redis的 `setNx`命令实现分布式锁，并不可靠，在某些情况会出现死锁，锁控制错乱等问题。[redisson](https://github.com/redisson/redisson)是java支持redis的redlock的`唯一`实现。`redisson`目前是官方唯一推荐的java版的分布式锁。

基于Redis的Redisson分布式可重入锁[`RLock`](http://static.javadoc.io/org.redisson/redisson/3.10.0/org/redisson/api/RLock.html) Java对象实现了`java.util.concurrent.locks.Lock`接口，所以其使用套路就和juc并发包下lock一样。如果对redisson不太了解的，可自行浏览[官方文档](https://github.com/redisson/redisson/wiki/目录)学习使用非常容易上手，或者阅读本人整理的[《分布式锁实现方案：Redisson》](http://www.shepherd126.top/archives/分布式锁实现方案redisson)文章

redis的`setNx`命令实现分布式锁存在问题示例如下：下面是我一个mall项目查询商品分类树的方法，不需要过多关注该方法业务，我们只需要关心分布锁的实现即可。

```java
  List<CategoryDTO> getCategoryTreeWithRedisLock() {

        //1、占分布式锁。去redis占坑 设置过期时间必须和加锁是同步的，保证原子性（避免死锁）
        String uuid = UUID.randomUUID().toString();
        Boolean lock = stringRedisTemplate.opsForValue().setIfAbsent("lock", uuid, 300, TimeUnit.SECONDS);
        if (lock) {
            log.info("获取分布式锁成功...");
            List<CategoryDTO> categoryDTOList = null;
            try {
                String categoryJson = stringRedisTemplate.opsForValue().get(CATEGORY_CACHE);
                if (StringUtils.isBlank(categoryJson)) {
                    //加锁成功...，并且redis还没有数据库，执行业务
                    categoryDTOList = getCategoryTree();
                    stringRedisTemplate.opsForValue().set(CATEGORY_CACHE, JSON.toJSONString(categoryDTOList), 5, TimeUnit.MINUTES);
                } else {
                    categoryDTOList = JSON.parseArray(categoryJson, CategoryDTO.class);
                }
            } finally {
                // lua 脚本解锁
                String script = "if redis.call('get', KEYS[1]) == ARGV[1] then return redis.call('del', KEYS[1]) else return 0 end";
                // 删除锁
                stringRedisTemplate.execute(new DefaultRedisScript<>(script, Long.class), Collections.singletonList("lock"), uuid);
            }
            //先去redis查询下保证当前的锁是自己的
            //获取值对比，对比成功删除=原子性 lua脚本解锁
            // String lockValue = stringRedisTemplate.opsForValue().get("lock");
            // if (uuid.equals(lockValue)) {
            //     //删除我自己的锁
            //     stringRedisTemplate.delete("lock");
            // }
            return categoryDTOList;
        } else {
            log.info("获取分布式锁失败...等待重试...");
            //加锁失败...重试机制
            try {
                TimeUnit.MILLISECONDS.sleep(10);
            } catch (InterruptedException e) {
                log.error("redis分布式锁发生错误", e);
            }
            return tryAgainWithTime();
        }
    }
```

redis实现分布式锁，使用命令set key value [EX seconds] [NX|XX]，但有如下问题：

- 加锁和设置过期时间必须是原子性的，不然有可能加锁之后还没有执行到设置过期时间代码时服务不可用，锁一直不释放，造成死锁-
- 主动删除key，即解锁需要注意：如果在key设置的过期时间之前删除key那么没问题，测试业务执行完正常解锁，但是如果删除key在过期之后就有问题了，此时当前线程的锁已经因为过期自动解锁，另外的请求线程拿到锁，所以这时候删除的不是当前线程的锁， 解决方案：利用CAS原理，删除之前先比较value值是不是之前存放进去的，这里为了保证每次的value都不一样，使用uuid生成value，但是这时候有一种情况就是你去拿value的时候锁没有过期，此时拿到value和传入的一样，但是当你刚刚获取完value之后锁就过期了，其他请求线程拿到锁，然后你再根据传入的值和获取的value值去删除锁，这时候删除的是其他请求线程的锁，造成问题，所以利用CAS原理值比较和删除锁必须是原子性操作。

当然上面代码并不是一定就会出现上述问题的，一般情况下是可以正常使用的，但是在某些情况下会出现，如并发量大的时候。

下面通过**`自定义注解`、`aop`、`redisson` 优雅地实现分布式锁**：

**锁类型枚举类：**

```java
public enum LockType {
    //可重入锁
    REENTRANT,
    //公平锁
    FAIR,
    //读锁
    READ,
    //写锁
    WRITE
}
```

**自定义注解**

```java
@Target({ElementType.METHOD, ElementType.TYPE})
@Retention(RetentionPolicy.RUNTIME)
@Documented
public @interface DistributedLock {
    /**
     * 锁的key
     */
    String key();
    /**
     * 获取锁的最大尝试时间(单位 {@code unit})
     * 该值大于0则使用 locker.tryLock 方法加锁，否则使用 locker.lock 方法
     */
    long waitTime() default 0;
    /**
     * 加锁的时间(单位 {@code unit})，超过这个时间后锁便自动解锁；
     * 如果leaseTime为-1，则保持锁定直到显式解锁
     */
    long leaseTime() default -1;
    /**
     * 参数的时间单位
     */
    TimeUnit unit() default TimeUnit.SECONDS;


    /**
     * 锁类型，默认为可重入锁
     * @return
     */
    LockType lockType() default LockType.REENTRANT;

}
```

**切面代码实现**

```java
@Aspect
@Order(OrderConstant.AOP_LOCK)
public class LockAspect extends AbstractAspectSupport {

    private static final Logger logger = LoggerFactory.getLogger(LockAspect.class);

    @Resource
    private RedissonClient redissonClient;

    // 指定切入点为DistributedLock注解
    @Pointcut("@annotation(com.plasticene.boot.redis.core.anno.DistributedLock)")
    public void distributedLockAnnotationPointcut() {
    }

    // 环绕通知
    @Around("distributedLockAnnotationPointcut()")
    public Object aroundLock(ProceedingJoinPoint pjp) {
        Method method = resolveMethod(pjp);
        DistributedLock distributedLock = method.getAnnotation(DistributedLock.class);
        String lockKey = distributedLock.key();
        if (StrUtil.isBlank(lockKey)) {
            throw new BizException("lock Key 不能为空");
        }
        // 加锁等待时间 大于0就调用redisson的tryLock()
        long waitTime = distributedLock.waitTime();
        // 锁自动过期时间
        long leaseTime = distributedLock.leaseTime();
        TimeUnit unit = distributedLock.unit();
        // 锁类型
        LockType lockType = distributedLock.lockType();
        logger.info("锁模式->{},  等待锁定时间->{}秒,   锁定最长时间->{}秒",lockType.name(), waitTime, leaseTime);
        RLock rLock = getLock(lockType, lockKey);
        boolean flag = true;
        try {
            if (waitTime > 0) {
                flag = rLock.tryLock(waitTime, leaseTime, unit);
            } else {
                rLock.lock(leaseTime, unit);
            }
            if (flag) {
                return pjp.proceed();
            } else {
                throw new BizException("获取锁等待超时");
            }
        } catch (Throwable e) {
            logger.error("加锁异常：", e);
            if (e instanceof BizException) {
                throw new BizException(e.getMessage());
            }
            throw new BizException("加锁失败");
        } finally {
            if (flag) {
                rLock.unlock();
            }
        }
    }

    /**
     * 可重入锁
     * 阻塞式等待。默认加的锁都是30s
     * 1）、锁的自动续期，如果业务超长，运行期间自动锁上新的30s。不用担心业务时间长，锁自动过期被删掉
     * 2）、加锁的业务只要运行完成，就不会给当前锁续期，即使不手动解锁，锁默认会在30s内自动过期，不会产生死锁问题
     * leaseTime为锁超时时间，leaseTime为-1代表不自动解锁
     * 1）、myLock.lock(10,TimeUnit.SECONDS);  10秒钟自动解锁,自动解锁时间一定要大于业务执行时间
     * 2）、如果没有指定锁的超时时间，就使用 lockWatchdogTimeout = 30 * 1000 【看门狗默认时间】。只要占锁成功，就会启动一个
     *  定时任务【重新给锁设置过期时间，新的过期时间就是看门狗的默认时间】,每隔10秒都会自动的再次续期，续成30秒
     * internalLockLeaseTime 【看门狗时间】 / 3， 10s
     *
     * 读写锁
     * 保证一定能读到最新数据，修改期间，写锁是一个排它锁（互斥锁、独享锁），读锁是一个共享锁
     * 写锁没释放读锁必须等待
     * 读 + 读 ：相当于无锁，并发读，只会在Redis中记录好，所有当前的读锁。他们都会同时加锁成功
     * 写 + 读 ：必须等待写锁释放
     * 写 + 写 ：阻塞方式
     * 读 + 写 ：有读锁。写也需要等待
     * 只要有读或者写的存都必须等待
     * @return
     */

    RLock getLock(LockType lockType, String lockKey) {
        // 获取一把锁，只要锁的名字一样，就是同一把锁
        RLock rLock = null;
        switch (lockType) {
            case FAIR:
                rLock = redissonClient.getFairLock(lockKey);
                break;
            case REENTRANT:
                rLock = redissonClient.getLock(lockKey);
                break;
            case READ:
                RReadWriteLock readWriteLock = redissonClient.getReadWriteLock(lockKey);
                rLock = readWriteLock.readLock();
                break;
            case WRITE:
                RReadWriteLock rwLock = redissonClient.getReadWriteLock(lockKey);
                rLock = rwLock.writeLock();
                break;
            default:
                rLock = redissonClient.getLock(lockKey);
        }
        return rLock;

    }

}
```

注意，这里只封装注入了单节点模式下的redisson client：

```java
    /**
     * 这里默认情况下条件装配注入一个单机模式的redisson client
     * 如需要其他模式，可在业务侧按要求自行注入redissonClient覆盖即可
     * @return
     */
    @Bean
    @ConditionalOnMissingBean(RedissonClient.class)
    public RedissonClient redissonClient() {
        // 1、创建配置
        Config config = new Config();
        String host = redisProperties.getHost();
        int port = redisProperties.getPort();
        config.useSingleServer().setAddress(REDISSON_PREFIX + host + ":" + port)
                .setDatabase(redisProperties.getDatabase())
                .setPassword(redisProperties.getPassword());
        // 2、根据 Config 创建出 RedissonClient 实例
        return Redisson.create(config);
    }
```

如果使用redis其他模式，比如说集群，请在业务侧自行注入对应模式的redisson client

**使用示例**

```java
    @DistributedLock(key = "distributed-lock")
    @GetMapping("/lock")
    public void lock() {
        log.info("加锁成功，执行业务..." + Thread.currentThread().getId());
        try {
            TimeUnit.SECONDS.sleep(60);
            log.info("执行结束：" + Thread.currentThread().getId());
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }
```

## 4.redis常规操作命令封装

对redis 基本操作进行封装、扩展，方面使用。详见[RedisUtils](https://github.com/plasticene/plasticene-boot-starter-parent/blob/main/plasticene-boot-starter-redis/src/main/java/com/plasticene/boot/redis/core/utils/RedisUtils.java)