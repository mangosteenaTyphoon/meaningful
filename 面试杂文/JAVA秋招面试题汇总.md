> 三年磨一剑，秋招要开始咯，复习下面试题。别浪费热爱与天赋。

# 1.JVM vs JDK vs JRE之间的区别

**JDK 是 Java Development Kit 缩写**，JAVA的标准开发包，它提供了编译、运行JAVA程序所需要的各种工具和资源，包括JAVA编译器，JAVA运行时环境，以及JAVA常用的类库等。

**JRE是JAVA运行时环境**，用于运行JAVA的字节码文件。JRE中包括了JVM以及JVM工作所需要的类库。普通用户而只需要安装JRE来运行JAVA程序，而程序开发者必须安装JDK来编译，调试程序。

**JVM是Java 虚拟机（JVM）**，它是运行 Java 字节码的虚拟机。JVM 有针对不同系统的特定实现（Windows，Linux，macOS），目的是使用相同的字节码，它们都会给出相同的结果。字节码和不同系统的 JVM 实现是 Java 语言“一次编译，随处可以运行”的关键所在。

**JVM 并不是只有一种！只要满足 JVM 规范，每个公司、组织或者个人都可以开发自己的专属 JVM。** 也就是说我们平时接触到的 HotSpot VM 仅仅是是 JVM 规范的一种实现而已。

