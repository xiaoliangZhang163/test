# java Stream

## 1.为什么java8中加入Stream

Stream 作为 Java 8 的一大亮点，它与 java.io 包里的 InputStream 和 OutputStream 是完全不同的概念。Java 8 中的 Stream 是对集合（Collection）对象功能的增强，它专注于对集合对象进行各种非常便利、高效的聚合操作（aggregate operation），或者大批量数据操作 (bulk data operation)。尤其是对于数据从业人员来说，对数据做各种操作转换是再正常不过的需求，基本每天都会用到。例如下面这么一个简单的小需求：求一个集合中字符串长度小于5的数量。 
在java8之前，我们一般这么做：

```java
@Test
public void lenIter() {
    List<String> list = Arrays.asList("java", "scala", "python", "shell", "ruby");
    int num = 0;
    for(String lan: list) {
        if(lan.length() < 5) {
            num++;
        }
    }
    System.out.println(num);
}
```
这段代码逻辑很简单，但是显得很冗长，可读性嘛也就呵呵了。如果用Stream，我们可以这样：

```java
@Test
public void lenStream() {
    List<String> list = Arrays.asList("java", "scala", "python", "shell", "ruby");
    long num = list.parallelStream().filter(x -> x.length() < 5).count();
    System.out.println(num);
}
```
代码量明显减少而且逻辑特别清楚，即使不懂代码的人看到也能猜出来是什么意思。如果大家了解过函数式编程，就会觉得特别亲切自然。

## 2.什么是Stream

Stream 不是集合元素，它不是数据结构并不保存数据，它是有关算法和计算的，它更像一个高级版本的 Iterator。原始版本的 Iterator，用户只能显式地一个一个遍历元素并对其执行某些操作；高级版本的 Stream，用户只要给出需要对其包含的元素执行什么操作，比如 “过滤掉长度大于 10 的字符串”、“获取每个字符串的首字母”等，Stream 会隐式地在内部进行遍历，做出相应的数据转换。 
Stream 就如同一个迭代器（Iterator），单向，不可往复，数据只能遍历一次，遍历过一次后即用尽了，就好比流水从面前流过，一去不复返。 
而和迭代器又不同的是，Stream 可以并行化操作，迭代器只能命令式地、串行化操作。顾名思义，当使用串行方式去遍历时，每个 item 读完后再读下一个 item。而使用并行去遍历时，数据会被分成多个段，其中每一个都在不同的线程中处理，然后将结果一起输出。Stream 的并行操作依赖于 Java7 中引入的 Fork/Join 框架（JSR166y）来拆分任务和加速处理过程。

Stream和Collection的区别主要有： 
1.stream本身并不存储数据，数据是存储在对应的collection里，或者在需要的时候才生成的； 
2.stream不会修改数据源，总是返回新的stream； 
3.stream的操作是懒执行(lazy)的：仅当最终的结果需要的时候才会执行，比如上面的例子中，结果仅需要前3个长度大于7的字符串，那么在找到前3个长度符合要求的字符串后，filter()将停止执行；

使用stream的步骤如下： 
1.创建stream； 
2.通过一个或多个中间操作(intermediate operations)将初始stream转换为另一个stream； 
3.通过中止操作(terminal operation)获取结果；该操作触发之前的懒操作的执行，中止操作后，该stream关闭，不能再使用了；

## 3.创建Stream的方法

### 3.1 使用Collection下的 stream() 和 parallelStream() 方法

```java
List<String> list = new ArrayList<>();
Stream<String> stream = list.stream(); //获取一个顺序流
Stream<String> parallelStream = list.parallelStream(); //获取一个并行流
```

### 3.2 使用Arrays 中的 stream() 方法，将数组转成流

```java
Integer[] nums = new Integer[10];
Stream<Integer> stream = Arrays.stream(nums);
```

### 3.3 使用Stream中的静态方法：of()、iterate()、generate()

```java
Stream<Integer> stream = Stream.of(1,2,3,4,5,6);

Stream<Integer> stream2 = Stream.iterate(0, (x) -> x + 2).limit(6);
stream2.forEach(System.out::println); // 0 2 4 6 8 10

Stream<Double> stream3 = Stream.generate(Math::random).limit(2);
stream3.forEach(System.out::println);
```

### 3.4 使用 BufferedReader.lines() 方法，将每行内容转成流

```java
BufferedReader reader = new BufferedReader(new FileReader("F:\\test_stream.txt"));
Stream<String> lineStream = reader.lines();
lineStream.forEach(System.out::println);
```

