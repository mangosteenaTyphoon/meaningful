> *bean在我们的spring中就是被IOC容器管理的对象，那么他的生命周期也当然被IOC容器来管理，这篇文章就让我们来探究一下bean的生命周期。*

### Bean生命周期四大阶段

对于Spring Bean的生命周期，从大的方向来说可以分为四个阶段，其中初始化完成后，这个Bean也就可以用了。

- 实例化 Instantiation
- 属性赋值 Populate
- 初始化 Initialization
- 销毁 Destruction

![image-20221227141819390](C:\Users\山竹\AppData\Roaming\Typora\typora-user-images\image-20221227141819390.png)

Bean实例化的时机也分为两种，`BeanFactory`管理的Bean是在**使用到Bean**的时候才会实例化Bean，`ApplicantContext`管理的Bean在**容器初始化**的时候就回完成Bean实例化。

### Bean详细的生命周期

那么Bean的生命周期，在这个过程中他们都做了什么，他们在spring中的具体表现是什么？在这个过程中，会有一些容器级的方法，进行前置和后置的处理，比如`BeanPostProcessor`、`InstantiationAwareBeanPostProcessor`接口方法。这些方法独立于Bean之外，并且会注册到Spring容器中，在Spring容器创建Bean的时候，进行一些处理。

![img](https://secure2.wostatic.cn/static/2TAcWd34FkecRconUHTf32/7H5K%5B%7BV7C5%5B)J47MF5B20)U.png?auth_key=1672121838-n7PHEqvHrckrupQntkF89h-0-7978c39af235be3927c7c33b267875cc)

这就好像，孩子出生之前，需要做一些准备，比如备孕、养胎、备产什么的，出生之后，需要做一些护理。孩子上学前后，也需要做一些学籍的管理。

那么有了各种各样的扩展之后，我们再接着看看Bean的详细的生命周期。首先，我们面临一个问题——**Bean的生命周期从什么时候开始的呢**？

上面写了，Bean实例化前后，可以进行一些处理，但是如果**从Bean实例化前算**开始，那么就要**追溯到容器的初始化**、**`beanDefiinition`****的加载开始**。

所以这篇文章里，我们**取生命周期直接从Bean实例化开始**，但是大家也要知道，Bean实例化前后，可以使用后处理器进行处理，例如`BeanFactoryPostProcessor`、`InstantiationAwareBeanPostProcessor`。

![img](https://secure2.wostatic.cn/static/wpuFNyUEBZWK8AcJKXq3hP/1A_L%25NXMKB7U(@R0ECR%5D5)4.png?auth_key=1672121838-go5Cb5qcT2HsfoWDzbZ4ij-0-73906ee67950bcb9786ff3a45e662dd9)

- **实例化**：第 1 步，实例化一个 Bean 对象
- **属性赋值**：第 2 步，为 Bean 设置相关属性和依赖
- **初始化**：初始化的阶段的步骤比较多，5、6步是真正的初始化，第 3、4 步为在初始化前执行，第 7 步在初始化后执行，初始化完成之后，Bean就可以被使用了
- **销毁**：第 8~10步，第8步其实也可以算到销毁阶段，但不是真正意义上的销毁，而是先在使用前注册了销毁的相关调用接口，为了后面第9、10步真正销毁 Bean 时再执行相应的方法

我们发现中间的一些**扩展过程**也可以分**四类**：

- **Aware接口**：Aware接口的作用是让Bean能拿到容器的一些资源，例如BeanNameAware可以拿到BeanName。就好像上学之前，要取一个学名——不知道多少人上学之前不知道自己大名叫什么，是吧？二毛。
- **后处理器**：在Bean的生命周期里，会有一些后处理器，它们的作用就是进行一些前置和后置的处理，就像上学之前，需要登记学籍，上学之后，会拿到毕业证。
- **生命周期接口**：人可能无法选择如何出生，但也许可以选择如何活着和如何死去，InitializingBean和DisposableBean 接口就是用来定义初始化方法和销毁方法的。
- **配置生命周期方法**：环境影响人，人也在影响环境，成长的时候认真努力，衰亡的时候也可以豁达乐观。可以通过配置文件，自定义初始化和销毁方法。

### PersonBean的一生

话不多说，接下来我们拿一个例子，来看看PersonBean的一生，我们先来看一下它的流程！

![img](https://secure2.wostatic.cn/static/h3RhtgpcYcEuT9qKD2W2TZ/image.png?auth_key=1672121838-qio1c4seSDCBddwiMgukpq-0-58145483a5bd5a164f5cf2393824e5b1)

用文字描述一下这个过程：

Bean容器在配置文件中找到Person Bean的定义，这个可以说是妈妈怀上了。

Bean容器使用Java 反射API创建Bean的实例，孩子出生了。

Person声明了属性no、name，它们会被设置，相当于注册身份证号和姓名。如果属性本身是Bean，则将对其进行解析和设置。

Person类实现了BeanNameAware接口，通过传递Bean的名称来调用setBeanName()方法，相当于起个学名。、

Person类实现了BeanFactoryAware接口，通过传递BeanFactory对象的实例来调用setBeanFactory()方法，就像是选了一个学校。

PersonBean实现了BeanPostProcessor接口，在初始化之前调用用postProcessBeforeInitialization()方法，相当于入学报名。

PersonBean类实现了InitializingBean接口，在设置了配置文件中定义的所有Bean属性后，调用afterPropertiesSet()方法，就像是入学登记。

配置文件中的Bean定义包含init-method属性，该属性的值将解析为Person类中的方法名称，初始化的时候会调用这个方法，成长不是走个流程，还需要自己不断努力。

Bean Factory对象如果附加了Bean 后置处理器，就会调用postProcessAfterInitialization()方法，毕业了，总得拿个证。

Person类实现了DisposableBean接口，则当Application不再需要Bean引用时，将调用destroy()方法，简单说，就是人挂了。

配置文件中的Person Bean定义包含destroy-method属性，所以会调用Person类中的相应方法定义，相当于选好地儿，埋了。

我们来用代码实现一下！

```Java
public class PersonBean implements InitializingBean, BeanFactoryAware, BeanNameAware, DisposableBean {

    private String name;

    private Integer no;

    public PersonBean(){
        System.out.println("1.调用构造方法");
    }



    public Integer getNo() {
        return no;
    }

    public void setNo(Integer no) {
        this.no = no;
        System.out.println("2.设置no属性");
    }
    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
        System.out.println("2.设置name属性");
    }

    @Override
    public void setBeanFactory(BeanFactory beanFactory) throws BeansException {
        System.out.println("3.检查Aware相关接口并设置相关依赖");
    }

    @Override
    public void setBeanName(String s) {
        System.out.println("3.检查Aware相关接口并设置相关依赖");
    }

    @Override
    public void destroy() throws Exception {
        System.out.println("7.这个bean要被销毁了");
    }

    @Override
    public void afterPropertiesSet() throws Exception {
        System.out.println("4.实现InitializingBean接口中的初始化方法");
    }

    public void init(){
        System.out.println("5.实现xml文件中的初始化bean");
    }

    public void destroyMethod(){
        System.out.println("8.实现了xml文件中的销毁方法");
    }

    public void work() {
        System.out.println("6.这个bean正在被使用");
    }
}
```

spring的配置文件

```XML
    <bean id="person" class="enity.PersonBean" init-method="init" destroy-method="destroyMethod">
        <property name="name" value="张三"></property>
        <property name="no"   value="11" ></property>
     </bean>
```

## 总结

Bean的生命周期大致可以分为四个阶段：实例化、属性赋值、初始化、销毁。在这四个片段中有很多扩展。