除了我们平时最常用的 HotSpot VM 外，还有 J9 VM、Zing VM、JRockit VM 等 JVM 。维基百科上就有常见 JVM 的对比：[Comparison of Java virtual machinesopen in new window](https://en.wikipedia.org/wiki/Comparison_of_Java_virtual_machines) ，感兴趣的可以去看看。并且，你可以在 [Java SE Specificationsopen in new window](https://docs.oracle.com/javase/specs/index.html) 上找到各个版本的 JDK 对应的 JVM 规范。

#### JAVA文件的运行过程

先是.java文件通过javac编译成字节码文件(.class)，然后通过JVM类加载器首先加载字节码文件，然后通过解释器逐行解释执行成机器语言。

# 2.hashCode()与equals()之间的关系

hashCode()是讲的两个对象的物理地址是否相同、equals()讲的是两个对象是否相同

equals(）是比较引用类型，我们创建的对象是放在堆中的，栈中存放的是对象的引用地址，如果要比较堆中的对象内容是否相同，那么就要重写equals方法了。

先举个栗子：我们直接创建两个字符串

```Java
 String a="32";
 String b="32";
 System.out.println(a==b);
```

毫无疑问，为true，因为我们没有创建对象，是直接创建的字符串，那么他们将在常量池中，==比较的就是他们的内存地址，因为他们内存一样，所以相等。但是如果我们使用对象创建

```Java
 String a=new String("32");
 String b=new String("32");
 System.out.println(a==b);
```

因为创建了对象，那么a和b的地址肯定是不同的，==比较的是内存中的地址，那么肯定是false。

那么如果我们使用equals呢，equals比较的是内容，当然这指的是重写后的。string类已经自己重写了，我们下面测验一下。

```Java
   String a=new String("32");
   String b=new String("32");
   System.out.println(a.equals(b));
```

hasCode是jdk根据对象的地址或者字符数字算出来的int类型的值。是为了获取哈希码,哈希码是为了确认对象在哈希表中的索引位置。

重写equals必须重写hasCode方法。

**我们在**[**重写**](https://so.csdn.net/so/search?q=重写&spm=1001.2101.3001.7020)**equals方法的同时，不对hashcode方法进行重写的话，默认地还是会使用Object类自带的hashcode方法，这样就会出现在某些情况下，明明两个对象的equals方法判断相等了，但是它们的hashcode居然不一样，这是不符合规范的**。

**hashcode经常用于散列数据的快速存取，例如在使用hash类数据集合时，都是先根据存储的对象的hashcode值去判断对象是否相同，因此如果不重写hashcode方法的话，会导致判断对象相同的时候，明明equals方法判断相等了，hashcode却判断不相等，就会造成在不同的位置中可以存放两个相同的对象，这就不合理了。**

# 3.泛型中extends和super的区别

<? extends T> 表示包括T在内的任何T的子类

<? super T> 表示包含T在内的任何T的父类

# 4.equals()与==之间的关系

==：如果是基本数据类型，那么比较的数值，如果是引用类型，比较的是引用地址。

equals：具体看类重写equals方法的比较逻辑，例如String类，虽然是引用类型，但是他们比较的是字符串是否相等。

# 5.重载和重写的区别

重载是指在同一类中，参数类型与个数不同，返回值可不同的同名方法。

重写是指子类在继承父类的方法时，将其方法参数个数，类型，返回值类型以及方法体。**需要注意的是返回值类型必须是父类方法返回值的子类。**

# 6.谈谈List和Set的区别

这一块看集合框架。

# 7.深拷贝和浅拷贝的区别

深拷贝和浅拷贝就是指对象拷贝。一个对象中存在两种类型的属性，一种是基本数据类型，另一种是引用数据类型。浅拷只会拷贝基本数据类型的值和引用数据类型的地址值。深拷贝则是拷贝基本数据类型，并且会针对实例对象的引用地址所指向的对象进行复制，深拷贝出来的对象，内部的属性指向不是同一个对象。

# 8.什么是字节码？采用字节码的好处是什么？

编译器（javac）将java源文件(.java)编译成为字节码文件(.class),可以做到一次编译到处运行，windows上编译好的class文件，可以直接在linux上运行，通过这种方式做到跨平台。

采用字节码的好处，一方面实现了跨平台，另外一方面也提高了代码执行的性能，编译器在编译源代码时可以做出一些编译期的优化。

# 9.Java中的异常体系

Java中的所有异常都来自于顶级父类`throwable`。 `Throwable`下有两个子类，`Exception`和`Error`。Error表示非常严重的错误，例如`java.lang.StackOverFlowError`和`java.lang.OutOfMemoryError`,通常这些错误出现时，仅仅想靠程序自己解决是不行的，因此通常不对此错误进行捕获。Exception表示异常，表示程序出现Exception时，是可以靠程序自己来解决的，比如NullPointerException等，我们可以捕获这些异常来做特殊处理。

Exception的子类通常又可以分为RuntimeException和非RuntimeException两类。RuntimeException表示运行时异常，表示这个异常是在代码运行过程中抛出的，这些异常是非检查异常，程序可以选择处理，这些异常一般是逻辑错误引起的，程序应该从逻辑角度尽可能避免这类异常的发生。比如：NullPointerException,IndexOutBoundsException等。

非运行时错误就是我们常说的检查异常，必须处理的异常，如果不处理，程序不能通过编译。如`SQLException`，等以及用户自定义的Exception。

# 10.在Java的异常处理机制中，什么时候应该抛出异常，什么时候捕获异常？

我们在写一个方法时，需要考虑本方法是否能够处理该异常，如果可以处理那么异常在该方法中被捕获，如果不能则向上抛异常。

# 11.Java中有哪些类加载器

**jdk 自带有三个类加载器：**`BootstrapClassLoader`、`ExtClassLoader`、`AppClassLoader`。

`BootstrapClassLoader `（启动类加载器）是 `ExtClassLoader`的父类加载器（并不是直接的一个继承关系，通过`ExtClassLoader `里面有一个parent属性，这个属性是`BootstrapClassLoader`），负责加载%JAVA_HOME%/lib/文件下的jar 包 和class文件，java核心类库，无法被java程序直接引用。

`ExtClassLoader`（扩展类加载器)`AppClassLoader `的父类加载器，负责加载%JAVA_HOME%/lib/ext文件下的jar包 和class 类。 `AppClassLoader`（默认的系统类加载器、线程上下文加载器） 是自定义加载类的父类，负责加载classpath 下的类文件（程序员自己写的代码以及引入的jar 包）。

# 12.说说类加载器双亲委派模型

jvm中存在的三个默认的类加载器：

- BootstrapClassLoader
- ExtClassLoader
- AppClassLoader

他们之间的关系是，AppClassLoader的父类是ExtClassLoader，ExtClassLoader的父类是BootstrapClassLoader

双亲委派模型说的是JVM在加载一个类时，会调用AppClassLoader的loader方法来加载这个类，不过在这个方法中，会先使用ExtClassLoader当方法来加载这个类，同样ExtClassLoader会先使用BootstrapClassLoader来加载这个类，如果BootstrapClassLoader加载成功，那么就成功了，如果失败，就返回让ExtClassLoader自己去加载，如果他失败了，就会让AppClassLoader去加载。

所以双亲委派指的是，JVM在加载类时，会委派给Ext和Bootstrap去加载，如果没有加载成功，自己再去。

# 13.说说对线程安全的理解

线程安全主要是指同一段代码在多个线程同时执行的情况下，能否得到正确的结果。

# 14.对守护线程的理解

线程分为用户线程和守护线程，用户线程就是普通线程，守护线程就是jvm的后台线程，比如垃圾回收线程就是一个守护线程，守护线程会在其他普通线程都停止运行之后自动关闭，我们可以通过设置thread.setDaemon(true)来把一个线程设置为守护线程。

# 15.ThreadLocal的底层原理

暂无。

# 16.并发，并行，串行之间的区别

并发：两个任务或多个任务看起来是同时执行，但其实是在很短很短的时间内进行切换，站在宏观的角度来说是同时执行，也是并行，但是微观来说，并不是。

并行：两个或多个任务同时执行。

串行：一个任务执行完下一个任务开始执行。

# 17.Java死锁如何避免

暂无。

# 18.谈谈你对IOC的理解

如果让我来说呢，IOC，控制反转，那他控制了什么，反转了什么。在我们学习spring之前开发项目的时候，都是由程序自己去new对象，我们需要什么对象都是程序自己内部去创建，这样将类与类之间的耦合度大大提高，而 使用了spring ioc之后，他将对象创建的方式和属性赋值进行了控制，对象由ioc进行管理，这样的方式不仅降低了类之间的耦合度，而且反转了依赖注入的方式。

# 19.单例模式和单例Bean的区别

我认为的单例Bean就是单例模式出来的产物，但两者不能等同。

# 20.Spring事务传播机制

多个事务方法相互调用的时候，事务如何在这些方法中传播，方法A是一个事务方法，方法A在执行中调用了方法B，那么方法B有无事务以及方法B对事务的要求不同都会对方法A的事务具体执行造成影响，同时方法A的事务对方法B也会造成影响，这种影响具体是什么就由两个方法所定义的事务传播类型所决定。

1. PROPAGATION_REQUIRED，如果当前没有事务，就创建一个事务，如果已经存在事务，就加入到这个事务。当前传播机制也是spring默认传播机制。
2. PROPAGATION_REQUIRES_NEW，新建事务，如果当前存在事务，就抛出异常.
3. PROPAGATION_SUPPORTS,支持当前事务，如果当前没有事务，就以非事务方式执行。
4. PROPAGATION_NOT_SUPPORTED，以非事务方式执行操作，如果当前存在事务，就把当前事务挂起。
5. PROPAGATION_MANDATORY，使用当前的事务，如果当前没有事务，就抛出异常。
6. PROPAGATION_NEVER，以非事务方式执行，如果当前存在事务，则抛出异常。
7. PROPAGATION_NESTED，如果当前存在事务，则在嵌套的事务内执行。如果当前没有事务，则执行与PROPAGATION_REQUIRED类似的操作。

# 21.Spring事务什么时候会失效

spring事务的原理是AOP，进行了切面增强，那么失效的根本原因是这个AOP不起作用了，常见的情况如下：

1. 发生自调用，类里面使用this调用本类的方法，此时这个this对象不是代理类，而是UserService对象本身。
2. 方法不是public 的，@Transactional只能用于public方法上，否则事务不会失效，如果非要用在非public方法上，可以开启AspectJ代理模式。
3. 数据库不支持事务。
4. 没有被spring管理。
5. 异常被吃掉，事务不会回滚。

# 22.Spring中的Bean创建的生命周期有哪些步骤

简单来说，spring中bean的生命周期大概有如下实例化，属性赋值，初始化，销毁。从细的来说，可分为

- 调用构造方法进行实例化
- 属性赋值
- 处理Aware接口
- 初始化
- 销毁

# 23.Spring中的Bean是线程安全的