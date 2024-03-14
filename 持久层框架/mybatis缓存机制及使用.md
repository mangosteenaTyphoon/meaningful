> mybatis是常见的Java数据库访问层框架。本文就mybatis的缓存机制进行记录学习。

## Mybatis缓存概览

当用户执行一个数据库查询时，MyBatis首先会查看缓存中是否已经有这个查询的结果。如果有，直接从缓存中获取数据，这样就避免了对数据库的访问，极大地提高了效率。如果没有，它才会执行SQL语句，然后将结果存入缓存。

Mybatis的缓存主要分为一级缓存和二级缓存。此外，MyBatis还支持第三方缓存插件，如Redis、Memcached等，这可以进一步提高缓存的效率和扩展性。同时，MyBatis也允许开发者自定义缓存实现，以满足特定的业务需求。

## 一级缓存深度解析

MyBatis的一级缓存是`SqlSession`级别的缓存，对于同一个`SqlSession`对象，在默认情况下会启用一级缓存机制，以提高查询效率和性能。 当多次查询相同的SQL，并且**没有执行提交、回滚或清空缓存**的操作时，MyBatis会将第一次查询的结果缓存在`SqlSession`中，下次再执行相同查询时，将直接从缓存中获取数据，避免再次发送SQL查询。

 需要注意的是，一级缓存是在`SqlSession`对象范围内有效的，不同的`SqlSession`对象之间的缓存是独立的，所以如果是在不同的`SqlSession`对象中进行查询，一级缓存是不会起作用的。

一级缓存的**生命周期**和`SqlSession`一致。当`SqlSession`结束或关闭时，与之关联的一级缓存也就不存在了。

## 二级缓存深度解析

MyBatis的二级缓存是一个跨`SqlSession`的缓存机制，它可以跨`SqlSession`共享缓存数据，从而提高系统性能。它是多个`SqlSession`操作相同`namespace`的映射器（Mapper）时，它们可以共享同一个二级缓存区域。

#### 工作流程

- 当`SqlSession`执行查询时，如果开启了二级缓存，并且该Mapper配置了二级缓存，则会先到二级缓存中查找数据。
- 如果在二级缓存中找到了数据，就直接返回给用户，避免了对数据库的查询操作。
- 如果在二级缓存中没有找到数据，会去数据库中查询，并将查询结果存入二级缓存。
- 当`SqlSession`执行增删改操作时，会清空该Mapper对应的二级缓存，保证缓存数据的有效性。

#### 配置方式

- 开启二级缓存：在MyBatis配置文件（如`mybatis-config.xml`）中设置`<setting name="cacheEnabled" value="true"/>`来开启二级缓存。

  ```xml
  <configuration>
      <settings>
          <!-- 开启全局二级缓存 -->
          <setting name="cacheEnabled" value="true"/>
      </settings>
  </configuration>
  ```

- 配置Mapper的二级缓存：在Mapper XML文件中配置`<cache/>`元素来启用该Mapper对应的二级缓存。

  ```xml
  <mapper namespace="com.example.mapper.UserMapper">
      <!-- 开启这个Mapper的二级缓存 -->
      <cache/>
      <!-- 其他SQL映射语句 -->
  </mapper>
  ```

#### 生命周期

- **SqlSessionFactory级别**：二级缓存的生命周期与SqlSessionFactory的生命周期相同。当SqlSessionFactory被关闭时，二级缓存会被清空，缓存中的数据会被释放。
- **Mapper级别**：每个Mapper对应的二级缓存的生命周期也与`SqlSessionFactory`的生命周期相关。当对应的Mapper XML文件或接口被重新加载时，Mapper级别的二级缓存会被清空，缓存中的数据会被释放。

总之，MyBatis的二级缓存是在Mapper级别和`SqlSessionFactory`级别具有作用范围，并且其生命周期与对应的Mapper和`SqlSessionFactory`的生命周期相关联。遵循这些作用域和生命周期规则，可以保证缓存数据的有效性，避免脏数据的出现。

## 缓存策略

MyBatis提供了多种缓存策略，可以根据实际需求选择合适的缓存策略。

1. 默认缓存：MyBatis的默认缓存策略是基于`PerpetualCache`的，即永久缓存，不会自动清除缓存数据，除非手动调用`clearCache()`方法。默认缓存的适用范围是SqlSession级别，即同一个SqlSession对象中共享缓存数据。

   ```xml
   <!-- 在 Mapper XML 文件中配置默认缓存策略 -->
   <cache/>
   ```

2. LRU缓存：Least Recently Used（LRU）缓存是一种基于最近最少使用的缓存策略，当缓存达到一定大小时，会自动清理最近最少使用的缓存项。

   ```xml
   <!-- 配置LRU缓存策略，设置缓存大小为100 -->
   <cache type="org.apache.ibatis.cache.decorators.LruCache" size="100"/>
   ```

3. FIFO缓存：First In, First Out（FIFO）缓存是一种先进先出的缓存策略，当缓存达到一定大小时，会删除最早进入缓存的数据。

   ```xml
   <!-- 配置FIFO缓存策略，设置缓存大小为100 -->
   <cache type="org.apache.ibatis.cache.decorators.FifoCache" size="100"/>
   ```

4. 定时刷新缓存：MyBatis也提供了定时刷新缓存的方式，可以设置缓存刷新的时间间隔，保持缓存数据的实时性。

   ```xml
   <!-- 设置缓存每隔30分钟刷新一次 -->
   <cache flushInterval="1800000"/>
   ```

