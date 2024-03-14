---
title: Java8新特性之Stream流的使用方法
date: 2023-10-27
tags: 
  - Stream流
categories: 
  - JAVA基础	
---

Java8新特性 **stream**流是我们在日常开放中是经常用到的，他可以帮我们很方便的去快捷操作集合对象等数据。

<!-- more -->

## Stream流

Stream也叫Stream流，它是一种流式编程，按照我的个人理解，它是将数据集合的处理类比为在流水线的上等待被作用的物品，每一件物品（数据）都在每一道工序下进行加工，最后返回给终端。

## Stream流有什么用

顾名思义，它可以根据我们的操作需求帮助我们方便快捷的处理集合数据变成我们想要的样子，比如现在我们需要将一个List集合进行去重并排序打印。

```java
List<Integer> myList = Arrays.asList(1, 432, 23, 12, 32, 23, 23, 100);

myList.stream() // 创建流
      .distinct()
      .sorted()
      .forEach(System.out::println);
```

这样我们就可以快速的对集合进行操作了，如果使用往常的foreach操作，那必然是需要很大的周折的。

## 如何使用Stream流

在学习使用Stream流之前我们需要知道一些基础的关键词，这些词可以帮助我们很好的去认识Steam流。

- 中间操作：指在处理数据流的**过程中**，对数据流进行的一系列转换和处理操作。这些操作不会改变数据源，而是产生一个新的数据流。常见的中间操作有：过滤、映射、排序、匹配等。
- 终端操作：终端操作是对数据流进行**最终处理**的操作，它会将中间操作得到的数据流进行计算并返回结果。常见的终端操作有：forEach、collect、reduce、min、max等。
- 构造操作：构造操作是用于**创建数据流**的操作，它可以从不同的数据源中获取数据并转换为Stream流。常见的构造操作有：of、empty、iterate、generate等。

### 创建Stream流

在 Java 8 中，创建流的方式有以下几种：

1. 使用 `Stream.of()` 方法创建一个包含元素的流。例如：

   ```java
   Stream<String> stream = Stream.of("A", "B", "C");
   ```

2. 使用 `Arrays.stream()` 方法将数组转换为流。例如：

   ```java
   String[] array = {"A", "B", "C"};
   Stream<String> stream = Arrays.stream(array);
   ```

3. 使用 `Collection.stream()` 方法将集合转换为流。例如：

   ```java
   List<String> list = Arrays.asList("A", "B", "C");
   Stream<String> stream = list.stream();
   ```

4. 使用 `StreamSupport.stream()` 方法从支持的迭代器中创建流。例如：

   ```java
   Iterator<String> iterator = Arrays.asList("A", "B", "C").iterator();
   Stream<String> stream = StreamSupport.stream(Spliterators.spliteratorUnknownSize(iterator, Spliterator.ORDERED), false);
   ```

5. 使用 `IntStream.range()`、`LongStream.range()` 等方法创建整数或长整数范围的流。例如：

   ```java
   IntStream intStream = IntStream.range(0, 10);
   LongStream longStream = LongStream.range(0, 10);
   ```

6. 使用 `Stream.generate()` 方法生成无限流。例如：

   ```java
   Stream<Integer> infiniteStream = Stream.generate(() -> (int) (Math.random() * 10));
   ```


### 使用Stream流

使用Stream流就是我们利用中间操作和终端操作对数据源进行一些列的操作，操作的常用API如下：

#### 中间操作

中间操作是不会改变当前流，只会产生一个新的流，常用的中间操作API如下：

1. filter(Predicate predicate)：过滤出大于2的元素

   ```java
   Stream<Integer> stream = Stream.of(1, 2, 3, 4, 5);
           stream = stream.filter(n -> n > 2); // 筛选出大于2的元素
           stream.forEach(System.out::println);
   ```

2. map(Function mapper)：将整数列表中的每个元素乘以2

   ```java
   Stream<Integer> stream = Stream.of(1, 2, 3, 4, 5);
   stream.map(n -> n * 2).forEach(System.out::println); // 将每个元素乘以2
   ```

3. flatMap(Function> mapper)：将字符串列表中的每个单词拆分成字符流

   ```Java
   List<String> words = Arrays.asList("hello", "world");
   Stream<Character> charStream = words.stream()
           .flatMap(word -> word.chars().mapToObj(c -> (char) c)); // 将每个单词拆分成字符流
   List<Character> chars = charStream.collect(Collectors.toList()); // 转换为列表
   System.out.println(chars); // 输出结果为 [h, e, l, l, o, w, o, r, l, d
   ```

