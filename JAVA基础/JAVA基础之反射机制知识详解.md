> 反射是java中十分重要的特性，框架的编写都离不开反射，比如主流框架spring，springboot等。

## 了解反射

### 反射能够干什么

JAVA反射机制是在运行状态中，对于任意一个类，都能够知道这个类的所有属性和方法；对于任意一个对象，都能够调用它的任意一个方法和属性；

### 自己对反射的理解

反射在框架中用处特别大，就比如用spirng框架的xml配置bean的时候，spring怎么会知道我们的类并且帮我们创建呢，那是因为spring通过xml配置文件里的写了类的全类名，Java用自己的反射机制将类的全部信息进行获取了。

### Class类

在完全了解Java反射机制之前，我们需要了解一下Class类，以及类加载机制，然后基于此我们如何通过反射获取Class类以及类中的成员变量、方法、构造方法等。

Java.lang.Class表示类的类型，一个Class类对象就代表某个类的字节码对象，获取到这个类的字节码对象然后该类中所有的内容都会被知道，然后就可以对这个类进行操作。

`Class`类是Java反射机制的起源和入口/用于获取与类相关的各种信息/提供了获取类信息的相关方法/Class类继承自`Object`类。

```Java
public final class Class<T> implements java.io.Serializable,
                              GenericDeclaration,
                              Type,
                              AnnotatedElement {
    private static final int ANNOTATION= 0x00002000;
    private static final int ENUM      = 0x00004000;
    private static final int SYNTHETIC = 0x00001000;

    private static native void registerNatives();
    static {
        registerNatives();
    }

    /*
     * Private constructor. Only the Java Virtual Machine creates Class objects.   
     * This constructor is not used and prevents the default constructor being
     * generated.
     */
     //私有构造器，只有JVM才能调用创建Class对象
    private Class(ClassLoader loader) {
        // Initialize final field for classLoader.  The initialization value of non-null
        // prevents future JIT optimizations from assuming this final field is null.
        classLoader = loader;
    }
```

到这我们也就可以得出以下几点信息：

- Class类也是类的一种，与class关键字是不一样的。
- 手动编写的类被编译后会产生一个Class对象，其表示的是创建的类的类型信息，而且这个Class对象保存在同名.class的文件中(字节码文件)
- 每个通过关键字class标识的类，在内存中有且只有一个与之对应的Class对象来描述其类型信息，无论创建多少个实例对象，其依据的都是用一个Class对象。
- Class类只存私有构造函数，因此对应Class对象只能有JVM创建和加载
- Class类的对象作用是运行时提供或获得某个对象的类型信息，这点对于反射技术很重要(关于反射稍后分析)。

### 反射的重要类

| 类                            | 含义                                                         |
| ----------------------------- | ------------------------------------------------------------ |
| java.lang.Class               | 代表整个字节码。代表一个类型，代表整个类                     |
| java.lang.reflect.Method      | 代表字节码中的方法字节码。代表类中的方法。                   |
| java.lang.reflect.Constructor | 代表字节码中的构造方法字节码。代表类中的构造方法。           |
| java.lang.reflect.Field       | 代表字节码中的属性字节码。代表类中的成员变量（静态变量+实例变量）。 |

### 反射的使用

> 我们需要知道反射如何获取Class类对象以及类中的成员变量、方法、构造方法等。

在Java中，Class类与java.lang.reflect类库一起对反射技术进行了全力的支持。在反射包中，我们常用的类主要有Constructor类表示的是Class 对象所表示的类的构造方法，利用它可以在运行时动态创建对象、Field表示Class对象所表示的类的成员变量，通过它可以在运行时动态修改成员变量的属性值(包含private)、Method表示Class对象所表示的类的成员方法，通过它可以动态调用对象的方法(包含private)，下面将对这几个重要类进行分别说明。

#### Class类对象的获取

在类加载的时候，jvm会创建一个class对象

