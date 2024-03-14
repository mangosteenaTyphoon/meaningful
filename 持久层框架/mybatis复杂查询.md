## 各种标签的使用

### 新增标签

如下代码所示为一个简单的新增使用：

```xml
<insert id="insertUseGeneratedKeys" useGeneratedKeys="true" keyProperty="id">
    INSERT INTO sys_user(user_name, user_password, user_email, user_info, head_img, create_time)
    VALUES (#{userName},#{userPassword},#{userEmail},#{userInfo},#{headImg,jdbcType=BLOB},#{createTime,jdbcType=TIMESTAMP})
</insert>
```

- `id`: 该属性指定了该插入语句的唯一标识符，用于在其他地方引用该语句（与mapper文件中的方法名对应）。
- `useGeneratedKeys="true"`: 这个属性指示MyBatis使用数据库生成的主键值。如果设置为`true`，则MyBatis将尝试获取由数据库自动生成的主键值，并将其赋值给指定的`keyProperty`属性。
- `keyProperty="id"`: 这个属性指定了要接收生成的主键值的Java对象的属性名。在这个例子中，生成的主键值将被赋给名为`id`的属性。
- `fetchSize`: 用于设置每次从数据库获取的记录数。这可以提高批量插入的性能。（比如批量插入1000条数据，fetchSize= 100 的时候，每次插入100条数据，插入10次）

### 全部标签

- `timeout`: 用于设置 SQL 语句执行的超时时间（以秒为单位），sql执行超时之后会返回异常。
- `flushCache`: 用于控制是否在插入操作之前清除缓存。默认情况下，MyBatis 会在插入操作之前清除缓存。
- `statementType`: 用于指定 SQL 语句的类型。默认情况下，MyBatis 使用预处理语句（`PREPARED`），但也可以设置为 `STATEMENT` 或 `CALLABLE`。

### 动态sql的使用

#### choose标签

`choose` 关键字通常用于 MyBatis 的动态 SQL 语句中，用于实现条件判断。它类似于 Java 中的 `switch` 语句，可以根据不同的条件选择执行不同的 SQL 片段。

```xml
<select id="findUsers" resultType="User">
  SELECT * FROM users
  <where>
    <choose>
      <when test="username != null and username != ''">
        AND username = #{username}
      </when>
      <when test="email != null and email != ''">
        AND email = #{email}
      </when>
      <otherwise>
        AND id = #{id}
      </otherwise>
    </choose>
  </where>
</select>
```

在这个示例中，`choose` 标签包含了三个 `when` 标签和一个 `otherwise` 标签。根据传入的参数，MyBatis 会选择执行其中一个 `when` 标签内的 SQL 片段。如果所有 `when` 标签的条件都不满足，那么将执行 `otherwise` 标签内的 SQL 片段。

#### set 标签

`<set>` 标签在 MyBatis 中用于构建动态的 SQL 更新语句。它的主要功能如下：

- **动态字段更新**：`<set>` 标签可以在 SQL 更新语句中添加动态的字段设置，这样可以根据条件决定是否更新某个字段，以及如何更新该字段。
- **去除多余的逗号**：如果 `<set>` 标签包含的内容为空，或者遇到以逗号结尾的情况，它会忽略这些多余的逗号，确保 SQL 语句的正确性。
- **与条件判断结合使用**：`<set>` 标签通常与 `<if>` 标签结合使用，以便在满足某些条件时才设置特定的字段值。

在实际使用中，`<set>` 标签可以帮助你构建更加灵活和动态的 SQL 更新语句，使得更新操作更加精确和高效。

```xml
<update id="updateUser" parameterType="com.example.User">
  UPDATE user
  <set>
    <if test="username != null">
      username = #{username},
    </if>
    <if test="age != null">
      age = #{age}
    </if>
  </set>
  WHERE id = #{id}
</update>
```

#### foreach 标签使用

`<foreach>` 标签在 MyBatis 中用于构建动态的 SQL 语句，它允许你根据传入的集合或数组进行迭代操作。

```xml
<select id="selectUsers" resultType="User">
  SELECT * FROM user
  <where>
    <foreach item="item" index="index" collection="ids" open="(" separator="," close=")">
      id = #{item}
    </foreach>
  </where>
</select>    
```

在这个例子中，我们使用 `<foreach>` 标签来构建一个查询用户的 SQL 语句。通过指定 `collection` 属性为 "ids"，表示我们要迭代的是一个名为 "ids" 的集合。`item` 属性表示每次迭代的元素，`index` 属性表示当前迭代的索引（可选）。

`open`、`separator` 和 `close` 属性分别用于指定迭代开始时的前缀、分隔符和结束时的后缀。在这个例子中，我们使用了圆括号作为前缀和后缀，逗号作为分隔符。

当执行这个查询时，MyBatis 会根据传入的 "ids" 集合中的每个元素生成对应的 SQL 片段，最终生成类似于以下的 SQL 语句：