4. distinct()：去除整数列表中的重复元素

   ```java
   List<Integer> nums = Arrays.asList(1, 2, 2, 3, 4, 4, 5);
   Stream<Integer> stream = nums.stream();
   stream = stream.distinct(); // 去除重复元素
   List<Integer> result = stream.collect(Collectors.toList()); // 转换为列表
   System.out.println(result); // 输出结果为 [1, 2, 3, 4, 5]
   ```

5. sorted()：对整数列表进行自然排序

   ```java
   List<Integer> nums = Arrays.asList(5, 3, 1, 4, 2);
   Stream<Integer> stream = nums.stream();
   stream = stream.sorted(); // 自然排序
   List<Integer> result = stream.collect(Collectors.toList()); // 转换为列表
   System.out.println(result); // 输出结果为 [1, 2, 3, 4, 5]
   ```

6. sorted(Comparator comparator)：根据提供的比较器对整数列表进行排序,

   ```java
   List<Integer> nums = Arrays.asList(5, 3, 1, 4, 2);
   Stream<Integer> stream = nums.stream();
   stream = stream.sorted((a, b) -> b - a); // 根据比较器排序
   List<Integer> result = stream.collect(Collectors.toList()); // 转换为列表
   System.out.println(result); // 输出结果为 [5, 4, 3, 2, 1]
   ```

   如果是对对象列表进行排序的话，需要重写对象列表的hash函数。 举个例子，假设有一个Person类，包含name和age属性，我们想要根据age属性对Person对象集合进行排序：

   ```java
   import java.util.*;
   import java.util.stream.*;
   
   class Person {
       String name;
       int age;
   
       Person(String name, int age) {
           this.name = name;
           this.age = age;
       }
   
       @Override
       public int hashCode() {
           return Objects.hash(name, age);
       }
   
       @Override
       public boolean equals(Object obj) {
           if (this == obj) {
               return true;
           }
           if (obj == null || getClass() != obj.getClass()) {
               return false;
           }
           Person person = (Person) obj;
           return age == person.age && Objects.equals(name, person.name);
       }
   }
   
   public class Main {
       public static void main(String[] args) {
           List<Person> people = Arrays.asList(
               new Person("张三", 30),
               new Person("李四", 25),
               new Person("王五", 35)
           );
   
           List<Person> sortedPeople = people.stream()
               .sorted(Comparator.comparingInt(person -> person.age))
               .collect(Collectors.toList());
   
           sortedPeople.forEach(person -> System.out.println(person.name + ": " + person.age));
       }
   }
   
   ```

7. peek(Consumer action)：对流中的每个元素执行指定的操作，但不改变流中的元素。

   ```java
   List<Integer> numbers = Arrays.asList(1, 2, 3, 4, 5);
   
           // 使用 peek() 方法打印每个元素
           numbers.stream()
               .peek(number -> System.out.println("当前元素： " + number))
               .forEach(System.out::println);
   ```

8. limit(long maxSize)：截取流中的前n个元素

   ```java
   IntStream numbers = IntStream.rangeClosed(1, 10);
   numbers.limit(5).forEach(System.out::println);
   ```

9. skip(long n)：跳过流中的前n个元素

   ~~~java
   IntStream numbers = IntStream.rangeClosed(1, 10);
   numbers.skip(3).forEach(System.out::println);
   ~~~

10. forEach(Consumer action)：对流中的每个元素执行指定的操作

    ~~~java
    IntStream numbers = IntStream.rangeClosed(1, 5);
    numbers.forEach(number -> System.out.println("数字：" + number));
    ~~~

上面就是Stream流中常用的一些中间操作。

#### 终端操作

1. forEach(Consumer<? super T> action)：对Stream流中的每一个元素执行给定的操作:

   ```java
   List<String> names = Arrays.asList("Alice", "Bob", "Charlie");
   names.stream().forEach(name -> System.out.println(name));
   ```

2. toArray：将流转换为数组:

   ```java
   List<Integer> numbers = Arrays.asList(1, 2, 3, 4, 5);
   int[] result = numbers.stream().mapToInt(Integer::intValue).toArray();
   ```