5. 软引用缓存（Soft Reference）：软引用是基于Java的SoftReference类实现的。当JVM内存充足时，软引用所指向的对象会保留在内存中；但当JVM内存不足时，垃圾回收器会回收软引用所指向的对象，以腾出内存空间。在MyBatis中，使用软引用作为缓存策略意味着缓存中的对象会在内存紧张时被回收，从而避免内存溢出的风险。

   在MyBatis中，软引用缓存和弱引用缓存并不直接支持，但你可以通过自定义缓存来实现软引用缓存或弱引用缓存。软引用缓存可以通过在自定义缓存实现中使用Java的`SoftReference`来实现。

   ```java
   import java.lang.ref.SoftReference;
   import java.util.HashMap;
   import java.util.Map;
   public class SoftReferenceCache implements Cache {
       private final Map<Object, SoftReference<Object>> cache = new HashMap<>();
       @Override
       public void putObject(Object key, Object value) {
           cache.put(key, new SoftReference<>(value));
       }
       @Override
       public Object getObject(Object key) {
           SoftReference<Object> softRef = cache.get(key);
           return (softRef != null) ? softRef.get() : null;
       }
       // 其他方法实现省略
   }

6. 弱引用缓存（Weak Reference）：弱引用是基于Java的`WeakReference`类实现的。与软引用不同，无论JVM内存是否充足，只要垃圾回收器运行，弱引用所指向的对象都会被回收。在MyBatis中，使用弱引用作为缓存策略会导致缓存中的对象在下一次垃圾回收时被清除，这通常意味着缓存的生命周期非常短暂。

   弱引用缓存可以通过在自定义缓存实现中使用Java的`WeakReference`来实现。

   ```java
   import java.lang.ref.WeakReference;
   import java.util.HashMap;
   import java.util.Map;
   public class WeakReferenceCache implements Cache {
       private final Map<Object, WeakReference<Object>> cache = new HashMap<>();
       @Override
       public void putObject(Object key, Object value) {
           cache.put(key, new WeakReference<>(value));
       }
       @Override
       public Object getObject(Object key) {
           WeakReference<Object> weakRef = cache.get(key);
           return (weakRef != null) ? weakRef.get() : null;
       }
       // 其他方法实现省略
   }
   ```

7. 自定义缓存：除了上述内置的缓存策略外，MyBatis还支持自定义缓存策略，开发者可以根据实际需求自行实现缓存接口来定制缓存策略。 通过选择合适的缓存策略，可以有效地提高系统性能，减少对数据库的频繁查询。

   在MyBatis中，自定义缓存策略需要实现`org.apache.ibatis.cache.Cache`接口。这个接口包含了缓存操作所需的基本方法，如`getObject`、`putObject`、`removeObject`等。

   ```java
   public class CustomCache implements Cache {
       // 缓存标识符
       private final String id;
   
       public CustomCache(String id) {
           this.id = id;
       }
   
       @Override
       public String getId() {
           return id;
       }
   
       @Override
       public void putObject(Object key, Object value) {
           // 实现添加缓存逻辑
       }
   
       @Override
       public Object getObject(Object key) {
           // 实现获取缓存逻辑
           return null;
       }
   
       @Override
       public Object removeObject(Object key) {
           // 实现移除缓存逻辑
           return null;
       }
   
       @Override
       public void clear() {
           // 实现清空缓存逻辑
       }
   
       @Override
       public int getSize() {
           // 实现获取缓存大小逻辑
           return 0;
       }
   }
   ```

## 缓存失效与维护

MyBatis中的缓存可能会由于以下情况而失效：

1. **并发更新、删除或插入**：当多个会话并发修改了数据库中的数据，造成缓存中的数据和数据库中的数据不一致时，缓存会失效。
2. **手动清除缓存**：如果在代码中手动清除了某个Mapper对应的缓存，缓存将会失效。
3. **执行了提交或回滚**：当MyBatis执行了提交（commit）或回滚（rollback）操作时，缓存会被清空。
4. **超过缓存大小限制**：如果启用了缓存的大小限制，并且缓存中的数据量超出了设定的阈值，会触发缓存的清理操作，导致缓存失效。
5. **Mapper映射文件或SQL语句发生改变**：当Mapper映射文件或SQL语句发生变化时，MyBatis会自动清空相应的缓存，以确保缓存数据与最新映射文件或SQL语句一致。
6. **配置的刷新间隔到期**：如果配置了缓存刷新间隔，当间隔到期时会刷新缓存，使缓存失效。
7. **使用了不同的SqlSession**(一级缓存)：如果在不同的SqlSession中执行了相同的查询，由于每个SqlSession有自己的缓存，会导致缓存失效。
8. **使用了不同的ExecutorType**：如果在不同的ExecutorType（比如SIMPLE、REUSE、BATCH）下执行同一个查询，由于每种ExecutorType有自己的缓存，会导致缓存失效。

## 总结

MyBatis的缓存机制，如果使用得当，可以显著提升应用的响应速度和处理能力。

- 合理选择缓存级别很重要，根据应用的具体需求选择最优缓存等级。
- 适当配置缓存参数（缓存大小、策略等）
- 对于多读少写的操作选择缓存比较好，如果是多写的数据尽量不缓存。



