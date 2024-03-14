---
title: Redis实战之缓存解析
date: 2022-12-15
tags:
 - Redis
categories:
 - NoSQL
 - Redis
---

缓存就是数据交换的缓冲区(称为Cache),是存储数据的临时地方，一般读写性能较高。

### Redis作为缓存

Redis缓存，属于服务器缓存。其实就是把Redis当作数据库的缓存，因为Redis是在内存上存储的，比在硬盘上存储的数据库运行速度快很多。客户端发起查询数据的请求，都先去Redis中查询，Redis中若有，则直接返回，反之没有，那就再去数据库查询，数据库查询到以后，再存入Redis中，然后返回。以此往复，Redis中的数据越来越多，命中率也越来越大，大大加快了服务端的响应速度。

### 缓存作用模型

缓存作用先从`redis`中拿取想要的数据，如果`reids`中没有命中，则去`mysql`数据库中查找，如果还未找到数据则返回数据查找失败信息，模型如下图。

![img](https://shanzhu-edu.oss-cn-shanghai.aliyuncs.com/image.png)

### 实际代码中的应用

给用户信息接口增加缓存，如下代码块：

```Java
    public R getUserById(String id) {
        String key="cache:user"+id;
        //从redis拿用户信息
        String userJson = stringRedisTemplate.opsForValue().get(key);
        //判断redis里是否存在
        if(StrUtil.isNotBlank(userJson)){
            //存在的话 直接返回
            User user = JSONUtil.toBean(userJson, User.class);
            return R.ok().data("user",user);
        }

        //没有的话 去数据查
        User user = getById(id);
        if(user==null){
          return R.error().message("没有该用户~");
        }
        //有的话写进Redis
        stringRedisTemplate.opsForValue().set(key,JSONUtil.toJsonStr(user));
        return R.ok().data("user",user);
    }
}
```

以上代码块用到了HuTool工具，其maven坐标如下：

```XML
    <dependency>
            <groupId>cn.hutool</groupId>
            <artifactId>hutool-all</artifactId>
            <version>5.4.7</version>
    </dependency>
```

我们通过尝试访问上面接口，在第一次查询是从数据库中，

```SQL
Creating a new SqlSession
SqlSession [org.apache.ibatis.session.defaults.DefaultSqlSession@437365c5] was not registered for synchronization because synchronization is not active
JDBC Connection [HikariProxyConnection@1241810483 wrapping com.mysql.cj.jdbc.ConnectionImpl@1230ed6a] will not be managed by Spring
==>  Preparing: SELECT id,name,age,sex,address,is_delete FROM user WHERE id=? AND is_delete=0 
==> Parameters: 1552213274989453313(String)
<==    Columns: id, name, age, sex, address, is_delete
<==        Row: 1552213274989453313, 王二, 11, 女, 陕西省咸阳市, 0
<==      Total: 1
Closing non transactional SqlSession [org.apache.ibatis.session.defaults.DefaultSqlSession@437365c5]
```

第二次查询后则从redis中进行查询。并没有sql语句执行。**那么问题来了**，如果我们此时修改了数据呢

我们修改姓名为王一：

```SQL
{
  "success": true,
  "code": 20000,
  "message": "修改成功~",
  "data": {}
}
```

再次查询，我们得到的值依旧是"王二"

```SQL
{
  "success": true,
  "code": 20000,
  "message": "成功",
  "data": {
    "user": {
      "id": "1552213274989453313",
      "name": "王二",
      "age": 11,
      "sex": "女",
      "address": "陕西省咸阳市",
      "isDelete": 0
    }
  }
}
```

这就涉及到缓存**缓存更新策略**。

### 缓存更新策略

#### 内存淘汰

不用自己维护，利用redis的内存淘汰机制，当内存不足时自动淘汰部分数据，下次查询时更新缓存。

#### 超时剔除

给缓存数据添加TTL时间，到期后自动删除缓存。下次查询时更新缓存

#### 主动更新

编写业务逻辑，在修改数据库的同时，更新缓存。

调用者只操作缓存，由其他线程异步的将魂村数据

![img](https://shanzhu-edu.oss-cn-shanghai.aliyuncs.com/image.png)

我们主要说一下**主动更新策略**。

### 主动更新策略

*在企业中主动更新策略主要分为以下3种：*

#### Cache Aside Pattern

由缓存的调用者，在更新数据库的同时更新缓存。

#### Read/Write Through  Pattern

缓存与数据库合为一个服务，有服务来维护一致性，调用者调用该服务吗，无需关心缓存一致性问题。

#### Wrtie Behind Caching Pattern

调用者只操作缓存，由其它线程异步的将缓存数据持久化到数据库中，保持最终一致。

#### 操作缓存和数据库需要考虑的问题

更新缓存的操作的实现是 删除缓存还是修改缓存？

删除缓存，因为如果是修改缓存的话，每次更新数据库都要更新缓存，但是若这期间若一个缓存值修改多次，期间却没有几次读操作，那么期间的无效写操作过多，浪费系统性能。而删除缓存，在每次更新数据库时都删除缓存，等之后有请求需要查找这条被删除的缓存数据时再去数据库中查找出来并再次写入缓存，那么这样不会有无效写操作，同时也最大节省了系统性能。

如何保证缓存与数据库的操作同时成功或失败(即原子性)？

单个系统：将缓存与数据库操作放在一个事务内

分布式系统：利用TCC等分布式事务方案

先操作缓存还是先操作数据库？

1.先删除缓存，再操作数据库

2.先操作数据库，再删除缓存

一般情况下选择2、**先操作数据库，再删除缓存**。由以下两图可得，不管是先缓存后数据库，还是先数据库后缓存，都会有脏数据的出现，但是因为Redis缓存是基于内存的，数据库是基于磁盘的，所以**操作缓存的速度是远大于操作数据库的，出现脏数据概率更小的是先数据库后缓存**。同时，这里出现的脏数据问题，可以通过超时剔除兜底，来缓解脏数据带来的影响。

#### 代码实战

在修改用户信息接口上添加缓存更新

```Java
  public R updateUserById(User user) {
        String id = user.getId();
        String key="cache:user"+id;
        boolean flag = updateById(user);
        if(flag){
            stringRedisTemplate.delete(key);
            return R.ok().message("修改成功~");
        }
        return R.error().message("修改失败~");
    }
```

在查询用户信息上缓存增加时间

```Java
 stringRedisTemplate.opsForValue().set(key,JSONUtil.toJsonStr(user),30L, TimeUnit.MINUTES);
```

#### 总结

在需要高一致性的场景下，我们一般都会选择主动更新策略以及超时剔除策略兜底，并且更新缓存的操作为删除缓存、先操作数据库再删除缓存。

### 缓存更新策略的最佳实践方案

低一致性需求：使用Redis自带的内存淘汰机制，视情况选择以超时剔除作为兜底方案

高一致性需求：主动更新，并以超时剔除作为兜底方案

- 读操作：

  缓存命中则直接返回

  缓存未命中则查询数据库，并写入缓存，设定超时时间

- 写操作：

  先写数据库，然后再删除缓存

  要确保数据库与缓存操作的原子性

### 缓存穿透

缓存穿透是指客户端请求的数据在缓存中和数据库中都不存在，这样缓存永远不会生效，这些请求永远都会打到数据库，占**用系统I/O性能**。要是有人利用这个原理，恶意多次请求这个不存在的数据，就会使系统宕机。

![img](https://shanzhu-edu.oss-cn-shanghai.aliyuncs.com/image.png)

#### 常见的解决方法

**缓存空对象**

实现原理：数据库中不存在也缓存到redis中，缓存值为null，可以设置较短的TTL，防止长期的数据不一致

优点：实现简单，维护方便

缺点：额外的内存消耗，可能造成短期的不一致

![img](https://shanzhu-edu.oss-cn-shanghai.aliyuncs.com/image.png)

**布隆过滤**

实现原理：基于Hash算法实现的二进制字节数组

优点：内存占用较少，没有多余key

缺点：自行实现复杂(好在Redis提供了BitMap类型，自带的一种布隆过滤器),存在误判可能(他判断不存在的值一定是不存在的，但是判断存在的值不一定真的存在)。

![img](https://shanzhu-edu.oss-cn-shanghai.aliyuncs.com/image.png)

#### 总结

- 缓存穿透产生的原因是什么？

  用户请求的数据在缓存中和数据库中**都不存在**，不断发起这样的请求，给数据库带来**巨大压力**。

- 缓存穿透的解决方案有哪些？

  缓存null值、布隆过滤的解决方案都属于被动解决。我们还有很多主动的解决方案，比如：

  - 给id设置一定的**格式规律**，同时增强id的复杂度，避免被猜测id规律，然后在此基础上每次都先对id进行**基础格式校验**。，这样就可以做到主动解决了。
  - 加强用户**权限校验**
  - 做好热点参数的**限流**

### 缓存雪崩

> 缓存雪崩是指在**同一时段大量**的**缓存key同时失效**或者**Redis服务宕机**时，导致大量请求直接到达数据库，带来巨大压力。

![img](https://shanzhu-edu.oss-cn-shanghai.aliyuncs.com/image.png)

![img](https://shanzhu-edu.oss-cn-shanghai.aliyuncs.com/image.png)

**解决方案**

- 给不同的Key的TTL添加随机值(尤其是缓存预热的时候，同时加入大批量数据)
- 利用Redis集群提高服务的可用性(高级篇会讲)：多台Redis服务器，可以实现多台宏观调控
- 给缓存业务添加降级限流策略(SpringCloud中学)：当服务器请求压力大时，直接拦截请求快速返回，不再让服务器承受那么多请求
- 给业务添加多级缓存(高级篇会讲)：就是多端缓存，利用本地缓存等。

### 缓存击穿

> 缓存击穿问题也叫热点Key问题，就是一个被**高并发访问**并且**缓存重建业务较复杂**的key突然失效了，无数的请求访问会在瞬间给数据库带来巨大的冲击。

**解决方法**

**互斥锁**：在缓存未命中后，线程准备去查询数据库重建缓存数据之前，获取一个互斥锁，除了第一个线程拿到锁以外，其他后来的线程拿互斥锁失败以后会休眠一段时间，然后再从查询缓存从新走流程。

![img](https://shanzhu-edu.oss-cn-shanghai.aliyuncs.com/image.png)

**逻辑过期**：设置热点key永不过期，但是把expire过期时间属性写入value中，而key的过期时间是(写入该数据当时时间加上过期时间的时间戳)。这样每次查询缓存的时候都可以拿到缓存数据，然后通过value中的expire属性来判断当前这个缓存是否已经过期。若已经过期，那就先获取互斥锁，然后在这个线程里开启新的线程，让这个新的线程去执行查询数据库重建缓存数据的任务，而原来的线程就直接返回以及过期数据先用着，在新线程释放锁之前(也有可能是写入缓存之前)，其他线程来查询缓存，都会因为因为尝试获取互斥锁失败后返回过期数据继续用着先。