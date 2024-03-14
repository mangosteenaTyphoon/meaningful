> *初步学习spring时，无法理解spring将创建对象的方法变得如此复杂，在后来学习其他框架的时候，虽然说会编写ssm程序，但是这个坎一个留在自己的心中，当然学了spirng家族其他框架，也更明白spring的重要性。因此翻看大量文档和视频来解决自己对spring的理解，以用此篇来记载。*

# 如何理解IOC

### IOC具体到Spring中怎么理解

**IOC**的全称为`Inversion of Control` ，意为**控制反转**，IoC也被称为**依赖性注入**(DI)，这是一个通过依赖注入对象的过程：对象仅通过构造函数、工厂方法，或者在对象实例化在其上设置的属性来定义其依赖关系(即与它们组合的其他对象)，然后容器在创建bean时注入这些需要的依赖。这个过程从根本上说是Bean本身通过使用直接构建类或诸如服务定位模式的机制，来控制其依赖关系的实例化或位置的逆过程（因此被称为**控制反转**）。

**简单来说**，**我理解的IOC**就是`Spring`用来管理`SpringBean`的一种设计模式，这种设计模式极大的在开发模块上面进行解耦。

#### SpringBean是什么

`Spring`里面的`bean`就类似是定义的一个组件，而这个组件的作用就是实现某个功能的，这里所定义的bean就相当于给了你一个更为简便的方法来调用这个组件去实现你要完成的功能。

#### 如何理解控制反转

首先我们要知道两个问题，**谁控制了谁**?，**哪些方面反转了**？ 我们在编写传统JAVA程序的时候，创建对象往往是在对象内部new出一个需要的对象，是程序主动去创建对象的。而IOC是有专门一个容器来创建这些对象，**IOC控制了对象的创建**。反转的话，很明显是IOC容器来帮我们创建以及注入依赖，对象只是被动的接受依赖对象，**依赖对象的获取方式被反转了**。

## IoC能做什么

> IoC **不是一种技术，只是一种思想**，一个重要的面向对象编程的法则，它能指导我们如何设计出松耦合、更优良的程序。

传统应用程序都是由我们在类内部主动创建依赖对象，从而导致类与类之间高耦合，难于测试；有了IoC容器后，**把创建和查找依赖对象的控制权交给了容器，由容器进行注入组合对象，所以对象与对象之间是松散耦合，这样也方便测试，利于功能复用，更重要的是使得程序的整个体系结构变得非常灵活**。

其实IoC对编程带来的最大改变不是从代码上，而是从思想上，发生了“主从换位”的变化。应用程序原本是老大，要获取什么资源都是主动出击，但是在IoC/DI思想中，应用程序就变成被动的了，被动的等待IoC容器来创建并注入它所需要的资源了。

IoC很好的体现了面向对象设计法则之一—— **好莱坞法则：“别找我们，我们找你”**；即由IoC容器帮对象找相应的依赖对象并注入，而不是由对象主动去找。

## IoC和DI是什么关系

> 控制反转是通过依赖注入实现的，其实它们是同一个概念的不同角度描述。通俗来说就是**IoC是设计思想，DI是实现方式**。

DI—Dependency Injection，即依赖注入：组件之间依赖关系由容器在运行期决定，形象的说，即由容器动态的将某个依赖关系注入到组件之中。依赖注入的目的并非为软件系统带来更多功能，而是为了提升组件重用的频率，并为系统搭建一个灵活、可扩展的平台。通过依赖注入机制，我们只需要通过简单的配置，而无需任何代码就可指定目标需要的资源，完成自身的业务逻辑，而不需要关心具体的资源来自何处，由谁实现。

我们来深入分析一下：

- **谁依赖于谁**？

当然是应用程序依赖于IoC容器；

- **为什么需要依赖**？

应用程序需要IoC容器来提供对象需要的外部资源；

- **谁注入谁**？

很明显是IoC容器注入应用程序某个对象，应用程序依赖的对象；

- **注入了什么**？

就是注入某个对象所需要的外部资源（包括对象、资源、常量数据）。

- **IoC和DI有什么关系呢**？

其实它们是同一个概念的不同角度描述，由于控制反转概念比较含糊（可能只是理解为容器控制对象这一个层面，很难让人想到谁来维护对象关系），所以2004年大师级人物Martin Fowler又给出了一个新的名字：“依赖注入”，相对IoC 而言，“依赖注入”明确描述了“被注入对象依赖IoC容器配置依赖对象”。通俗来说就是**IoC是设计思想，DI是实现方式**。

