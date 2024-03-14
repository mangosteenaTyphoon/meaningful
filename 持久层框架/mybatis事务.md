> 事务是在日常开发中十分重要的一环，MyBatis也提供了非常便捷的事务管理机制。

主要分为：事务在mybatis的主要管理方式、 几种方式如何使用、关于每种方式的深研究

## 概述

MyBatis作为一个持久层框架，其事务的特性是指每对数据库的一系列操作之后，要确保该系列操作中如果操作成功，那就该系列操作都成功，如果其中一个操作失败，将使数据库回滚至该系列操作前的状态，确保数据库中的数据一致性和完整性。

MyBatis中管理事务的方式分别如下：

1. **编程式事务管理**： 在编程式事务管理中，需要显式地在代码中控制事务的开始、提交和回滚。通过获取数据库连接、手动设置事务提交或回滚等操作来管理事务。虽然可以更精细地控制事务的行为，但需要编写更多的代码来处理事务，维护性较差。
2. **声明式事务管理**： 使用声明式事务管理，可以通过注解或XML配置文件来定义事务的行为。在MyBatis中，通常结合Spring的`@Transactional`注解或XML配置文件中的事务配置来实现声明式事务管理。这种方式更加简洁和方便，避免了手动管理事务的复杂性。
3. **基于AOP的事务管理**： 基于AOP的事务管理是声明式事务管理的一种实现方式。通过对方法进行拦截，AOP可以在方法执行前后加入事务处理逻辑，实现事务的切面功能。Spring框架提供了`TransactionInterceptor`来实现AOP事务管理。
4. **全局事务管理器**： MyBatis框架本身并不提供全局事务管理功能，但通常会结合Spring或其他框架提供的全局事务管理器（如JTA）来实现跨多个数据源的事务管理。全局事务管理器能够协调多个数据源的事务，实现分布式事务的统一管理。 

## 使用方法

#### 声明式事务管理

在使用SpringBoot开发项目时，可以通过使用声明式事务管理方式开启业务事务。这也是最常用的管理事务方式，它将事务管理和业务逻辑分开，降低代码的耦合度。

基于`@Transactional`注解：通过在方法上添加`@Transactional`注解来实现事务管理。

```java
@Service
public class UserService {
    @Autowired
    private UserRepository userRepository;
    @Transactional
    public void createUser(String username) {
        User user = new User();
        user.setUsername(username);
        userRepository.save(user);
    }
    @Transactional(propagation = Propagation.REQUIRED, isolation = Isolation.READ_COMMITTED)
    public void updateUser(String username, Long userId) {
        User user = userRepository.findById(userId).orElse(null);
        if (user != null) {
            user.setUsername(username);
            userRepository.save(user);
        }
    }
    @Transactional(rollbackFor = Exception.class)
    public void deleteUser(Long userId) throws Exception {
        User user = userRepository.findById(userId).orElse(null);
        if (user != null) {
            userRepository.delete(user);
        } else {
            throw new Exception("User not found");
        }
    }
}
```

在上面的代码中，UserService类中的三个方法分别用来创建用户、更新用户和删除用户，每个方法上都添加了`@Transactional`注解来开启事务。`@Transactional`注解是Spring框架提供的用于声明式事务管理的注解，可以应用在方法级别或类级别上。

#### 编程式事务管理

在Spring Boot中使用编程式事务管理，主要通过编写代码手动控制事务的开启、提交和回滚。

```java
@Service
public class UserService {
    @Autowired
    private UserRepository userRepository;
    
    @Autowired
    private PlatformTransactionManager transactionManager;
    
    public void createUserWithManualTransaction(String username) {
        TransactionStatus status = transactionManager.getTransaction(new DefaultTransactionDefinition());
        
        try {
            User user = new User();
            user.setUsername(username);
            userRepository.save(user);
            
            transactionManager.commit(status);
        } catch (Exception e) {
            transactionManager.rollback(status);
            throw e;
        }
    }
}
```

在上述代码中，`createUserWithManualTransaction`方法使用编程式事务管理来创建用户。在方法内部，首先通过`transactionManager.getTransaction`方法获取事务的起始点，并返回一个`TransactionStatus`对象，然后在try-catch块内进行业务的处理，最后根据业务结果调用`transactionManager.commit`提交事务或者调用`transactionManager.rollback`回滚事务。

## Transactional注解

`@Transactional`注解是Spring框架中用于实现声明式事务管理的一种方式。通过在方法或类上添加`@Transactional`注解，告诉Spring框架需要对被注解的方法或类进行事务管理。Spring框架会根据注解的配置来自动处理事务的开始、提交和回滚，从而实现简化事务管理的目的。当Spring框架与mybatis结合使用时，mybatis可以通过Spring的事务管理器来统一管理MyBatis的事务。当调用MyBatis的SQL操作时，事务管理器会负责处理事务的开始、提交和回滚。

#### 定义事务传播行为

`@Transactional`注解还可以定义事务的传播行为、隔离级别、超时时间等参数。例如，指定事务传播行为为REQUIRED：

```java
@Transactional(propagation = Propagation.REQUIRED)
public void updateUser(User user) {
    // 业务逻辑代码
    userDao.updateUser(user);
}
```

##### 捕获特定异常回滚事务

可以通过设置`rollbackFor`属性来指定遇到特定异常时回滚事务。例如，当遇到`SQLException`异常时回滚事务

```java
@Transactional(rollbackFor = SQLException.class)
public void updateUser(User user) {
    // 业务逻辑代码
    userDao.updateUser(user);
}
```

##### 类级别的`@Transactional`注解

除了可以在方法级别上添加`@Transactional`注解外，还可以在类级别上添加，表示该类的所有方法都将受事务管理。、

```java
@Service
@Transactional
public class UserService {
    // 类的方法
}
```

## 事务控制的原理

