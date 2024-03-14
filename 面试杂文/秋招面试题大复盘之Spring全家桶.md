## Spring

- [ ] **谈谈Spring的优点**

  Spring主要有如下特点：1.轻量，它的大小大约只有2MB。2.通过IOC的控制反转和依赖注入实现松耦合。3.支持面向切面编程，并且把业务逻辑和系统服务分开。4.通过切面和模板减少板式代码。5.方便集成各种优秀框架。内部提供了对各种优秀框架的直接支持。6.方便程序测试，Spring支持Junit4。添加注解便可测试Spring程序。

- [ ] **什么是IOC，如何实现的**

  IOC顾名思义就是控制反转，在我们传统的应用程序中，创建对象我们需要去new一个对象，创建程序的主动权在对象自己，对象与对象之间耦合度太高了，但是Spring的IOC容器他帮我们去创建对象，对象的生命周期都掌握在他的手中，这样的话大大的降低了整个系统各个依赖类之间的耦合度。IOC的实现原理是反射和工厂模式，spring框架启动的时候，IOC根据XML文件获取Bean标签中的信息，将类的信息都获取到，然后通过反射获取类的全部信息。这样的话就反转了对象的创建。

  IOC顾名思义就是控制反转，在传统应用程序中，都是程序类自己去创建自己需要的对象，而在Spring程序中，IOC容器控制了对象的创建。管理了对象之间的相互依赖。大大的降低了对象与对象之间的耦合度。IOC的实现是工厂模式和反射，通过xml获取全类名然后通过反射来创建对象，使用工厂模式隔断类与类之间的关系。

- [ ] **IOC容器初始化的过程**

  先从xml中读取配置文件将bean标签解析成BeanDefinition然后再将BeanDefinition注册到容器BeanDefinitionMap中去，BeanFactory根据BeanDefinition的定义信息创建实例化和初始化bean。

- [ ] **什么是AOP？有哪些AOP的概念**

  AOP是面向切面编程，它可以在不改变原代码的情况下对模块之间进行增强。比如说我们想在查询SQL的时候，打印SQL语句，那么AOP就可以做到。这样的话，就大大的增强了代码之间的扩展性。

  AOP就是面向切面编程，他的用处就是在一些模块与模块之间，我们可以在不影响原程序的情况下进行一些增强，比如每次使用sql查询的时候，我们可以通过打印日志来处理错误。这样很利于开发程序之间的扩展性。AOP是基于动态代理的，如果要代理的对象实现了某个接口，那么Spring AOP就会使用JDK的动态代理去创建对象；而对于没有实现接口的对象，就使用CGLib动态代理生成了一个被代理对象的子类来作为代理。AOP包含的几个概念点：连接点，通知，目标对象。织入等。

- [ ] **AOP的应用场景有哪些**

  记录日志。在我们一些操作前后可以打印此操作的过程。

  监控性能。统计方法的运行时间。

  权限控制。调用方法前校验是否有权限。

  事务管理。调用方法前开启事务，调用方法后关闭事务。

- [ ] **有哪些AOP Advice通知的类型**

  前置通知，后置通知，环绕通知，返回后通知，抛出异常后通知。

- [ ] **AOP的实现原理是什么**

  Spring中的AOP是基于动态代理实现的，在我们为Spring中的Bean织入切面，那么这个Bean在创建的时候，就会生成一个代理对象，在我们后续使用这个bean的时候，都是调用的这个代理对象，他里面包含了重写的方法。Spring的AOP基于JDK的动态代理和CGlib的动态代理。

  AOP的是基于动态代理实现的，如果我们为Spring的某个bean配置了切面，那么spring在创建这个bean的时候，就会创建这个bean的代理对象，我们后续对bean中的方法进行调用，其实都调用的是代理对象重写的方法，而spring的AOP使用了两种动态代理，分别为JDK的动态代理以及Glib的动态代理。

- [ ] **spring AOP采用的两种不同的动态代理的区别**

  spring的AOP使用了两种动态代理，分别为JDK的动态代理以及Glib的动态代理。先说说JDK的动态代理，如果目标类实现了接口，AOP会选择使用JDK动态代理目标类。JDK动态代理的代理类根据目标类实现的接口动态生成，不需要自己编写。生成的动态代理类和目标类都实现了相同的接口。再是CGLib动态代理，如果目标类没有实现接口 ,那么Spring AOP会选择使用CGLib来动态代理目标类,他可以在运行时动态生成类的字节码，动态创建目标类的子类对象，在子类对象中进行增强目标类。CGLib是通过继承的方式实现动态代理的。因此如果某个类被final修饰，那么他是无法使用CGLib动态代理的。