# IOC的操作Bean管理

> ioc操作Bean主要有两个方面，创建对象和注入属性。

## 基于XML的方式实现

### Bean的实例

#### 构造器实例化

> 构造器实例化是指Spring容器通过Bean对应类中默认的**无参构造方法**来实例化Bean。

创建一个User类

```Java
public class User {

    private String name;

    private Integer age;

    @Override
    public String toString() {
        return "User{" +
                "name='" + name + '\'' +
                ", age=" + age +
                '}';
    }
}
```

对他进行无参构造方法实例化

```Java
  <bean id="user" class="enity.User"></bean>
```

#### 静态工厂方式实例化

> 使用静态工厂是实例化Bean的另一种方式，该方式要求开发者创建一个**静态工厂**的方法来创建Bean的实例，其Bean配置中的class属性所指定的不再是Bean实例的实现类，而是静态工厂类，同时还需要使用factory-method属性来指定所创建的静态工厂方法。下面通过一个案例来演示如何使用静态工厂方式实例化Bean。

创建一个工厂类

```Java
public class BeanFactory {

    public static BeanFactory createBean(){
        return new BeanFactory();
    }
}
```

创建配置文件

```Java
 <bean id="beanFactor" class="enity.BeanFactory" factory-method="createBean"></bean>
```

#### 实例工厂方式实例化

> 还有一种实例化Bean的方式就是采用实例工厂。此种方式的工厂类中，不再使用静态方法创建Bean实例，而是采用直接创建Bean实例的方式。同时，在配置文件中，需要实例化的Bean也不是通过class属性直接指向的实例化类，而是通过factory-bean属性指向配置的实例工厂，然后使用factory-method属性确定使用工厂中的哪个方法。下面通过一个案例来演示实例工厂方式的使用。

创建一个User工厂

```Java
public class UserFactory {

    public UserFactory(){
        System.out.println("user正在实例化。。。");
    }

    public User createUser(){
        return new User();
    }
}
```

然后配置xml文件,实例化工厂

```Java
 <bean id="userFactory" class="enity.UserFactory"></bean>
 <bean id="user" factory-bean="userFactory" factory-method="createUser"></bean>
```

### 依赖注入方式

#### 基于XML的方式注入属性

> XML配置文件的根元素是<beans>，<beans>中包含了多个<bean>子元素，每一个<bean>子元素定义了一个Bean，并描述了该Bean如何被装配到Spring容器中。<bean>元素中同样包含了多个属性以及子元素，其常用属性及子元素如表1所示。

| 属性或子元素名称 | 描述                                                         |
| ---------------- | ------------------------------------------------------------ |
| id               | 是一个Bean的唯一标识符，Spring容器对Bean的配置、管理都通过该属性来完成。 |
| name             | Spring容器同样可以通过此属性对容器中的Bean进行配置和管理，name属性中可以为Bean指定多个名称，每个名称之间用逗号或分号隔开。 |
| class            | 该属性指定了Bean的具体实现类，它必须是一个完整的类名，使用类的全限定名。 |
| scope            | 用来设定Bean实例的作用域，其属性值有：singleton（单例）、prototype（原型）、request、session、global Session、application和websocket。其默认值为singleton。 |
| constructor-arg  | <bean>元素的子元素，可以使用此元素传入构造参数进行实例化。该元素的index属性指定构造参数的序号（从0开始），type属性指定构造参数的类型，参数值可以通过ref属性或value属性直接指定，也可以通过ref或value子元素指定。 |
| property         | <bean>元素的子元素，用于调用Bean实例中的setter方法完成属性赋值，从而完成依赖注入。该元素的name属性指定Bean实例中的相应属性名， ref属性或value属性用于指定参数值。 |
| ref              | <property>、<constructor-arg>等元素的属性或子元素，可以用于指定对Bean工厂中某个Bean实例的引用。 |
| value            | <property>、<constructor-arg>等元素的属性或子元素，可以用来直接指定一个常量值。 |
| list             | 用于封装List或数组类型的依赖注入。                           |
| set              | 用于封装Set类型属性的依赖注入。                              |
| map              | 用于封装Set类型属性的依赖注入。                              |
| entry            | <map>元素的子元素，用于设置一个键值对。其key属性指定字符串类型的键值，ref或value子元素指定其值，也可以通过value-ref或value属性指定其值。 |