class对象是可以说是反射中最常用的，获取class对象的方式的主要有三种

- 根据类名：类名.class

```Java
Class<Book> bookClass = Book.class;
```

- 根据对象：对象.getClass()

```Java
Class<? extends Book> bookClass = new Book().getClass();
```

- 根据全限定类名：Class.forName(全限定类名)

```Java
Class<?> aClass = Class.forName("enity.Book");
```

#### 通过反射实例化对象

使用获得到的Class类newInstance()方法，该方法调用了无参构造函数，必须保证无参构造的存在才可以。

```Java
  //通过forName获取Class对象 
  Class<?> aClass = Class.forName("enity.Book");
  //使用newInstance()方法来调用无参构造器
  Book book  = (Book) aClass.newInstance();
```

> 如果你只是希望一个类的静态代码块运行，那么只需要Class.forName("enity.Book");就好了。因为我们知道静态代码块是在类第一次加载时执行，而非静态代码块则是在构造函数时执行。

#### Class类的方法

| 方法名             | 说明                                                         |
| ------------------ | ------------------------------------------------------------ |
| forName()          | 1.获取Class对象的一个引用，但引用的类还没有加载(该类的第一个对象没有生成)就加载了这个类。2.为了产生Class引用，forName()立即就进行了初始化。 |
| Object-getClass()  | 获取Class对象的一个引用，返回表示该对象的实际类型的Class引用。 |
| getName()          | 取全限定的类名(包括包名)，即类的完整名字。                   |
| getSimpleName()    | 获取类名(不包括包名)                                         |
| getCanonicalName() | 获取全限定的类名(包括包名)                                   |
| isInterface()      | 判断Class对象是否是表示一个接口                              |
| getInterfaces()    | 返回Class对象数组，表示Class对象所引用的类所实现的所有接口。 |
| getSupercalss()    | 返回Class对象，表示Class对象所引用的类所继承的直接基类。应用该方法可在运行时发现一个对象完整的继承结构。 |
| newInstance()      | 返回一个Oject对象，是实现“虚拟构造器”的一种途径。使用该方法创建的类，必须带有无参的构造器 |
| getFields()        | 获得某个类的所有的公共（public）的字段，包括继承自父类的所有公共字段。 类似的还有getMethods和getConstructors。 |
| getDeclaredFields  | 获得某个类的自己声明的字段，即包括public、private和proteced，默认但是不包括父类声明的任何字段。类似的还有getDeclaredMethods和getDeclaredConstructors。 |

**getName、getCanonicalName与getSimpleName的区别**

- getSimpleName：只获取类名
- getName：类的全限定名，jvm中Class的表示，可以用于动态加载Class对象，例如Class.forName。
- getCanonicalName：返回更容易理解的表示，主要用于输出（toString）或log打印，大多数情况下和getName一样，但是在内部类、数组等类型的表示形式就不同了。

#### Constructor类及其用法

> Constructor类存在于反射包(java.lang.reflect)中，反映的是Class 对象所表示的类的构造方法。

获取Constructor对象是通过Class类中的方法获取的，Class类与Constructor相关的主要方法如下

| 方法返回值       | 方法名称                                           |
| ---------------- | -------------------------------------------------- |
| static Class<?>  | forName(String className)                          |
| Constructor      | getConstructor(Class<?>... parameterTypes)         |
| Constructor<?>[] | getConstructors()                                  |
| Constructor      | getDeclaredConstructor(Class<?>... parameterTypes) |
| Constructor<?>[] | getDeclaredConstructors()                          |
| T                | newInstance(Object... initargs)                    |

**用上述的api举个例子：**

获取Book类中所有的构造函数

```Java
 Class<?> aClass1 = Class.forName("enity.Book");
        Constructor<?>[] constructors = aClass1.getConstructors();
        for (Constructor c: constructors) {
            System.out.println(c);
         }
```

获取Book类中指定为string参数类型的public权限的构造参数