```mysql
SELECT * FROM user WHERE id IN (1, 2, 3);
```

## 嵌套查询

使用association标签实现嵌套查询。

`<association>` 标签在 MyBatis 中用于处理一对一关系的数据映射。它通常与 `<resultMap>` 标签一起使用，用于将查询结果中的关联对象映射到 Java 对象中。

假设我们有两个表，一个是用户表（user），包含字段 id、username、email；另一个是用户配置文件表（profile），包含字段 id、phone、address。这两个表之间存在一对一关系，即每个用户都有一个对应的配置文件。

首先，我们需要定义两个 Java 类来表示这两个表的数据：

```java
public class User {
  private int id;
  private String username;
  private String email;
  private Profile profile;
  // getter 和 setter 方法省略
}

public class Profile {
  private int id;
  private String phone;
  private String address;
  // getter 和 setter 方法省略
}

```

接下来，在 MyBatis 的映射文件中定义一个查询语句，并使用 `<association>` 标签进行数据映射：

```xml
<select id="selectUserWithProfile" resultMap="userResultMap">
  SELECT u.id as user_id, u.username, u.email, p.id as profile_id, p.phone, p.address
  FROM user u
  INNER JOIN profile p ON u.id = p.user_id
  WHERE u.id = #{userId}
</select>

<resultMap id="userResultMap" type="User">
  <id property="id" column="user_id"/>
  <result property="username" column="username"/>
  <result property="email" column="email"/>
  <association property="profile" javaType="Profile">
    <id property="id" column="profile_id"/>
    <result property="phone" column="phone"/>
    <result property="address" column="address"/>
  </association>
</resultMap>

```

```
    
```

在这个例子中，我们使用 `<association>` 标签将查询结果中的关联对象映射到 User 类的 profile 属性中。通过指定 `property` 为 "profile"，`javaType` 为 "Profile"，MyBatis 会自动将查询结果中的关联对象映射到 User 对象的 profile 属性中。

当我们执行 `selectUserWithProfile` 查询时，MyBatis 会根据传入的参数查询数据库，并将查询结果映射到 User 对象中，包括关联的 Profile 对象。这样，我们就可以方便地获取用户的详细信息，包括其关联的配置文件信息。

可以通过设置 `<association>` 标签的 `fetchType` 属性来实现懒加载。

`fetchType` 属性用于指定关联对象的加载方式，其取值可以是以下几种：

- `lazy`：表示使用懒加载，只有当访问到关联对象时才会进行加载。
- `eager`：表示立即加载，即在查询主对象时就会同时加载关联对象。

默认情况下，`fetchType` 属性的值为 `eager`，即立即加载关联对象。如果需要实现懒加载，可以将 `fetchType` 设置为 `lazy`。

## 一对多查询

在MyBatis中，一对多查询通常是指一个对象包含多个子对象的情况。例如，一个订单（Order）包含多个订单项（OrderItem）。下面是一个简单的例子：

首先，创建两个实体类：Order和OrderItem。

```java
public class Order {
    private int id;
    private String orderNo;
    private Date createTime;
    private List<OrderItem> items;
    // getter和setter方法
}

public class OrderItem {
    private int id;
    private int orderId;
    private String productName;
    private double price;
    // getter和setter方法
}
```

要使用MyBatis进行一对多查询，首先需要在映射文件中定义关联查询。以下是一个简单的示例：

1. 在OrderMapper.xml文件中添加关联查询的SQL语句：

   ```java
   <mapper namespace="com.example.mapper.OrderMapper">
       <!-- 其他映射语句 -->
   
       <resultMap id="OrderResultMap" type="com.example.entity.Order">
           <id property="id" column="order_id"/>
           <result property="orderNo" column="order_no"/>
           <result property="createTime" column="create_time"/>
           <collection property="items" ofType="com.example.entity.OrderItem" select="com.example.mapper.OrderItemMapper.selectByOrderId" column="order_id"/>
       </resultMap>
   
       <select id="selectOrderWithItems" resultMap="OrderResultMap">
           SELECT * FROM orders
       </select>
   </mapper>
   ```

2. 在OrderItemMapper.xml文件中添加根据订单ID查询订单项的SQL语句：

   ```java
   <mapper namespace="com.example.mapper.OrderItemMapper">
       <!-- 其他映射语句 -->
   
       <select id="selectByOrderId" resultType="com.example.entity.OrderItem">
           SELECT * FROM order_items WHERE order_id = #{orderId}
       </select>
   </mapper>
   ```

3. 在OrderMapper接口中添加查询方法：

   ```java
   public interface OrderMapper {
       // 其他方法
       List<Order> selectOrderWithItems();
   }
   ```

4. 在Service层调用OrderMapper的查询方法：

   ```java
   @Service
   public class OrderService {
       @Autowired
       private OrderMapper orderMapper;
   
       public List<Order> getOrdersWithItems() {
           return orderMapper.selectOrderWithItems();
       }
   }
   ```

   