在配置文件中，通常一个普通的Bean只需要定义id（或name）和class 两个属性即可.

#### 使用set方式进行注入

创建类，定义属性和对应的 set 方法

```Java
public class User {

    private String name;

    private Integer age;


    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public Integer getAge() {
        return age;
    }

    public void setAge(Integer age) {
        this.age = age;
    }

    @Override
    public String toString() {
        return "User{" +
                "name='" + name + '\'' +
                ", age=" + age +
                '}';
    }
}
```

在 spring 配置文件配置对象创建，配置属性注入

```Java
  <bean id="user" class="enity.User">
        <property name="name" value="张山"></property>
        <property name="age"  value="11"></property>
  </bean>
```

#### 使用 p 名称空间注入

第一步 添加 p 名称空间在配置文件中

```XML
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context"
       加入p名称空间
       xmlns:p="http://www.springframework.org/schema/p"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
       http://www.springframework.org/schema/beans/spring-beans.xsd
       http://www.springframework.org/schema/context
       http://www.springframework.org/schema/context/spring-context.xsd">
</beans>
```

第二步 进行属性注入，在 bean 标签里面进行操作

```XML
  <bean id="user" class="enity.User" p:name="张三" p:age="11"/>
```

#### **有参构造注入**

先创建User类,并且创建相关属性的构造方法

```Java
public class User {

    private String name;

    private Integer age;
    public User(){

    }

    public User(String name, Integer age) {
        this.name = name;
        this.age = age;
    }

    @Override
    public String toString() {
        return "User{" +
                "name='" + name + '\'' +
                ", age=" + age +
                '}';
    }
}
```

在 spring 配置文件中进行配置

```Java
   <bean id="user" class="enity.User">
        <constructor-arg name="name" value="张三"/>
        <constructor-arg name="age" value="11"/>
   </bean>
```

#### xml 注入其他类型属性

- 设置属性为null 值

```XML
     <property name="address">
            <null/>
        </property>
```

- 属性包含特殊符号

```XML
<!--属性值包含特殊符号
1 把<>进行转义 &lt; &gt;
2 把带特殊符号内容写到 CDATA
-->
<property name="address">
  <value><![CDATA[<<南京>>]]></value>
</property>
```

#### 注入属性-外部bean

我们在实际开发中，会需要将一些`bean`注入到 一个`bean`中去，**也就是一个类中需要另一个类的方法**，此时就需要外部bean注入 ，例如service 层调用dao层就是引入外部bean

先创建BookDao接口以及实现类

```Java
public interface BookDao {

    void getUser();
}
public class BookDaoImpl implements BookDao {

    @Override
    public void getUser() {
        System.out.println("获取学生信息");
    }


}
```

再创建BookService接口以及实现类

```Java
public interface BookService {
    void getUser();

    void setBookDao(BookDao bookDao);
}
public class BookServiceImpl implements BookService {
    private BookDao bookDao;

    @Override
    public void getUser() {
       bookDao.getUser();
    }

    public void setBookDao(BookDao bookDao){
        this.bookDao=bookDao;
    }
}
```

在配置文件中注入

```XML
   <bean id="bookDao" class="dao.impl.BookDaoImpl"></bean>

    <bean id="bookService" class="service.impl.BookServiceImpl">
        <property name="bookDao" ref="bookDao"></property>
    </bean>
```

#### 注入属性-内部bean

一对多关系：部门和员工  一个部门有多个员工，一个员工属于一个部门  部门是一，员工是多

创建部门类

```Java
//部门类
public class Dept {
    private String dname;
    public void setDname(String dname) {
        this.dname = dname;
    }
 
    @Override
    public String toString() {
        return "Dept{" +
                "dname='" + dname + '\'' +
                '}';
    }
}
```

创建员工类

```Java
//员工类
public class Emp {
    private String ename;
    private String gender;
    //员工属于某一个部门，使用对象形式表示
    private Dept dept;
    public void setDept(Dept dept) {
        this.dept = dept;
    }
    public void setEname(String ename) {
        this.ename = ename;
    }
    public void setGender(String gender) {
        this.gender = gender;
 
 
    }
 
    public void add(){
        System.out.println(ename+":"+gender+":"+dept);
    }
}
```

在 spring 配置文件中进行配置

