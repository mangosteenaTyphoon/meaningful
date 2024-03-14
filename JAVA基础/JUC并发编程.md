## 前序知识补充

在学习并发编程之前，我们必须对一下概念了解通透，否则不管是有再多的代码知识，也不能很好的灵活运用。

#### 进程是什么

进程是计算机中的程序关于某数据集合上的一次运行活动，是系统进行**资源分配**和**调度**的**基本单位**，是**操作系统**结构的基础。在早期面向进程设计的计算机结构中，**进程是程序的基本执行实体**；在当代面向线程设计的计算机结构中，**进程是线程的容器**。程序是指令、数据及其组织形式的描述，进程是程序的实体。

其实**通俗的来讲**，**进程可以视为程序的一个实例**，比如打开qq音乐，网易云音乐等，有的程序可以同时运行多个进程，比如浏览器，问道等可以双开的游戏。

#### 线程是什么

线程是属于进程的，是一个基本的 CPU 执行单元，是程序执行流的最小单元。线程是进程中的一个实体，是系统**独立调度的基本单位**，线程本身不拥有系统资源，只拥有一点在运行中必不可少的资源，与同属一个进程的其他线程共享进程所拥有的全部资源。

通俗来说，**线程就好比一个程序中的多个同时执行的任务**，比如你在使用火绒安全管家的时候，你可以一边清理缓存，一边杀毒。

#### 线程和进程的区别和关系

关系：线程是进程的基本执行单元，一个进程的所有任务都在线程中执行；进程要想执行任务，必须得有线程。

区别：同一进程的线程共享本进程的地址空间，而进程之间则是独立的地址空间；同一进程内的线程共享本进程的资源，而进程间的资源是独立的。

#### 并发和并行的区别

**并行**在操作系统中指，一组程序按**独立异步**的速度执行，**无论从微观还是宏观，程序都是一起执行的**。对比地，并发是指:在同一个时间段内，**两个或多个程序执行，有时间上的重叠**(宏观上是同时,**微观上仍是顺序执行**)。

简而言之，并行指在同一时刻，有多个指令在多个 CPU 上同时执行。并发指在同一时刻，有多个指令在单个 CPU 上同时执行。

## 创建线程

Java语言的JVM允许程序运行多个线程，它通过java.lang.Thread类来体现。JDK1.5之前创建新执行线程有两种方法

- 继承Thread类的方法
- 实现Runnable接口

> Thread类的特性

每个线程都是通过某个特定Thread对象的run()方法来完成操作，经常把run()方法的主体称为线程体。

通过该Thread对象的start()方法来启动这个线程，而非直接调用run()

#### 继承Thread

定义子类继承Thread类并重写子类中Thread类中的run方法

```Java
public class Thread_  extends Thread{
    @Override
    public void run() {

        for (int i = 0; i < 100; i++) {

                System.out.println("新开的线程:"+i);

        }
    }
```

创建Thread子类对象，即创建了线程对象。调用线程对象start方法：启动线程，调用run方法。

```Java
   public static void main(String[] args) {
        Thread_ thread_ = new Thread_();

        thread_.start();
        for (int i = 0; i < 100; i++) {
            System.out.println("主线程:"+i);
        }

    }
```

#### 优缺点

优点：编写简单，如果需要访问当前线程直接使用this即可获得当前线程

缺点：线程类已经继承了Thread类，不能再继承其它的父类

#### 实现Runnable接口

定义子类，实现Runnable接口，并实现Ruabble接口中的run()方法

```Java
 class Window implements Runnable{
    private static int count=10;

    @Override
    public void run() {
        while (count>0){
            count--;
            System.out.println(Thread.currentThread().getName()+"卖票了：还有"+count+"张票");
        }
    }
}
```

通过Thread类含参构造器创建线程对象，将Runnable接口的子类对象作为实际参数传递给Thread类的构造器中，调用Thread类中start方法：开启线程，调用Runnable子类接口的run方法。

```Java
    Window window = new Window();
    Thread window1 = new Thread(window,"窗口1");
    window1.start();
```

这样就实现了接口创建多线程的方式

#### 优缺点

优点：线程类只实现了Runnable接口，还可以继承其他的类。可以**实现多个线程共享一个目标对象**，非常适合多个相同线程来处理同一份资源的情况

缺点：编程稍微复杂，需要访问当前线程，必须使用Thread.currentThread()方法

### 注意事项

- 如果自己手动调用run()方法，那么就只是普通方法，没有启动多线程模式。
- run()方法由JVM调用，什么时候调用，执行的过程控制都有操作系统的CPU调度决定。
- 想要启动多线程，必须调用start方法。
- 一个线程对象只能调用一次start()方法启动，如果重复调用了，则将抛出以上的异常“IllegalThreadStateException”。