- [ ] **Spring AOC 和 AspectJ AOP 有什么区别**

  SpringAOP是运行时增强，而AspectJ是编译时增强，他是基于字节码操作的。AspectJ相对于SpringAOP来说吗更为强大，但是SpringAOP来说更简单。切面多的话可以选择AspectJ。

- [ ] **Spring中bean的作用域有哪些**

  作用域有singleton，单例创建。prototype，每次请求都会创建一个实例bean。request，每一次HTTP请求就会产生一个新的bean，该bean仅在request内有效。session，每一次请求都会产生一个bean，该bean仅在session。

- [ ] **Spring中的单例bean的线程安全问题了解**

  在多个线程操作同时操作同一个对象的时候，对这个对象的非静态成员变量的写操作会存在线程安全问题。通常的解决方式是在类中定义一个ThreadLocal成员变量，将需要的可变成员变量保存在ThreadLocal中。

- [ ] **Spring中bean的生命周期**

  总的来说有四个大方面，实例化，属性赋值，初始化，销毁。从细的方面来划分的话 先是实例化，然后属性赋值，检查Aware相关接口并设置相关依赖，BeanPostProcessor前置处理器，检查InitailizinBean接口的方法afterPropertiesSet，检查xml中的初始化方法，BeanPostProcessor后置处理器，最后销毁。销毁也是有两个配置，一种是DisposableBean接口，另一种是xml的配置。

- [ ] **Spring框架中用到的设计模式**

  工厂模式：Spring使用工厂模式通过BeanFactory和ApplicationContext创建Bean对象。

  代理设计模式：Spring AOP功能实现就是使用的静态代理和动态代理。

  单例设计模式：Spring中的bean默认是单例的。

  模板方法模式：Spring中的jdbcTemplate、hibernateTemplate等以TemPlate结尾的对数据库操作的类。

  观察者模式：Spring事件驱动模型就是观察者模式很经典的一个应用。

  适配器模式：SpringAOP的增强或通知使用到了适配器模式、SpringMvc中也是用到了适配器模式Controller