```XML
    <!--内部 bean-->
    <bean id="emp" class="bean.Emp">
        <!--设置两个普通属性-->
        <property name="ename" value="lucy"/>
        <property name="gender" value="女"/>
        <!--设置对象类型属性，也是级联赋值的一种写法-->
        <property name="dept">
            <bean id="dept" class="bean.Dept">
                <property name="dname" value="安保部"/>
            </bean>
        </property>
    </bean>
```

#### xml注入集合属性

创建类，定义数组、list、map、set 类型属性，生成对应 set 方法

```Java
public class Stu {
    //1 数组类型属性
    private String[] courses;
    //2 list 集合类型属性
    private List<String> list;
    //3 map 集合类型属性
    private Map<String,String> maps;
    //4 set 集合类型属性
    private Set<String> sets;
 
    public void setSets(Set<String> sets) {
        this.sets = sets;
    }
    public void setCourses(String[] courses) {
        this.courses = courses;
    }
    public void setList(List<String> list) {
        this.list = list;
    }
    public void setMaps(Map<String, String> maps) {
        this.maps = maps;
    }
 
    @Override
    public String toString() {
        return "Stu{" +
                "courses=" + Arrays.toString(courses) +
                ", list=" + list +
                ", maps=" + maps +
                ", sets=" + sets +
                '}';
    }
}
```

在 spring 配置文件进行配置

```XML
    <!--1 集合类型属性注入-->
    <bean id="stu" class="pojo.Stu">
        <!--数组类型属性注入-->
        <property name="courses">
            <array>
                <value>java 课程</value>
                <value>数据库课程</value>
            </array>
        </property>
        <!--list 类型属性注入-->
        <property name="list">
            <list>
                <value>张三</value>
                <value>小三</value>
            </list>
        </property>
        <!--map 类型属性注入-->
        <property name="maps">
            <map>
                <entry key="JAVA" value="java"/>
                <entry key="PHP" value="php"/>
            </map>
        </property>
        <!--set 类型属性注入-->
        <property name="sets">
            <set>
                <value>MySQL</value>
                <value>Redis</value>
            </set>
        </property>
    </bean>
```

#### **集合里面为对象类型值**

先创建一个User类

```Java
public class User {

    private String name;

    private Integer age;
    public User(){

    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public Integer getAge() {
        return age;
    }

    public void setAge(Integer age) {
        this.age = age;
    }

    public User(String name, Integer age) {
        this.name = name;
        this.age = age;
    }

    @Override
    public String toString() {
        return "User{" +
                "name='" + name + '\'' +
                ", age=" + age +
                '}';
    }
}
```

再创建一个Stu类

```Java
public class Stu {

    private List<User> userList;

    public List<User> getUserList() {
        return userList;
    }

    public void setUserList(List<User> userList) {
        this.userList = userList;
    }
}
```

在spring配置文件中进行配置

```XML
 <bean id="user2" class="enity.User">
        <property name="name" value="张武"/>
        <property name="age" value="234"/>
    </bean>

    <bean id="Stu" class="enity.Stu">
        <property  name="userList">
            <list>
                <ref bean="user1"></ref>
                <ref bean="user2"></ref>
            </list>
        </property>
    </bean>
```

#### 自动装配

根据指定装配规则（属性名称或者属性类型），Spring 自动将匹配的属性值进行注入

根据属性名称自动注入

```Java
public class Emp {
    private Dept dept;

    public void setDept(Dept dept) {
        this.dept = dept;
    }

    @Override
    public String toString() {
        return "Emp{" +
                "dept=" + dept +
                '}';
    }
    public void test(){
        System.out.println(dept);
    }
}
public class Dept {
    @Override
    public String toString() {
        return "Dept{}";
    }
}
```

根据属性名称自动注入

```Java
<!--实现自动装配
    bean标签属性autowire,配置自动装配
    autowire 属性常用两个值：
        byName 根据属性名称注入，注入值bean的id值和类型性名称一样
        byType 根据属性类型注入
-->
<bean id="emp" class="com.oy.online.Spring.autowire.Emp" autowire="byName"></bean>

<bean id="dept" class="com.oy.online.Spring.autowire.Dept"></bean>
```

根据属性类型自动注入

```Java
<!--实现自动装配
    bean标签属性autowire,配置自动装配
    autowire 属性常用两个值：
        byName 根据属性名称注入，注入值bean的id值和类型性名称一样
        byType 根据属性类型注入
-->
<bean id="emp" class="com.oy.online.Spring.autowire.Emp" autowire="byType"></bean>

<bean id="dept" class="com.oy.online.Spring.autowire.Dept"></bean>
```

