> > *AOP作为Spring框架核心组件之一，本文将记录什么是Aop，如何使用AOP以及对AOP(面向切面编程的理解），并深入了解AOP的原理以及底层。*
>
> ## 如何理解AOP
>
> AOP为`Aspect Oriented Programming`的缩写，意为：面向切面编程；他的本质是为了解耦，是一种设计思想。我们在平时使用**面向对象编程**（OOP）有一些弊端，当需要为多个不具有继承关系的对象引人同一个公共行为时，例如日志、安全检测等，我们只有在每个对象里引用公共行为，这样程序中就产生了大量的重复代码，程序就不便于维护了。所以就有了一个对面向对象编程的补充，即面向方面编程 （ AOP ), AOP 所关注的方向是**横向**的，区别于 OOP 的**纵向**。
>
> **简而言之**，AOP就是为了方便在某个方法或者接口的执行前后添加操作，而且是在不影响原方法或接口的前提下。举个明显的栗子，如下：
>
> 如何给如下UserServiceImpl中所有方法添加进入方法的日志，
>
> ```Java
> /**
>  * @author pdai
>  */
> public class UserServiceImpl implements IUserService {
> 
>     /**
>      * find user list.
>      *
>      * @return user list
>      */
>     @Override
>     public List<User> findUserList() {
>         System.out.println("execute method： findUserList");
>         return Collections.singletonList(new User("pdai", 18));
>     }
> 
>     /**
>      * add user
>      */
>     @Override
>     public void addUser() {
>         System.out.println("execute method： addUser");
>         // do something
>     }
> 
> }
> ```
>
> 我们将记录日志功能解耦为日志切面，它的目标是解耦。进而引出AOP的理念：就是将分散在各个业务逻辑代码中相同的代码通过**横向切割**的方式抽取到一个独立的模块中！
>
> OOP面向对象编程，针对业务处理过程的实体及其属性和行为进行抽象封装，以获得更加清晰高效的逻辑单元划分。而AOP则是针对业务处理过程中的切面进行提取，它所面对的是处理过程的某个步骤或阶段，以获得逻辑过程的中各部分之间低耦合的隔离效果。这两种设计思想在目标上有着本质的差异。
>
> ## AOP术语
>
> > 首先让我们从一些重要的AOP概念和术语开始。**这些术语不是Spring特有的**。
>
> **连接点（Jointpoint）**：表示需要在程序中插入横切关注点的扩展点，**连接点可能是类初始化、方法执行、方法调用、字段调用或处理异常等等**，Spring只支持方法执行连接点，在AOP中表示为**在哪里干**；
>
> **切入点（Pointcut）**： 选择一组相关连接点的模式，即可以认为连接点的集合，Spring支持perl5正则表达式和AspectJ切入点模式，Spring默认使用AspectJ语法，在AOP中表示为**在哪里干的集合**；
>
> **通知（Advice）**：在连接点上执行的行为，通知提供了在AOP中需要在切入点所选择的连接点处进行扩展现有行为的手段；包括前置通知（before advice）、后置通知(after advice)、环绕通知（around advice），在Spring中通过代理模式实现AOP，并通过拦截器模式以环绕连接点的拦截器链织入通知；在AOP中表示为**干什么**；
>
> **方面/切面（Aspect）**：横切关注点的模块化，比如上边提到的日志组件。可以认为是通知、引入和切入点的组合；在Spring中可以使用Schema和@AspectJ方式进行组织实现；在AOP中表示为**在哪干和干什么集合**；
>
> **引入（inter-type declaration）**：也称为内部类型声明，为已有的类添加额外新的字段或方法，Spring允许引入新的接口（必须对应一个实现）到所有被代理对象（目标对象）, 在AOP中表示为**干什么（引入什么）**；
>
> **目标对象（Target Object）**：需要被织入横切关注点的对象，即该对象是切入点选择的对象，需要被通知的对象，从而也可称为被通知对象；由于Spring AOP 通过代理模式实现，从而这个对象永远是被代理对象，在AOP中表示为**对谁干**；
>
> **织入（Weaving）**：把切面连接到其它的应用程序类型或者对象上，并创建一个被通知的对象。这些可以在编译时（例如使用AspectJ编译器），类加载时和运行时完成。Spring和其他纯Java AOP框架一样，在运行时完成织入。在AOP中表示为**怎么实现的**；
>
> **AOP代理（AOP Proxy）**：AOP框架使用代理模式创建的对象，从而实现在连接点处插入通知（即应用切面），就是通过代理来对目标对象应用切面。在Spring中，AOP代理可以用JDK动态代理或CGLIB代理实现，而通过拦截器模型应用切面。在AOP中表示为**怎么实现的一种典型方式**
>
> ### 通知类型
>
> **前置通知（Before advice）**：在某连接点之前执行的通知，但这个通知不能阻止连接点之前的执行流程（除非它抛出一个异常）。
>
> **后置通知（After returning advice）**：在某连接点正常完成后执行的通知：例如，一个方法没有抛出任何异常，正常返回。
>
> **异常通知（After throwing advice）**：在方法抛出异常退出时执行的通知。
>
> **最终通知（After (finally) advice）**：当某连接点退出的时候执行的通知（不论是正常返回还是异常退出）。
>
> **环绕通知（Around Advice）**：包围一个连接点的通知，如方法调用。这是最强大的一种通知类型。环绕通知可以在方法调用前后完成自定义的行为。它也会选择是否继续执行连接点或直接返回它自己的返回值或抛出异常来结束执行。
>
> 环绕通知是最常用的通知类型。和AspectJ一样，Spring提供所有类型的通知，我们推荐你使用尽可能简单的通知类型来实现需要的功能。例如，如果你只是需要一个方法的返回值来更新缓存，最好使用后置通知而不是环绕通知，尽管环绕通知也能完成同样的事情。用最合适的通知类型可以使得编程模型变得简单，并且能够避免很多潜在的错误。比如，你不需要在JoinPoint上调用用于环绕通知的proceed()方法，就不会有调用的问题。
>
> **我们把这些术语串联到一起，方便理解**
>
> ![img](https://pdai.tech/_images/spring/springframework/spring-framework-aop-3.png)
>
> ## AOP的工作流程
>
> 1.Spring容器启动
>
> 2.读取所有切面配置中的切入点。
>
> 3.初始化bean，判定bean对应的类中的方法是否匹配到任意切入点。
>
> - 匹配失败，创建对象
> - 匹配成功，创建原始对象（目标对象)的代理对象。
>
> 4.获取bean执行方法。
>
> - 获取bean，调用方法并执行，完成操作。
>
> ## 如何使用AOP
>
> > *Spring AOP 支持对XML模式和基于@AspectJ注解的两种配置方式。*
>
> ### 使用XMl文件配置
>
> 1.先加入相关依赖
>
> ```XML
>   <dependency>
>          <groupId>org.springframework</groupId>
>          <artifactId>spring-context</artifactId>
>          <version>5.2.10.RELEASE</version>
>   </dependency>
>   <dependency>
>         <groupId>org.aspectj</groupId>
>         <artifactId>aspectjweaver</artifactId>
>         <version>1.9.4</version>
>    </dependency>
> ```
>
> 2.创建一个切面类
>
> ```Java
> public class LogAspect {
> 
>     public void begin(){
>         System.out.println("-----------开启事务");
>     }
>     public void commit(){
>         System.out.println("-----------提交事务");
>     }
> }
> ```
>
> 3.配置xml文件
>
> 要引入一下aop的命名空间
>
> ```XML
> xmlns="http://www.springframework.org/schema/beans"
>        xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
>        xmlns:aop="http://www.springframework.org/schema/aop"
>        xsi:schemaLocation="http://www.springframework.org/schema/beans
>        http://www.springframework.org/schema/beans/spring-beans.xsd
>        http://www.springframework.org/schema/aop
>        http://www.springframework.org/schema/aop/spring-aop.xsd">
> ```
>
> 先把该切面类交给spring管理
>
> ```XML
>   <bean id="LogAspect" class="com.shanzhu.util.LogAspect"/>
> ```
>
> 然后配置相关aop文件
>
> ```XML
>   <aop:config>
>         <!--声明切入点-->
>         <aop:pointcut id="book_all" expression="execution(void com.shanzhu.dao.BookDao.getUser())"/>
>         <!--声明LogAspect为切面类-->
>         <aop:aspect ref="LogAspect">
>         <!--通知策略-->
>             <aop:before method="begin" pointcut-ref="book_all"/>
>             <aop:after method="commit" pointcut-ref="book_all"/>
>         </aop:aspect>
>     </aop:config>
> ```
>
> ### 使用注解配置
>
> 基于XML的声明式AspectJ存在一些不足，需要在Spring配置文件配置大量的代码信息，为了解决这个问题，Spring 使用了@AspectJ框架为AOP的实现提供了一套注解。
>
> | 注解名称        | 解释                                                         |
> | --------------- | ------------------------------------------------------------ |
> | @Aspect         | 用来定义一个切面。                                           |
> | @pointcut       | 用于定义切入点表达式。在使用时还需要定义一个包含名字和任意参数的方法签名来表示切入点名称，这个方法签名就是一个返回值为void，且方法体为空的普通方法。 |
> | @Before         | 用于定义前置通知，相当于BeforeAdvice。在使用时，通常需要指定一个value属性值，该属性值用于指定一个切入点表达式(可以是已有的切入点，也可以直接定义切入点表达式)。 |
> | @AfterReturning | 用于定义后置通知，相当于AfterReturningAdvice。在使用时可以指定pointcut / value和returning属性，其中pointcut / value这两个属性的作用一样，都用于指定切入点表达式。 |
> | @Around         | 用于定义环绕通知，相当于MethodInterceptor。在使用时需要指定一个value属性，该属性用于指定该通知被植入的切入点。 |
> | @After-Throwing | 用于定义异常通知来处理程序中未处理的异常，相当于ThrowAdvice。在使用时可指定pointcut / value和throwing属性。其中pointcut/value用于指定切入点表达式，而throwing属性值用于指定-一个形参名来表示Advice方法中可定义与此同名的形参，该形参可用于访问目标方法抛出的异常。 |
> | @After          | 用于定义最终final 通知，不管是否异常，该通知都会执行。使用时需要指定一个value属性，该属性用于指定该通知被植入的切入点。 |
> | @DeclareParents | 用于定义引介通知，相当于IntroductionInterceptor (不要求掌握)。 |
>
> 在之前创建好的切面类上加入注解
>
> ```Java
> @Component
> @Aspect
> public class LogAspect {
>     @Pointcut("execution(void com.shanzhu.dao.BookDao.getUser())")
>     private  void pt(){}
> 
> 
>     @After("pt()")
>     public void begin(){
>         System.out.println("-----------开启事务");
>     }
> 
>     @Before("pt()")
>     public void commit(){
>         System.out.println("-----------提交事务");
>     }
> }
> ```
>
> 这里说一下环绕通知 在对有返回值的方法进行增强时，要对原始方法进行返回值获取，然后返回。
>
> ```Java
>   @Around("pt()")
>     public void beginAndCommit(ProceedingJoinPoint joinPoint) throws Throwable {
>         System.out.println("-----------开启事务");
>         joinPoint.proceed();
>         System.out.println("-----------开启事务");
>     }
>     
>     //有返回值时
>      @Around("pt()")
>     public int beginAndCommit(ProceedingJoinPoint joinPoint) throws Throwable {
>         System.out.println("-----------开启事务");
>         int i=joinPoint.proceed();
>         System.out.println("-----------开启事务");
>         return i;
>     }
> ```
>
> @Around注意事项
>
> 1.环绕通知必须依赖形参ProceedingJoinPoint才能实现对原始方法的调用，进而实现原始方法调用前后同时添加通知；                                                                                                                                                        2.通知中如果未使用ProceedingJoinPoint对原始方法进行调用将跳过原始方法的执行。                                   3.对原始方法的调用可以不接收返回值，通知方法设置成void即可，如果接受返回值，必须设定为object类型。                                                                                                                                                               4.原始方法的返回值如果为void类型，通知方法的返回值类型可以设置成void，也可以设置成Object。              5. 由于无法预知原始方法运行后是否会抛出异常，因此环绕通知方法必须抛出Throwable对象。
>
> 然后在Spring的配置类中加入注解
>
> ```Java
> @EnableAspectJAutoProxy
> @Configuration
> @ComponentScan("com.shanzhu")
> public class SpringConfig {
> 
> }
> ```
>
> 这样就ok了；
>
> ### 切入点（pointcut）的申明规则
>
> Spring AOP 用户可能会经常使用 execution切入点指示符。执行表达式的格式如下：
>
> ```Java
> execution（modifiers-pattern? ret-type-pattern declaring-type-pattern? name-pattern（param-pattern） throws-pattern?）
> ```
>
> `ret-type-pattern` 返回类型模式, name-pattern名字模式和param-pattern参数模式是必选的， 其它部分都是可选的。返回类型模式决定了方法的返回类型必须依次匹配一个连接点。 你会使用的最频繁的返回类型模式是*，它代表了匹配任意的返回类型。
>
> `declaring-type-pattern`, 一个全限定的类型名将只会匹配返回给定类型的方法。
>
> `name-pattern `名字模式匹配的是方法名。 你可以使用*通配符作为所有或者部分命名模式。
>
> `param-pattern` 参数模式稍微有点复杂：()匹配了一个不接受任何参数的方法， 而(..)匹配了一个接受任意数量参数的方法（零或者更多）。 模式()匹配了一个接受一个任何类型的参数的方法。 模式(,String)匹配了一个接受两个参数的方法，第一个可以是任意类型， 第二个则必须是String类型。
>
> 对应到我们上面的例子：
>
> ![img](https://pdai.tech/_images/spring/springframework/spring-framework-aop-7.png)
>
> ```Java
> // 任意公共方法的执行：
> execution（public * *（..））
> 
> // 任何一个名字以“set”开始的方法的执行：
> execution（* set*（..））
> 
> // AccountService接口定义的任意方法的执行：
> execution（* com.xyz.service.AccountService.*（..））
> 
> // 在service包中定义的任意方法的执行：
> execution（* com.xyz.service.*.*（..））
> 
> // 在service包或其子包中定义的任意方法的执行：
> execution（* com.xyz.service..*.*（..））
> 
> // 在service包中的任意连接点（在Spring AOP中只是方法执行）：
> within（com.xyz.service.*）
> 
> // 在service包或其子包中的任意连接点（在Spring AOP中只是方法执行）：
> within（com.xyz.service..*）
> 
> // 实现了AccountService接口的代理对象的任意连接点 （在Spring AOP中只是方法执行）：
> this（com.xyz.service.AccountService）// 'this'在绑定表单中更加常用
> 
> // 实现AccountService接口的目标对象的任意连接点 （在Spring AOP中只是方法执行）：
> target（com.xyz.service.AccountService） // 'target'在绑定表单中更加常用
> 
> // 任何一个只接受一个参数，并且运行时所传入的参数是Serializable 接口的连接点（在Spring AOP中只是方法执行）
> args（java.io.Serializable） // 'args'在绑定表单中更加常用; 请注意在例子中给出的切入点不同于 execution(* *(java.io.Serializable))： args版本只有在动态运行时候传入参数是Serializable时才匹配，而execution版本在方法签名中声明只有一个 Serializable类型的参数时候匹配。
> 
> // 目标对象中有一个 @Transactional 注解的任意连接点 （在Spring AOP中只是方法执行）
> @target（org.springframework.transaction.annotation.Transactional）// '@target'在绑定表单中更加常用
> 
> // 任何一个目标对象声明的类型有一个 @Transactional 注解的连接点 （在Spring AOP中只是方法执行）：
> @within（org.springframework.transaction.annotation.Transactional） // '@within'在绑定表单中更加常用
> 
> // 任何一个执行的方法有一个 @Transactional 注解的连接点 （在Spring AOP中只是方法执行）
> @annotation（org.springframework.transaction.annotation.Transactional） // '@annotation'在绑定表单中更加常用
> 
> // 任何一个只接受一个参数，并且运行时所传入的参数类型具有@Classified 注解的连接点（在Spring AOP中只是方法执行）
> @args（com.xyz.security.Classified） // '@args'在绑定表单中更加常用
> 
> // 任何一个在名为'tradeService'的Spring bean之上的连接点 （在Spring AOP中只是方法执行）
> bean（tradeService）
> 
> // 任何一个在名字匹配通配符表达式'*Service'的Spring bean之上的连接点 （在Spring AOP中只是方法执行）
> bean（*Service）
> ```
>
> 此外Spring 支持如下三个逻辑运算符来组合切入点表达式
>
> ```Java
> &&：要求连接点同时匹配两个切入点表达式
> ||：要求连接点匹配任意个切入点表达式
> !:：要求连接点不匹配指定的切入点表达式
> ```
>
> ### 多种增强通知的顺序
>
> 如果有多个通知想要在同一连接点运行会发生什么？Spring AOP遵循跟AspectJ一样的优先规则来确定通知执行的顺序。 在“进入”连接点的情况下，最高优先级的通知会先执行（所以给定的两个前置通知中，优先级高的那个会先执行）。 在“退出”连接点的情况下，最高优先级的通知会最后执行。（所以给定的两个后置通知中， 优先级高的那个会第二个执行）。
>
> 当定义在不同的切面里的两个通知都需要在一个相同的连接点中运行， 那么除非你指定，否则执行的顺序是未知的。你可以通过指定优先级来控制执行顺序。 在标准的Spring方法中可以在切面类中实现org.springframework.core.Ordered 接口或者用**Order注解**做到这一点。在两个切面中， Ordered.getValue()方法返回值（或者注解值）较低的那个有更高的优先级。
>
> 当定义在相同的切面里的两个通知都需要在一个相同的连接点中运行， 执行的顺序是未知的（因为这里没有方法通过反射javac编译的类来获取声明顺序）。 考虑在每个切面类中按连接点压缩这些通知方法到一个通知方法，或者重构通知的片段到各自的切面类中 - 它能在切面级别进行排序。
>
> ## 在使用AOP开发中的问题小结
>
> 如果要使用Spring aop面向切面编程，调用切入点方法的对象必须通过Spring容器获取 如果一个类中的方法被声明为切入点并且织入了切点之后，通过Spring容器获取该类对象，实则获取到的是一个代理对象 如果一个类中的方法没有被声明为切入点，通过Spring容器获取的就是这个类真实创建的对象