- [ ] **@Component和@Bean的区别**

  1.作用对象不同。@Component作用于类，@Bean作用于方法。

  2.@Component注解通常是通过类路径来自动侦测以及自动装配到Spring容器中。(当然也可以使用@ComponentScan注解自定义扫描的路径）。@Bean注解通常是标有该注解的方法中定义这个Bean，告诉Spring这是某个类的实例，当我需要他的时候还给我。

- [ ] **将一个类声明为Spring的bean的注解有哪些**

  1. @Component注解。通用的注解，可标注任意类为Spring组件。如果一个Bean不知道属于哪一个层，可以使用@Component注解标注。
  2. Repository注解。对应持久层，即Dao层，主要用于数据库相关操作。
  3. Service注解。对应服务层，即Service层，主要涉及一些复杂的逻辑，需要用到Dao层（注入）。
  4. Controller注解。对应Spring MVC的控制层，即Controller层，主要用于接受用户请求并调用Service层的方法返回数据给前端页面。

- [ ] **Spring事务管理的方式有哪几种**

  编程式事务和声明式事务。编程式事务是在代码中硬编码。声明式事务是指在配置文件中配置，分别基于xml的声明式事务和基于注解的声明式事务。

- [ ] **Spring事务隔离级别有哪几种**

  在TransactionDefinition接口中定义了五个表示隔离级别的常量：

  默认： ISOLATION_DEFAULT：使用后端数据库默认的隔离级别，Mysql默认采用的REPEATABLE_READ隔离级别；Oracle默认采用的READ_COMMITTED隔离级别。

  读未提交：ISOLATION_READ_UNCOMMITTED：最低的隔离级别，允许读取尚未提交的数据变更，可能会导致脏读、幻读或不可重复读。

  读已提交：ISOLATION_READ_COMMITTED：允许读取并发事务已经提交的数据，可以阻止脏读，但是幻读或不可重复读仍有可能发生

  可重复读：ISOLATION_REPEATABLE_READ：对同一字段的多次读取结果都是一致的，除非数据是被本身事务自己所修改，可以阻止脏读和不可重复读，但幻读仍有可能发生。

  串行化： ISOLATION_SERIALIZABLE：最高的隔离级别，完全服从ACID的隔离级别。所有的事务依次逐个执行，这样事务之间就完全不可能产生干扰，也就是说，该级别可以防止脏读、不可重复读以及幻读。但是这将严重影响程序的性能。通常情况下也不会用到该级别

- [ ] **Spring事务中有哪几种事务传播行为**

  在TransactionDefinition接口中定义了7个表示事务传播行为的常量。

  支持当前事务的情况：

  PROPAGATION_REQUIRED：如果当前存在事务，则加入该事务；如果当前没有事务，则创建一个新的事务。

  PROPAGATION_SUPPORTS： 如果当前存在事务，则加入该事务；如果当前没有事务，则以非事务的方式继续运行。

  PROPAGATION_MANDATORY： 如果当前存在事务，则加入该事务；如果当前没有事务，则抛出异常。（mandatory：强制性）。

  不支持当前事务的情况：

  PROPAGATION_REQUIRES_NEW： 创建一个新的事务，如果当前存在事务，则把当前事务挂起。

  PROPAGATION_NOT_SUPPORTED： 以非事务方式运行，如果当前存在事务，则把当前事务挂起。

  PROPAGATION_NEVER： 以非事务方式运行，如果当前存在事务，则抛出异常。

  其他情况：

  PROPAGATION_NESTED： 如果当前存在事务，则创建一个事务作为当前事务的嵌套事务来运行；如果当前没有事务，则该取值等价于PROPAGATION_REQUIRED

- [ ] **BeanFactory和ApplicationContext有什么区别**

  BeanFactory和ApplicationContext是Spring的两大核心接口，都可以当做容器。其中ApplicationContext是BeanFactory的子接口。所以说ApplicationContext肯定比BeanFactory提供的功能更加完整，BeanFactory采用的是延迟加载的形式来注入Bean的，即只有在使用到某国Bean的时候（getBean）才会对该Bean进行实例化。这样我们就不能发现一些存在Spring的配置问题。ApplicationContext是在容器启动的时候，一次性创建了所有的Bean，这样我们就可以在容器启动的时候监测配置错误了。但是这样也会占用很多内存，当应用程序Bean比较多时，程序启动缓慢。

- [ ] **如何定义bean的范围**

  通过bean定义中的scope属性定义。

- [x] **可以通过多少种方式完成依赖的注入**

  有三种注入方式，构造器注入，setter注入，接口注入。

- [ ] **解决多类型冲突的三种方式**

  @Authowired通常是先根据类型byType再根据默认名称来注入的，但是如果有多个类型相同的bean时，就需要@Qualifier来解决，@Qualifier配合@Autowired使用。用于指定通过name匹配bean。也可以使用@Primary注解在Bean类上，指定优先注入的Bean的实例。

## SpringMVC

- [ ] **谈谈自己对SpringMvc的了解**

  MVC是一种设计模式，SpringMvc是对Servelt的一种封装，他帮我们很大程度上减少了web程序开发的繁杂。

- [ ] **SpringMvc的工作原理,工作流程**

  客户端（浏览器）将request请求发给前端控制器（DispatchServlet），前端控制器通过（处理器映射器）HandlerMapping解析请求对应的Handler，处理器映射器会返回一条执行链给前端控制器，前端控制器请求处理适配器执行（HandlerApater），处理器适配器在Handler处理器（也就是我们常说的Controller）中完成相应的业务。然后将结果（ModelAndView)返回给处理器适配器，处理器适配器将ModelAndView返回给前端控制器，前端控制器不能自己去解析此视图，需要ViewResolver根据逻辑view去查找实际的View，ViewResolver解析完成后将视图返回给前端控制器，前端控制器将刚才得到到model给view进行视图渲染。然后得到完成的视图信息后前端控制器返回响应给客户端（浏览器）。

- [ ] **SpringMVC的主要组件**

  前端控制器（DispatcherServlet） 处理器映射器（HandlerMapping） 处理器适配器(HandlerAdapter) 处理器(Handler) 视图解析器（ViewResolve） 视图（View)

- [ ] **SpringMVC的常用注解有哪些**

  @Controller:用于标识此类的实例是一个控制器。

  @RequestMapping 映射Web请求

  @ResponseBody 注解返回数据而不是返回页面

  @RequestBody 注解实现接受http请求的json数据，将json数据转换为java对象。

  @PathVariable：获取URL中路径变量的值

  @RestController @Controller + @ResponseBody

  @ExceptionHandler标识一个方法为全局异常处理方法

- [ ] **SpringMVC的异常处理**

  可以将异常抛给Spring框架，由Spring框架来处理；我们只需要配置简单的异常处理器，在异常z处理器中添加视图页面。

- [ ] **介绍下SpringMVC的拦截器**

- [ ] **SpringMVC的拦截器和Filter过滤器有什么差别**

  功能相同:拦截器和Filter都能实现相应的功能。

  容器不同：拦截器构建在SpringMvc体系中；Filter构建在Servlet容器之上

  使用便利性不同：拦截器提供了三个方法，分别在不同的时机执行，过滤器仅提供了一个方法。