## 基于注解方式实现

> 通过在类上加注解的方式，来声明一个类交给Spring管理，Spring会自动扫描带有@Component，@Controller，@Service，@Repository这四个注解的类，然后帮我们创建并管理，前提是需要先配置Spring的注解扫描器。

- **优点**：开发便捷，通俗易懂，方便维护。
- **缺点**：具有局限性，对于一些第三方资源，无法添加注解。只能采用XML或JavaConfig的方式配置

### Spring 针对 Bean 管理中创建对象提供注解

- `@Component`：仅仅表示spring容器的一种普通组件（Bean） ，并且可以作用在任何层次。
- `@Service`：通常作用在业务层（Service层），用于将业务层的类标识为Spring中的Bean，其功能与@Component 相同。
- `@Controller`：通常作用在控制层（如Spring MVC的Controller），用于将控制层的类标识为Spring中的Bean，其功能与@Component 相同。
- `@Repository`：用于将数据访问层（DAO层）的类标识为Spring中的Bean，其功能与@Component 相同。

**上面四个注解功能是一样的，都可以用来创建 bean 实例**

### Bean的实例化

第一步 使用xml的方式开启组件扫描需要在`spring`的配置文件先导入`context`名称空间

```XML
xmlns:context="http://www.springframework.org/schema/context"
xsi:schemaLocation="http://www.springframework.org/schema/context
    http://www.springframework.org/schema/context/spring-context.xsd">
```

第二步 通过`context:component-scan`标签来开启组件扫描

- 通过设置`base-package`属性的值类指定扫描的包。
- 通过设置`use-default-filters`属性的值（值为false或true）来确定是否使用默认过滤器。
- 设置`<context:include-filter>`来规定扫描哪些类。
- 设置`<context:exclude-filter>`配置不扫描哪些类。
- 这个两个标签的type属性表示过滤规则，`expression`表过滤的类。跟使用@Filter注解配置type和classes属性一样。
- 如果扫描多个包，多个包使用逗号隔开
- 如果扫描多个包，也可以扫描包上层目录

```XML
<context:component-scan base-package="service"/>
```

第三步 创建类，在类上面添加创建对象的注解 这里的value 指定bean的名称，默认是首字母小写的类名

```Java
@Service(value="bookserviceImpl")
public class BookServiceImpl implements BookService {
    private BookDao bookDao;

    @Override
    public void getUser() {
       bookDao.getUser();
    }

    public void setBookDao(BookDao bookDao){
        this.bookDao=bookDao;
    }
}
```

这样就实例化了对象。

### 依赖注入方式

`@Autowired`: 根据属性类型进行**自动装配**

第一步：在BookDao接口的实现类上加注解`@Repository`

```Java
@Repository
public class BookDaoImpl implements BookDao {

    @Override
    public void getUser() {
        System.out.println("获取学生信息");
    }


}
```

第二步：在BookService接口的实现类上加上注解，并对其中需要注入的属性加上自动装配(bookDao)

```Java
@Service
public class BookServiceImpl implements BookService {

    @Autowired
    private BookDao bookDao;

    @Override
    public void getUser() {
       bookDao.getUser();
    }
    
}
```

`@Qualifier`：根据名称进行注入

@Qualifier 注解的使用 ，和上面@Autowired 一起使用

```Java
@Autowired //根据类型进行注入
@Qualifier(value = "userDaoImpl1") //根据名称进行注入
private UserDao userDao;
```

`@Resource`: 可以根据类型注入，可以根据名称注入

```Java
@Resource(name = "userDaoImpl1") //根据名称进行注入
private UserDao userDao;
```

`@Value`：注入普通类型的属性

```Java
@Value(value = "abc")
private String name;
```

#### 完全直接开发

创建配置类，替代 xml 配置文件

```Java
@Configuration // 作为配置类，替代xml 配置文件
@ComponentScan(basePackages = {"com.shanzhu.online.Spring"})
public class SpringConfig {
}
```

# Bean的生命周期

### bean的生命周期过程

- **初始化容器**

  创建对象（内存分配）

  执行构造方法

  执行属性注入（set操作）

  执行bean初始化方法

- **使用bean**

  执行业务操作

- **关闭/销毁容器**

  执行bean销毁方法

具体的可以看啊可能这篇文章：Spring框架核心之Bean的生命周期