### 3.5 使用 Pattern.splitAsStream() 方法，将字符串分隔成流

```java
Pattern pattern = Pattern.compile(",");
Stream<String> stringStream = pattern.splitAsStream("a,b,c,d");
stringStream.forEach(System.out::println);
```



## 4.Stream的转换

Stream最大的用途就是各种转换了。跟Spark中的Rdd类似，Rdd里面也是各种transfer操作。 

### 4.1  筛选与切片 

​    filter：过滤流中的某些元素
​    limit(n)：获取n个元素
​    skip(n)：跳过n元素，配合limit(n)可实现分页
​    distinct：通过流中元素的 hashCode() 和 equals() 去除重复元素 

```java
Stream<Integer> stream = Stream.of(6, 4, 6, 7, 3, 9, 8, 10, 12, 14, 14);
 
Stream<Integer> newStream = stream.filter(s -> s > 5) //6 6 7 9 8 10 12 14 14
        .distinct() //6 7 9 8 10 12 14
        .skip(2) //9 8 10 12 14
        .limit(2); //9 8
newStream.forEach(System.out::println);
```
### 4.2 映射

  map：接收一个函数作为参数，该函数会被应用到每个元素上，并将其映射成一个新的元素。
  flatMap：接收一个函数作为参数，将流中的每个值都换成另一个流，然后把所有流连接成一个流。 

```java
List<String> list = Arrays.asList("a,b,c", "1,2,3");

//将每个元素转成一个新的且不带逗号的元素
Stream<String> s1 = list.stream().map(s -> s.replaceAll(",", ""));
s1.forEach(System.out::println); // abc  123

Stream<String> s3 = list.stream().flatMap(s -> {
    //将每个元素转换成一个stream
    String[] split = s.split(",");
    Stream<String> s2 = Arrays.stream(split);
    return s2;
});
s3.forEach(System.out::println); // a b c 1 2 3
```

### 4.3 排序

sorted()：自然排序，流中元素需实现Comparable接口
sorted(Comparator com)：定制排序，自定义Comparator排序器 

```java
List<String> list = Arrays.asList("aa", "ff", "dd");
//String 类自身已实现Compareable接口
list.stream().sorted().forEach(System.out::println);// aa dd ff

Student s1 = new Student("aa", 10);
Student s2 = new Student("bb", 20);
Student s3 = new Student("aa", 30);
Student s4 = new Student("dd", 40);
List<Student> studentList = Arrays.asList(s1, s2, s3, s4);

//自定义排序：先按姓名升序，姓名相同则按年龄升序
studentList.stream().sorted(
        (o1, o2) -> {
            if (o1.getName().equals(o2.getName())) {
                return o1.getAge() - o2.getAge();
            } else {
                return o1.getName().compareTo(o2.getName());
            }
        }
).forEach(System.out::println);
```

### 4.4 消费

        peek：如同于map，能得到流中的每一个元素。但map接收的是一个Function表达式，有返回值；而peek接收的是Consumer表达式，没有返回值。

```java
Student s1 = new Student("aa", 10);
Student s2 = new Student("bb", 20);
List<Student> studentList = Arrays.asList(s1, s2);

studentList.stream()
        .peek(o -> o.setAge(100))
        .forEach(System.out::println);   

//结果：
Student{name='aa', age=100}
Student{name='bb', age=100}         
```

   

## 5.流的终止操作

### 5.1 匹配、聚合操作

        allMatch：接收一个 Predicate 函数，当流中每个元素都符合该断言时才返回true，否则返回false
        noneMatch：接收一个 Predicate 函数，当流中每个元素都不符合该断言时才返回true，否则返回false
        anyMatch：接收一个 Predicate 函数，只要流中有一个元素满足该断言则返回true，否则返回false
        findFirst：返回流中第一个元素
        findAny：返回流中的任意元素
        count：返回流中元素的总个数
        max：返回流中元素最大值
        min：返回流中元素最小值

```java
List<Integer> list = Arrays.asList(1, 2, 3, 4, 5);

boolean allMatch = list.stream().allMatch(e -> e > 10); //false
boolean noneMatch = list.stream().noneMatch(e -> e > 10); //true
boolean anyMatch = list.stream().anyMatch(e -> e > 4);  //true

Integer findFirst = list.stream().findFirst().get(); //1
Integer findAny = list.stream().findAny().get(); //1

long count = list.stream().count(); //5
Integer max = list.stream().max(Integer::compareTo).get(); //5
Integer min = list.stream().min(Integer::compareTo).get(); //1
```