## 线程的常见方法

#### 设置获取优先级

> **使用Thread类的**`setPriority(int newPriority) `方法改变线程的优先级    `getPriority() `：返回线程优先值

线程的优先等级主要有如下：

- MAX_PRIORITY：10
- MIN _PRIORITY：1
- NORM_PRIORITY：5

注意：低优先级只是获得调度的概率低，并非一定是在高优先级线程之后才被调用

#### 线程休眠

线程休眠指的是，在当前线程的执行中，暂停当前线程指定毫秒数，释放cpu的时间片。

**线程休眠的好处**

没有线程休眠，我们无法控制多个线程的运行顺序

当有线程时，我们可以可控制线程的执行顺序，干涉cpu执行的时间，到达我们想要的目的

**线程休眠的方法**

调用sleep(),sleep需要一个参数用于指定该线程休眠的时间，单位是毫秒，它通常是放在run()方法内的循环被使用

```Java
try{
    Thread.sleep(休眠时间)；
}catch(InterruptedException e)
{
    e.printStackTrace();
}
```

#### 线程让步

线程让步是指让当前线程由“运行状态”进入到“就绪状态”，从而让其它具有相同优先级的等待线程获取执行权；但是，并不能保证在当前线程调用yield()之后，其它具有相同优先级的线程就一定能获得执行权；也有可能是当前线程又进入到“运行状态”继续运行！

线程让步使用Thread的静态方法`yield();`

**yield()与wait()的比较**

> 我们知道wait()的作用是让当前线程由"运行状态"进入"等待(阻塞)状态"的同时，也会释放同步锁，而yield()的作用是让步，它也会让当前线程离开"运行状态"。他们的区别是

- wait是会让线程由运行状态进入到等待(阻塞)状态的，而yield是让线程由运行状态进入到就绪状态
- wait()是会让线程释放它所持有对象的同步锁，而yield()方法不会释放锁

#### 线程的合并

Thread中，join()方法的作用是调用线程等待该线程完成之后，才能继续往下执行。

join是Thread类的一个方法，启动线程后直接调用，即join()的作用是"等待该线程终止",这里需要理解的就是该线程是指主线程等待子线程的终止，就是在子线程调用了join()方法后的代码，只有等到了子线程结束才能执行。

**join()方法的作用**：在很多情况下，主线程生成并启动了子线程，如果子线程要进行大量的耗时运算，主线程往往将于子线程之前结束，但是如果主线程处理完其他的事务之后，需要用到子线程的处理结果，也就是主线程需要等待子线程执行完成之后再结束，这个时候就要用到join()方法了。

## 守护线程

线程分为两类：用户线程（前台线程） 守护线程（后台线程）

如果程序中所有的前台线程都执行完毕，那么后台线程就会自动结束

使用`setDaemmon(true/false)`来设置守护线程

```Java
public class  DeamonThread extends Thread {

    @Override
    public void run() {
        for (int i = 0; i < 100; i++) {
            System.out.println("守护线程打印"+i);

        }
        try {
            Thread.sleep(2000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }
}

class Main{
    public static void main(String[] args) {
        DeamonThread deamonThread = new DeamonThread();
        deamonThread.setDaemon(true);
        deamonThread.start();

        for (int i = 0; i < 100; i++) {
            System.out.println("主线程打印"+i);
        }
    }
}
```

## 线程的生命周期