## Mybatis

- [ ] **谈谈你对Mybatis的理解**

  Mybatis是一个数据持久层的框架，它内部封装了通过JDBC访问数据库的操作，支持普通的SQL查询，存储过程和高级映射。他的一大好处就是将SQL语句与程序脱离出来，配置在配置文件中.

- [ ] **ORM是什么**

  ORM是对象关系映射

- [ ] **Mybatis和Hibernate的区别**

- [ ] **Mybatis框架的优缺点以及使用的场合**

- [ ] **Mybatis的工作原理**

  Mybatis应用程序通过SqlSessionFactoryBuilder从mybatis-config.xml配置文件来构建SqlSessionFactory（sqlSessioonFactory是线程安全的）然后SqlSessionFactory实例直接开启一个SqlSession,再通过SqlSession实例获取Mapper对象并运行Mapper映射的SQL语句，完成对数据库的CRUD和事务提交，之后关闭SqlSession。

  说明：SqlSession是单线程对象，因此他是非线程安全的，是持久化操作的独享对象，类似于JDBC中的Connection。底层就封装了Jdbc连接。

- [ ] **Mybatis都有哪些Executor执行器？他们的区别是什么**

  Mybatis有三种基本的Executor执行器，SimpleExecutor（简单执行器）、ReuseExecutor（可重用执行器）、BatchExecutor(批处理执行器)。

  简单执行器：每执行一次update或select，就开启一个Statement对象，用完立刻关闭Statement对象。

  可重用执行器：执行update或select，以sql作为key查找statement对象，存在就使用，不存在就创建。用完后，不关闭Statement对象，而是放置于Map<String,Statement>内，供下一次使用。简而言之，就是重复使用Statement对象。

  批处理执行器：执行过update（没有select，jdbc批处理不支持select）将所有sql都添加到批处理中，等待统一执行，它缓存了多个Statement对象，每个Statement对象都是addBatch()完毕后，等待逐一执行executeBatch()批处理。

- [ ] **Mybatis接口绑定有几种实现方式**

  两种方式，1.通过注解绑定。在接口的方法上加入@Select@update等注解里面包含sql语句就好了2.通过xml文件，xml文件里面写上sql语句，里面的id绑定接口的名称。

- [ ] **Mybatis是如何进行分页的**

  我用过mybatisplus的分页，也使用过pagfehelper分页，在学javaweb的时候，也手撸过分页。现在分页的话，基本写sql进行物理分页。要说mybatis分页原理，我觉得吧是使用了拦截器，在sql执行的使用进行拦截，然后重写拼接上分页拦截。用的是动态代理技术。

- [ ] **插件的基本原理是什么**

  MyBatis插件运行的基本原理基于JDK动态代理+拦截器链实现，

  Interceptor 是[拦截器](https://so.csdn.net/so/search?q=拦截器&spm=1001.2101.3001.7020)，可以拦截 Executor, StatementHandle, ResultSetHandler, ParameterHandler 四个接口

  实际就是利用JDK动态代理，生成对应的代理类实例，通过`InvocationHandler#invoke` 实现拦截逻辑

- [ ] **简述Mybatis插件运行原理**

- [ ] **Mybatis是否支持延迟加载，其原理是什么**

- [ ] **#{}和￥{}的区别是什么**

  \#{}被解析成预编译语句，预编译之后可以直接执行，不需要重新编译sql

  ${}仅仅为一个字符串替换，每次执行sql之前都需要进行编译，存在sql注入问题。

## SpringBoot

- [ ] 简单谈一下你对SpringBoot的认识
- [ ] 为什么使用SpringBoot
- [ ] Spring SpringMvc和 SpringBoot有什么区别
- [ ] SpringBoot自动装配的原理
- [ ] SpringBoot的核心注解是哪些？他主要由哪些注解组成
- [ ] 可以使用Xml配置吗
- [ ] SpringBoot中的监听器是什么
- [ ] 怎么理解Spring Boot Starter; 有哪些常用的
- [ ] spring-boot-starter-parent 有什么作用
- [ ] 为什么需要spring-boot-maven-plugin
- [ ] spring打包成的jar包普通的jar包的区别
- [ ] 如何使用SpringBoot处理异常
- [ ] SpringBoot可以兼容老项目吗

## Redis

- [ ] redis的优缺点
- [ ] redis为什么这么快
- [ ] redis的应用场景
- [ ] redis的线程模型
- [ ] redis的数据类型有哪些
- [ ] redis的事务