### 5.2 规约操作

        Optional<T> reduce(BinaryOperator<T> accumulator)：第一次执行时，accumulator函数的第一个参数为流中的第一个元素，第二个参数为流中元素的第二个元素；第二次执行时，第一个参数为第一次函数执行的结果，第二个参数为流中的第三个元素；依次类推。
        T reduce(T identity, BinaryOperator<T> accumulator)：流程跟上面一样，只是第一次执行时，accumulator函数的第一个参数为identity，而第二个参数为流中的第一个元素。
        <U> U reduce(U identity,BiFunction<U, ? super T, U> accumulator,BinaryOperator<U> combiner)：在串行流(stream)中，该方法跟第二个方法一样，即第三个参数combiner不会起作用。在并行流(parallelStream)中,我们知道流被fork join出多个线程进行执行，此时每个线程的执行流程就跟第二个方法reduce(identity,accumulator)一样，而第三个参数combiner函数，则是将每个线程的执行结果当成一个新的流，然后使用第一个方法reduce(accumulator)流程进行规约。

```java
//经过测试，当元素个数小于24时，并行时线程数等于元素个数，当大于等于24时，并行时线程数为16
List<Integer> list = Arrays.asList(1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11, 12, 13, 14, 15, 16, 17, 18, 19, 20, 21, 22, 23, 24);

Integer v = list.stream().reduce((x1, x2) -> x1 + x2).get();
System.out.println(v);   // 300

Integer v1 = list.stream().reduce(10, (x1, x2) -> x1 + x2);
System.out.println(v1);  //310

Integer v2 = list.stream().reduce(0,
        (x1, x2) -> {
            System.out.println("stream accumulator: x1:" + x1 + "  x2:" + x2);
            return x1 - x2;
        },
        (x1, x2) -> {
            System.out.println("stream combiner: x1:" + x1 + "  x2:" + x2);
            return x1 * x2;
        });
System.out.println(v2); // -300

Integer v3 = list.parallelStream().reduce(0,
        (x1, x2) -> {
            System.out.println("parallelStream accumulator: x1:" + x1 + "  x2:" + x2);
            return x1 - x2;
        },
        (x1, x2) -> {
            System.out.println("parallelStream combiner: x1:" + x1 + "  x2:" + x2);
            return x1 * x2;
        });
System.out.println(v3); //197474048
```

### 5.3 收集操作

        collect：接收一个Collector实例，将流中元素收集成另外一个数据结构。
        Collector<T, A, R> 是一个接口，有以下5个抽象方法：
            Supplier<A> supplier()：创建一个结果容器A
            BiConsumer<A, T> accumulator()：消费型接口，第一个参数为容器A，第二个参数为流中元素T。
            BinaryOperator<A> combiner()：函数接口，该参数的作用跟上一个方法(reduce)中的combiner参数一样，将并行流中各个子进程的运行结果(accumulator函数操作后的容器A)进行合并。
            Function<A, R> finisher()：函数式接口，参数为：容器A，返回类型为：collect方法最终想要的结果R。
            Set<Characteristics> characteristics()：返回一个不可变的Set集合，用来表明该Collector的特征。有以下三个特征：
                CONCURRENT：表示此收集器支持并发。（官方文档还有其他描述，暂时没去探索，故不作过多翻译）
                UNORDERED：表示该收集操作不会保留流中元素原有的顺序。
                IDENTITY_FINISH：表示finisher参数只是标识而已，可忽略。
        注：如果对以上函数接口不太理解的话，可参考我另外一篇文章：Java 8 函数式接口



collect()方法可以对stream中的元素进行各种处理后，得到stream中元素的值。并且Collectors接口提供了很方便的创建Collector对象的工厂方法。

```java
@Test
public void collectTest() {
    List<String> list = Stream.of("hello", "world", "hello", "java").collect(Collectors.toList());
    list.forEach(x -> System.out.print(x + " "));
    System.out.println();
    Set<String> set = Stream.of("hello", "world", "hello", "java").collect(Collectors.toSet());
    set.forEach(x -> System.out.print(x + " "));
    System.out.println();
    Set<String> treeset = Stream.of("hello", "world", "hello", "java").collect(Collectors.toCollection(TreeSet::new));
    treeset.forEach(x -> System.out.print(x + " "));
    System.out.println();
    String resultStr = Stream.of("hello", "world", "hello", "java").collect(Collectors.joining(","));
    System.out.println(resultStr);
}
```
最后的输出结果为：

hello world hello java 
world java hello 
hello java world 
hello,world,hello,java