![img](https://shanzhu-edu.oss-cn-shanghai.aliyuncs.com/image.png)

#### 线程有五种基本状态

- 新建状态(new)

当线程对象被创建之后，即进入了新建状态，如Thread t=new MyThread();

- 就绪状态(Runnable)

当调用线程对象start方法后，线程就会进入就绪状态。处于就绪状态的线程，知识说明此线程已经做好了准备，随时等待cpu调度执行，并不是说执行了start方法后就会立即执行。

- 运行状态

当cpu开始调度处于就绪状态的线程时，此使线程才得以真正的执行，即进入到运行状态。注：就绪状态是进入到运行状态的唯一入口。

- 阻塞状态

处于运行状态中的线程由于某种原因，暂时放弃对cpu的使用权，停止执行，此时进入阻塞状态，直到其进入就绪状态，才有机会再次被cpu调用以进入到运行状态，根据阻塞产生的原因不同，阻塞状态又可以分为三种

1. 等待阻塞：运行状态中的线程执行wait(0）方法,使本线程进入到等待阻塞状态；
2. 同步阻塞：线程在获取synchronized同步锁失败（因为锁被其他线程所占用），它会进入同步阻塞状态；
3. 其他阻塞：通过调用线程的slee()或join或发出了I/O请求时，线程会进入阻塞状态。当sleep()超时，join()等待线程终止超时，或者I/O处理完毕时，线程重新转入就绪状态。

- 死亡状态

线程执行完了或因为异常退出，该线程结束生命周期。

## 线程的安全问题

线程安全问题都是由全局变量及静态变量引起的。若每个线程中对全局变量、静态变量只有读操作，而无写操作，一般来说这个全局变量线程是安全的；若有多个线程同时执行写操作，一般都要考虑线程同步，否则的话就可能影响线程安全。

### 举个经典买票例子

如下代码，多个线程同时操作票的数量

```Java
 class Window implements Runnable{
    private static int count=10;
    Object lock=new Object();
    @Override
    public void run() {
        sellTicket();
    }
    public  void sellTicket(){
        while (true){
            if(count>0)
            {
                System.out.println(Thread.currentThread().getName()+"卖票了：还有"+--count+"张票");
                try {
                    Thread.sleep(1000);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }

            }
         }

        }
    }


public class Ticket{
    public static void main(String[] args) {
        Window window = new Window();
        Thread window1 = new Thread(window,"窗口1");
        Thread window2 = new Thread(window,"窗口2");
        Thread window3 = new Thread(window,"窗口3");
        window1.start();
        window2.start();
        window3.start();

    }
}
```

按照代码逻辑来讲，理想情况下出现的结果应该是每一张票卖一次，卖到0为至，然后退出线程

![img](https://shanzhu-edu.oss-cn-shanghai.aliyuncs.com/image.png)

实际上有一张票被卖多次，还有负数的票出现，这就是线程安全问题。

![img](https://shanzhu-edu.oss-cn-shanghai.aliyuncs.com/image.png)

### 线程安全问题的出现

- 当线程并发访问临界资源时，如果破快原子操作，可能会造成数据不一致。
- 理解资源：共享资源，一次仅容许一个线程操作，才可保持其正确性。
- 原子操作：不可分割的多步操作，被视为一个整体，其顺序和步骤不可打乱或缺省。

### 解决线程问题的方法

对多条操作共享数据的语句，只能让一个线程都执行完，在执行过程中，其他线程不可以参与执行。Java对于多线程的安全问题提供了专业的解决方式：同步机制

#### 同步机制

同步代码块的使用方法

```Java
synchronized (对象){
// 需要被同步的代码；
}
```

同步方法的使用方法

```Java
public synchronized void show (String name){
//需要被同步的代码方法
}
```

#### 同步锁机制

同步机制中的锁在《Thinking in Java》中，是这么说的：对于并发工作，你需要某种方式来防止两个任务访问相同的资源（其实就是共享资源竞争）。 防止这种冲突的方法就是当资源被一个任务使用时，在其上加锁。第一个访问某项资源的任务必须锁定这项资源，使其他任务在其被解锁之前，就无法访问它了，而在其被解锁之时，另一个任务就可以锁定并使用它了

#### **synchronized的锁是什么**

任意对象都可以作为同步锁。所有对象都自动含有单一的锁（监视器）。
 同步方法的锁：静态方法（类名.class）、非静态方法（this）
 同步代码块：自己指定，很多时候也是指定为this或类名.class

#### 注意事项

必须确保使用同一个资源的多个线程共用一把锁，这个非常重要，否则就无法保证共享资源的安全。
 一个线程类中的所有静态方法共用同一把锁（类名.class），所有非静态方法共用同一把锁（this），同步代码块（指定需谨慎）

#### 使用Lock锁

从JDK 5.0开始，Java提供了更强大的线程同步机制——通过显式定义同步锁对象来实现同步。同步锁使用Lock对象充当。

java.util.concurrent.locks.Lock接口是控制多个线程对共享资源进行访问的工具。锁提供了对共享资源的独占访问，每次只能有一个线程对Lock对象加锁，线程开始访问共享资源之前应先获得Lock对象。

ReentrantLock 类实现了 Lock ，它拥有与synchronized 相同的并发性和内存语义，在实现线程安全的控制中，比较常用的是ReentrantLock，可以显式加锁、释放锁。

**使用方法**：

```Java
   lock.lock();
            if(count>=1)
            {
                try {
                     保证线程安全的代码
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }finally {
                    lock.unlock();
                }
              
            }
```

#### synchronized 与 Lock的异同

1. Lock是显式锁（手动开启和关闭锁，别忘记关闭锁），synchronized是隐式锁，出了作用域自动释放。
2. Lock只有代码块锁，synchronized有代码块锁和方法锁。
3. 使用Lock锁，JVM将花费较少的时间来调度线程，性能更好。并且具有更好的扩展性（提供更多的子类）

### 线程死锁

线程死锁指的是不同的线程分别占用对方需要的同步资源不放弃，都在等待对方放弃自己需要的同步资源，就形成了线程的死锁。出现死锁后，不会出现异常，不会出现提示，只是所有的线程都处于阻塞状态，无法继续。

#### 如何解决线程死锁问题

专门的算法、原则  、尽量减少同步资源的定义  、尽量避免嵌套同步

## 线程间通信

[线程间通信](https://so.csdn.net/so/search?q=线程间通信&spm=1001.2101.3001.7020)使线程成为一个整体，提高系统之间的交互性，在提高CPU利用率的同时可以对线程任务进行有效的把控与监督。

先举一个例子：一个顾客去早餐店买早餐，顾客线程先告诉老板需要什么吃的，然后老板线程知道后进行制作，老板线程制作完成后告诉顾客线程进行食用

代码如下：用到了 wait() 以及notify() 方法，也是**等待唤醒机制**的常用方法。

```Java
public class Demo {
    public static void main(String[] args) {
        Object lock = new Object();//协调者的角色

        new Thread(new Runnable() {
            @Override
            public void run() {
                //进入等到
                synchronized (lock) {
                    System.out.println("1.告诉老板要的早餐和数量。。。");
                    try {
                        lock.wait();
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    }
                    System.out.println("3.拿到老板给的早餐..");
                }
            }
        },"顾客线程").start();

        new Thread(new Runnable() {
            @Override
            public void run() {
                try {
                    Thread.sleep(2000);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
                synchronized (lock){
                    System.out.println("2. 老板做好了 交给顾客");
                    lock.notify();
                }
            }
        },"老板线程").start();
    }
}
```

### 等待唤醒机制

#### 什么是等待唤醒机制

这是多个线程间的一种协作机制。谈到线程我们经常想到的是线程间的竞争(race)，比如去争夺锁，但这并不是故事的全部，线程间也会有协作机制。就好比在公司里你和你的同事们，你们可能存在在晋升时的竞争，但更多时候你们更多是一起合作以完成某些任务。

就是在一个线程进行了规定操作后，就进入等待状态(wait)，等待其他线程执行完他们的指定代码过后再将其唤醒(notify0);在有多个线程进行等待时，如果需要，可以使用notifyAll)来唤醒所有的等待线程。|

#### wait() 和 notify() 等待唤醒

等待唤醒机制可以基于 `wait() `和 `notify() `方法来实现，在一个线程内调用该线程锁对象的 wait() 方法， 线程将进入等待队列等待直到被唤醒。wait()/notify() 方法只能在同步方法或同部块中调用。

调用 `wait()` 方法，线程需要获取该对象的对象级别锁，如果没有持锁，将会抛出**IllegalMonitorStateException**，执行 wait() 方法后，当前线程**立即释放锁**

| 方法                                 | 说明                                   |
| ------------------------------------ | -------------------------------------- |
| public final void wait()             | 释放锁 进入等待队列                    |
| public final void wait(long timeout) | 在超过指定时间前，释放锁，进入等待队列 |
| public final void notify()           | 随即唤醒 通知一个线程                  |
| public final void notifyAll()        | 唤醒所有线程                           |

执行 notify() 方法后，当前线程**不会立即释放锁**，需要等到执行 notify() 方法的线程将程序执行完，也就是退出synchronized代码块后，当前线程才会释放锁，此时 wait() 状态所在的线程才可以获取该对象的锁。

#### 注意事项

哪怕只通知了一个等待的线程，被通知线程也不能立即恢复执行，因为它当初中断的地方是在同步块内，而此刻它已经不持有锁，所以她需要再次尝试去获取锁（很可能面临其它线程的竞争)，成功后才能在当初调用wait方法之后的地方恢复执行。

#### 总结

如果能获取锁，线程就从 WAITING状态变成RUNNABLE状态;

#### 调用wait和notify方法需要注意的细节

1. wait方法与notfy方法必须要由同一个锁对象调用。因为:对应的锁对象可以通过notify唤醒使用同一个锁对象调用的wait方法后的线程。
2. wait方法与notify方法必须要在同步代码块或者是同步函数中使用。因为:必须要通过锁对象调用这2个方法。
3. wait方法与notify方法必须要在同步代码块或者是同步函数中使用。因为:必须要通过锁对象调用这2个方法。

## 线程池

### 概述

我们使用线程的时候就去创建一个线程，这样实现起来非常简便，但是就会有一个问题:

如果并发的线程数量很多，并且每个线程都是执行一个时间很短的任务就结束了，这样频繁创建线程就会大大降低系统的效率，因为频繁创建线程和销毁线程需要时间.那么有没有一种办法使得线程可以复用，就是执行完一个任务，并不被销毁，而是可以继续执行其他的任务?

在Java中可以通过线程池来达到这样的效果。

**线程池**:其实就是一个容纳多个线程的容器，其中的线程可以反复使用，省去了频繁创建线程对象的操作，无需反复创建线程而消耗过多资源。合理利用线程池能够带来三个好处:

1. 降低资源消耗。减少了创建和销毁线程的次数，每个工作线程都可以被重复利用，可执行多个任务。
2. 提高响应速度。当任务到达时，任务可以不需要的等到线程创建就能立即执行。
3. 提高线程的可管理性。可以根据系统的承受能力，调整线程池中工作线线程的数目，防止因为消耗过多的内存，而把服务器累趴下(每个线程需要大约1MB内存，线程开的越多，消耗的内存也就越大，最后死机)。

### 线程池的使用

Java里面线程池的顶级接口是`java.util.concurrent .Executor`，但是严格意义上讲Executor并不是一个线程池，而只是一个执行线程的工具。真正的线程池接口是`java.util.concurrent.ExecutorService`。

要配置一个线程池是比较复杂的，尤其是对于线程池的原理不是很清楚的情况下，很有可能配置的线程池不是较优的，因此在`java.util.concurrent .Executors`线程工厂类里面提供了一些静态工厂，生成一些常用的线程池。官方建议使用Executors工程类来创建线程池对象。

Java类库提供了许多静态方法来创建一个线程池:

Executors类中创建线程池的方法如下:

- `newFixedThreadPool`创建一个固定长度的线程池，当到达线程最大数量时，线程池的规模将不再变化
- `newCachedThreadPool`创建一个可缓存的线程池，如果当前线程池的规模超出了处理需求，将回收空的线程;当需求增加时，会增加线程数量;线程池规模无限制。
- `newSingleThreadPoolExecutor `创建一个单线程的Executor，确保任务对了，串行执行
- `newScheduledThreadPool`创建一个固定长度的线程池，而且以延迟或者定时的方式来执行，类似Timer;

#### 使用线程池创建对象的步骤

1. 创建线程池对象

2. 创建Runnable接口子类对象。(task)

3. 提交Runnable接口子类对象。(take task)

   获取到了一个线程池ExecutorService对象，定义了一个使用线程池对象的方法如下;public Future<?> submit(Runnable task):获取线程池中的某一个线程对象，并执行Future接口:用来记录线程任务执行完毕后产生的结果。线程池创建与使用。

4. 关闭线程池。

## 实现Callable接口创建线程

一般情况下，使用`Runnable`接口、`Thread`实现的线程我们都是无法返回结果的。但是如果对一些场合需要线程返回的结果。就要使用用`callble`、`Future`这几个类.`cllable`只能在`Executorservice`的线程池中跑，但有返回结果，也可以通过返回的`Future`对象查询执行状态。`Future`本身也是一种设计模式，它是用来取得异步任务的结果

```Java
public interface callable<v> 
{
  v call( ) throws Exception ;
 }
```

它只有一个call方法，并且有一个返回V，是泛型。可以认为这里返回V就是线程返回的结果。ExecutorService接口:线程池执行调度框架

```Java
<T> Future<T> submit(Callable<T> task);
<T> Future<T> submit( Runnable task, T result);
Future<? > submit(Runnable task);
```

#### 使用Callable接口来计算不同线程下随机数的值

```Java
public class Callable_ implements Callable {
    @Override
    public Object call() throws Exception {

        return new Random().nextInt(100);
    }

    public static void main(String[] args) {
        ExecutorService executorService = Executors.newFixedThreadPool(3);
        Callable_ t1 = new Callable_();
        Callable_ t2 = new Callable_();
        Callable_ t3 = new Callable_();
        Future result1 = executorService.submit(t1);
        Future result2 = executorService.submit(t2);
        Future result3 = executorService.submit(t3);
        try {
            System.out.println(result1.get());
            System.out.println(result2.get());
            System.out.println(result3.get());
        } catch (InterruptedException e) {
            e.printStackTrace();
        } catch (ExecutionException e) {
            e.printStackTrace();
        }

    }
}
```