```Java
Class<?> aClass1 = Class.forName("enity.Book");
Constructor<?> constructor = aClass1.getConstructor(String.class, String.class);
Object o = constructor.newInstance("hello", "world");
```

获取Book类中指定为string参数类型的private权限的构造参数

```Java
 Class<?> aClass1 = Class.forName("enity.Book");
 Constructor<?> constructor = aClass1.getDeclaredConstructor(String.class);
 System.out.println(constructor);
```

#### Field类及其用法

> Field 提供有关类或接口的单个字段的信息，以及对它的动态访问权限。反射的字段可能是一个类（静态）字段或实例字段。

同样的道理，我们可以通过Class类的提供的方法来获取代表字段信息的Field对象，Class类与Field对象相关方法如下

| 方法返回值 | 方法名称                      | 方法说明                                                     |
| ---------- | ----------------------------- | ------------------------------------------------------------ |
| Field      | getDeclaredField(String name) | 获取指定name名称的(包含private修饰的)字段，不包括继承的字段  |
| Field[]    | getDeclaredFields()           | 获取Class对象所表示的类或接口的所有(包含private修饰的)字段,不包括继承的字段 |
| Field      | getField(String name)         | 获取指定name名称、具有public修饰的字段，包含继承字段         |
| Field[]    | getFields()                   | 获取修饰符为public的字段，包含继承字段                       |

**用上述的api举个例子**

使用上面的方法获取Book类中的全部private属性并打印出来

```Java
  Class<?> aClass1 = Class.forName("enity.Book");
  Field[] fields = aClass1.getDeclaredFields();
  for (Field field:fields) {
      System.out.println(field);
    }
```

使用上述方法将Book类的全部public属性打印出来

```Java
    Class<?> aClass1 = Class.forName("enity.Book");
        Field[] fields = aClass1.getFields();
        for (Field field:fields) {
            System.out.println(field);
        }
```

#### Method类及其方法

> Method 提供关于类或接口上单独某个方法（以及如何访问该方法）的信息，所反映的方法可能是类方法或实例方法（包括抽象方法）。

下面是Class类获取Method对象相关的方法

| 方法返回值 | 方法名称                                                   | 方法说明                                                     |
| ---------- | ---------------------------------------------------------- | ------------------------------------------------------------ |
| Method     | getDeclaredMethod(String name, Class<?>... parameterTypes) | 返回一个指定参数的Method对象，该对象反映此 Class 对象所表示的类或接口的指定已声明方法。 |
| Method[]   | getDeclaredMethods()                                       | 返回 Method 对象的一个数组，这些对象反映此 Class 对象表示的类或接口声明的所有方法，包括公共、保护、默认（包）访问和私有方法，但不包括继承的方法。 |
| Method     | getMethod(String name, Class<?>... parameterTypes)         | 返回一个 Method 对象，它反映此 Class 对象所表示的类或接口的指定公共成员方法。 |
| Method[]   | getMethods()                                               | 返回一个包含某些 Method 对象的数组，这些对象反映此 Class 对象所表示的类或接口（包括那些由该类或接口声明的以及从超类和超接口继承的那些的类或接口）的公共 member 方法 |

**同样用上述的api举个例子**

获取所有的公共方法，包括继承 超类和实现超接口

```Java
  Class<?> aClass1 = Class.forName("enity.Book");
  Method[] methods = aClass1.getMethods();
  for (Method method:methods)
  {
     System.out.println(method);
   }
```

获取全部的方法但不包括继承的

```Java
 Class<?> aClass1 = Class.forName("enity.Book");
        Method[] methods = aClass1.getDeclaredMethods();
        for (Method method:methods){
            System.out.println(method);
    }
```

### 反射机制执行的流程

首先我们来看看反射获取类实例这一块，forName通过全类名获取获取到类信息之后并没有将实现留给Java，而是交给了Jvm去加载。主要是先获取ClassLoader

这一块知识先暂停，先去好好了解Jvm