3. reduce：将流中的元素组合起来，得到一个值:

   ```java
   List<Integer> numbers = Arrays.asList(1, 2, 3, 4, 5);
   int sum = numbers.stream().reduce(0, Integer::sum);
   ```

4. min/max：找出流中的最小值或最大值:

   ```java
   List<Integer> numbers = Arrays.asList(1, 2, 3, 4, 5);
   Optional<Integer> max = numbers.stream().max(Integer::compare);
   ```

5. count：计算流中元素的个数:

   ```Java
   List<String> names = Arrays.asList("张三", "李四", "王五");
   long count = names.stream().count();
   ```

6. anyMatch/allMatch/noneMatch：判断流中的元素是否满足给定的条件:

   ```java
   List<Integer> numbers = Arrays.asList(1, 2, 3, 4, 5);
   boolean allPositive = numbers.stream().allMatch(number -> number > 0);
   ```

## 使用Stream流处理一些集合

给定一个整数列表，使用Stream流找出列表中的最大值。

```java
List<Integer> list = Arrays.asList(1,23,21,32,13,213,213,21,3214,32);
Optional<Integer> max = list.stream().max(Integer::compareTo);
System.out.println(max.get());
```

给定一个字符串列表，使用Stream流将列表中的所有字符串连接成一个大字符串。

```java
List<String> list1 = Arrays.asList("a", "b", "c", "d", "e", "c");
String sum = list1.stream().reduce("",String::concat);
System.out.println(sum);
```

给定一个学生列表，每个学生对象包含姓名和年龄属性，使用Stream流筛选出年龄大于18岁的学生

```java
Person person = new Person(); person.setName("John"); person.setAge(18);
Person person1 = new Person();  person1.setName("zd1"); person1.setAge(12);
ArrayList<Person> people = new ArrayList<>();
people.add(person1);  people.add(person);
List<Person> collect = people.stream().filter(o -> o.getAge() >= 18).collect(Collectors.toList());
collect.forEach(System.out::println);
```

给定一个整数列表，使用Stream流计算列表中所有整数的平均值

```Java
List<Integer> list2 = Arrays.asList(1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11);
OptionalDouble average = list2.stream().mapToDouble(Integer::intValue).average();
System.out.println(average.getAsDouble());
```

给定一个字符串列表，使用Stream流统计列表中有多少个字符串的长度大于等于5

```Java
List<String> list3 = Arrays.asList("worker", "factory", "devices", "ping");
long count = list3.stream().filter(o -> o.length() >=5).count();
System.out.println(count);
```

## Stream流的种类

从执行方式上来看，Stream又可以分为顺序流和并行流。顺序流由主线程按顺序对流执行操作，也就是我们常说的普通流；而并行流则以多线程的方式对流进行操作，内部实现并行处理，但前提是流中的数据处理没有顺序要求。这种并行处理方式能够大大提高程序的执行效率。

#### 举个例子

并行流和串行流在处理数据时的效率和使用方式存在明显差异。以下是一个使用Java 8的Stream API进行数字相加的例子，用以说明这两种流的区别：

```java
import java.util.Arrays;
import java.util.List;

public class ParallelStreamExample {
    public static void main(String[] args) {
        // 创建一个包含10个元素的列表
        List<Integer> numbers = Arrays.asList(1, 2, 3, 4, 5, 6, 7, 8, 9, 10);

        // 使用串行流计算所有数字的和
        long sumSerial = numbers.stream().mapToInt(Integer::intValue).sum();
        System.out.println("Sum using serial stream: " + sumSerial);

        // 使用并行流计算所有数字的和
        long sumParallel = numbers.parallelStream().mapToInt(Integer::intValue).sum();
        System.out.println("Sum using parallel stream: " + sumParallel);
    }
}
```

#### 优缺点

**并行流**在处理大规模数据时，由于其可以充分利用多核处理器的优势，实现多个元素的同步处理，因此执行效率较高。但相较于串行流，由于线程间的切换以及线程资源的争抢等问题，可能会增加一定的开销。另外，对于某些特定的任务，如数据处理的逻辑非常复杂，或者对结果的顺序有要求等，可能并不适合使用并行流。

**串行流**的执行逻辑较为简单直观，易于理解和操作。并且，串行流在处理小规模数据或者对性能要求不高的情况下，执行效率也是可以接受的。然而，如果需要处理的数据量非常大，那么串行流由于其顺序处理数据的特点，可能会导致程序的执行效率较低。
