# lang包

 java.lang包是提供利用java编程语言进行程序设计的基础类，在项目中使用的时候不需要import。

![](C:\Users\zhangxiaoliang\Desktop\总结\java常用类库\java lang包结构图.png)

## Object

object是所有类的超类

Object类定义了一些有用的方法，由于是根类，这些方法在其他类中都存在，一般是进行重载或者重写覆盖，实现了给子类的具体功能。比如：

equals：返回值类型boolean，比较两个对象是否相同

hashCode：返回值类型int，返回对象的哈希码值

toString：返回值类型String，返回对象的字符串表示形式

在Obect类中有些方法使用native修改，native修饰示该方法的实现不是由java语言编写的，native可以与其他修饰符一起使用，但是不能与abstract一起使用，abstract表示该方式是抽象的，不能有方法体，而native虽然没有写方法体，实质是存在方法体的。

注意：我们经常会比较两个字符创内容是否相同，会使用equals()方法，或者使用工具类StringUtils，其实字符串String已经重写了equals方法，所以可以直接使用比较两个字符串内容是否一致，如代码1，如果要比较一个自定义的实体类，那么就需要我们自己重写equals()，比如代码2，当然如果使用lombok

代码1：   

```java
public boolean equals(Object anObject) {
        if (this == anObject) {
            return true;
        }
        if (anObject instanceof String) {
            String anotherString = (String)anObject;
            int n = value.length;
            if (n == anotherString.value.length) {
                char v1[] = value;
                char v2[] = anotherString.value;
                int i = 0;
                while (n-- != 0) {
                    if (v1[i] != v2[i])
                        return false;
                    i++;
                }
                return true;
            }
        }
        return false;
    }
```

代码2：


```java
public class TestEquals {
    private  String name;

    private  String address;

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public String getAddress() {
        return address;
    }

    public void setAddress(String address) {
        this.address = address;
    }

    public boolean equals(TestEquals testEquals){
        return this == testEquals || 				   (StringUtils.equals(this.getAddress(),testEquals.getAddress())
                && StringUtils.equals(this.getName(),testEquals.getName()));
    }
}
```

测试代码：

```java
public static void main(String[] args) {
    TestEquals testEquals = new TestEquals();
    testEquals.setAddress("地球");
    testEquals.setName("中国");
    TestEquals testEquals1 = new TestEquals();
    testEquals1.setName("中国");
    testEquals1.setAddress("地球");
    System.out.println(testEquals.equals(testEquals1));
}
```
结果：
true

## 包装类

Boolean，Character，Byte，Short，Integer，Long，Float，Double

知识点：装箱和拆箱，类型转换

自动装箱：自动加基本数据类型转成包装类，如：Integer i = 1；

自动拆箱：自动将包装类转成基本数据类型，如：Integer i = 2；int n = i;

基本类型	包装类	字节(byte)	位数(bit)	范围
boolean	Boolean	N/A		
byte	Byte	1 字节	8 位	-128 ~ 127
char	Character	2 字节	16 位	Unicode0 ~ Unicode(2^16)-1
short	Short	2 字节	16 位	-2^15 ~ 2^15 - 1
int	Integer	4 字节	32 位	-2^31 ~ 2^31 - 1
long	Long	8 字节	64 位	-2^63 ~ 2^63 - 1
float	Float	4 字节	32 位	IEEE754 ~ IEEE754
double	Double	8 字节	64 位	IEEE754 ~ IEEE754
装箱过程是通过调用包装器的valueOf方法实现的，而拆箱过程是通过调用包装器的 xxxValue方法实现的。（xxx代表对应的基本数据类型）。

比较常见的问题，如下：


​         
```java
public class Main {
    public static void main(String[] args) {
        Integer i1 = 100;
        Integer i2 = 100;
        Integer i3 = 200;
        Integer i4 = 200;

        System.out.println(i1==i2);
        System.out.println(i3==i4);
	}
}
```

结果：

true
false
原因：其实查看Integer的ValueOf方法的源码就知道了

```java
//IntegerCache.high = 127
public static Integer valueOf(int i) {
        if(i >= -128 && i <= IntegerCache.high){
            return IntegerCache.cache[i + 128];
        }else{
            return new Integer(i);
        }
}
```


如果i在[-128,127]集合内，返回指向的是IntegerCache.cache中已经存在的对象的引用，否则创建一个新的Integer对象，其实包装类的ValueOf方法的实现方式不相同，比如：Double每次都会new一个Double对象，Integer只是一个例子，如果想了解其他包装类的实现可以看看源码。

比如：Double和Float每次都会创建一个新对象

```java
public static Double valueOf(double d) {
    return new Double(d);
}
public static Float valueOf(float f) {
    return new Float(f);
}
```
Short、Long和Integer类似

```java
public static Short valueOf(short s) {
    final int offset = 128;
    int sAsInt = s;
    if (sAsInt >= -128 && sAsInt <= 127) { 
        return ShortCache.cache[sAsInt + offset];
    }
    return new Short(s);
}
public static Long valueOf(long l) {
    final int offset = 128;
    if (l >= -128 && l <= 127) { // will cache
        return LongCache.cache[(int)l + offset];
    }
    return new Long(l);
}
```
为什么要有包装类？

我们知道Java是一个面相对象的编程语言，基本类型并不具有对象的性质，为了让基本类型也具有对象的特征，就出现了包装类型（如我们在使用集合类型Collection时就一定要使用包装类型而非基本类型），它相当于将基本类型“包装起来”，使得它具有了对象的性质，并且为其添加了属性和方法，丰富了基本类型的操作。

包装类和基本数据类型的区别

声明方式不同：
基本类型不使用new关键字，而包装类型需要使用new关键字来在堆中分配存储空间；

存储方式及位置不同：
基本类型是直接将变量值存储在栈中，而包装类型是将对象放在堆中，然后通过引用来使用；

初始值不同：
基本类型的初始值如int为0，boolean为false，而包装类型的初始值为null；

使用方式不同：
基本类型直接赋值直接使用就好，而包装类型在集合如Collection、Map时会使用到。

## 字符串

String ，StringBuilder， StringBuffer

在java.lang中还提供了处理字符串的String类，String类用于处理“不可变”的字符串，它们的值在创建后不能被更改；相对地，还提供了StringBuffer类和StringBuilder用于处理“可变”字符串。Stirng类，StringBuffer类和StringBuilder都被声明为final类型，因此不能将其当做父类再被继承使用了。

StringBuilder类和StringBuffer类是字符串缓冲区类，用于创建长度可变的字符串对象，这里的“长度可变”是指通过某些方法的调用可以改变字符串的长度和内容，比如通过在原字符串的基础上追加新的字符串序列，或者在原字符串的某个位置上插入新的字符序列等构成新的字符串对象。常用的方法：append()和insert()，比如：

```java
    StringBuilder sb = new StringBuilder("abc");
    System.out.println("1:"+sb);
    sb.append("d");
    System.out.println("2:"+sb);
    sb.insert(1,"f");
    System.out.println("3:"+sb);
```
结果：

1:abc
2:abcd
3:afbcd

### String，StringBuffer和StringBuilder的区别

1.首先说运行速度，或者说是执行速度，在这方面运行速度快慢为：StringBuilder > StringBuffer > String

原因：String为字符串常量，而StringBuilder和StringBuffer均为字符串变量，即String对象一旦创建之后该对象是不可更改的，但后两者的对象是变量，是可以更改的

Java中对String对象进行的操作实际上是一个不断创建新的对象并且将旧的对象回收的一个过程，所以执行速度很慢。

而StringBuilder和StringBuffer的对象是变量，对变量进行操作就是直接对该对象进行更改，而不进行创建和回收的操作，所以速度要比String快很多

2.线程安全

在线程安全上，StringBuilder是线程不安全的，而StringBuffer是线程安全的

如果一个StringBuffer对象在字符串缓冲区被多个线程使用时，StringBuffer中很多方法可以带有synchronized关键字，所以可以保证线程是安全的，但StringBuilder的方法则没有该关键字，所以不能保证线程安全，有可能会出现一些错误的操作。所以如果要进行的操作是多线程的，那么就要使用StringBuffer，但是在单线程的情况下，还是建议使用速度比较快的StringBuilder。

String直接赋值与new String()的区别

举例说明：

        String s1 = "abc";
        String s2 = new String("abc");
        String s3 = new String("ccc");
        String s4 = new String("ccc");
        System.out.println(s1 == s2);
        System.out.println(s3 == s4);
结果：
false
false
首先s1是直接赋值，那么首先会去字符串常量池中查找是否存在abc，如果存在则直接将其内存地址赋值给s1，若不存在就会在字符串的常量池中创建，其次s2首先会在堆中创建字符串对象，并将引用地址赋值给s2，查看常量池中是否存在abc对象，如果存在就不再创建，若不存在则在常量池中创建

## System

System类代表系统，系统级的很多属性和控制方法都放置在该类的内部。由于该类的构造方法是private的，所以无法创建该类的对象，也就是无法实例化该类。其内部的成员变量和成员方法都是static的，所以也可以很方便的进行调用。

成员变量：in、out、err，分别代表标准输入流(键盘输入)，标准输出流(显示器)和标准错误输出流(显示器)。

常用方法：

返回值	方法	功能
static void	arraycopy(Object src, int srcPos, Object dest, int destPos, int length)	将指定源数组中的数组从指定位置复制到目标数组的指定位置。
static void	currentTimeMillis()	返回当前时间（以毫秒为单位）。
static Properties	getProperties()	获取指定键指示的系统属性。
static void	gc()	运行垃圾回收器。
注意：在项目中不能使用system.out.println()，首先System.out.println只是将信息输出到控制台，使用logger日志是可以输出到指定的文件中，方便后续查看，还有更严重的问题就是将信息直接输出到控制台会导致内存占用增大，甚至溢出，项目崩溃等问题。

## Math

math类中包含执行基本数学运算的方法，比如：绝对值-abs()、较大值-max()、较小值-min{}、四舍五入-round()

## Throwable

### 1.Throwable

Throwable是 Java 语言中所有错误或异常的超类。
Throwable包含两个子类: Error 和 Exception。它们通常用于指示发生了异常情况。
Throwable包含了其线程创建时线程执行堆栈的快照，它提供了printStackTrace()等接口用于获取堆栈跟踪数据等信息。

### 2.Exception

Exception及其子类是 Throwable 的一种形式，它指出了合理的应用程序想要捕获的条件。

### 3.RuntimeException

RuntimeException是那些可能在 Java 虚拟机正常运行期间抛出的异常的超类。
编译器不会检查RuntimeException异常。例如，除数为零时，抛出ArithmeticException异常。RuntimeException是ArithmeticException的超类。当代码发生除数为零的情况时，倘若既"没有通过throws声明抛出ArithmeticException异常"，也"没有通过try…catch…处理该异常"，也能通过编译。这就是我们所说的"编译器不会检查RuntimeException异常"！
如果代码会产生RuntimeException异常，则需要通过修改代码进行避免。例如，若会发生除数为零的情况，则需要通过代码避免该情况的发生！

### 4.Error

和Exception一样，Error也是Throwable的子类。它用于指示合理的应用程序不应该试图捕获的严重问题，大多数这样的错误都是异常条件。
和RuntimeException一样，编译器也不会检查Error

## 异常分类

Java将可抛出(Throwable)的结构分为三种类型：被检查的异常(Checked Exception)，运行时异常(RuntimeException)和错误(Error)。

### 运行时异常

定义: RuntimeException及其子类都被称为运行时异常。
特点: Java编译器不会检查它。也就是说，当程序中可能出现这类异常时，倘若既"没有通过throws声明抛出它"，也"没有用try-catch语句捕获它"，还是会编译通过。例如，除数为零时产生的ArithmeticException异常，数组越界时产生的IndexOutOfBoundsException异常，fail-fail机制产生的ConcurrentModificationException异常等，都属于运行时异常。
虽然Java编译器不会检查运行时异常，但是我们也可以通过throws进行声明抛出，也可以通过try-catch对它进行捕获处理。
如果产生运行时异常，则需要通过修改代码来进行避免。例如，若会发生除数为零的情况，则需要通过代码避免该情况的发生！

### 被检查的异常

定义: Exception类本身，以及Exception的子类中除了"运行时异常"之外的其它子类都属于被检查异常。
特点: Java编译器会检查它。此类异常，要么通过throws进行声明抛出，要么通过try-catch进行捕获处理，否则不能通过编译。例如，CloneNotSupportedException就属于被检查异常。当通过clone()接口去克隆一个对象，而该对象对应的类没有实现Cloneable接口，就会抛出CloneNotSupportedException异常。
被检查异常通常都是可以恢复的。

### 错误

定义: Error类及其子类。
特点: 和运行时异常一样，编译器也不会对错误进行检查。
当资源不足、约束失败、或是其它程序无法继续运行的条件发生时，就产生错误。程序本身无法修复这些错误的。例如，VirtualMachineError就属于错误。
我们最常见的异常就是运行时异常（非检查异常）

异常	描述
ArithmeticException	当出现异常的运算条件时，抛出此异常。例如，一个整数"除以零"时，抛出此类的一个实例。
ArrayIndexOutOfBoundsException	用非法索引访问数组时抛出的异常。如果索引为负或大于等于数组大小，则该索引为非法索引。
ArrayStoreException	试图将错误类型的对象存储到一个对象数组时抛出的异常。
ClassCastException	当试图将对象强制转换为不是实例的子类时，抛出该异常
IllegalArgumentException	抛出的异常表明向方法传递了一个不合法或不正确的参数。
IndexOutOfBoundsException	指示某排序索引（例如对数组、字符串或向量的排序）超出范围时抛出。
NullPointerException	当应用程序试图在需要对象的地方使用 null 时，抛出该异常
NumberFormatException	当应用程序试图将字符串转换成一种数值类型，但该字符串不能转换为适当格式时，抛出该异常。
我们常见的检查异常

异常	描述
ClassNotFoundException	应用程序试图加载类时，找不到相应的类，抛出该异常。
NoSuchFieldException	请求的变量不存在
NoSuchMethodException	请求的方法不存在
IOException	表示发生某种类型的I / O异常。 此类是由失败或中断的I / O操作产生的一般异常类。
SQLException	提供有关数据库访问错误或其他错误的信息的异常。
FileNotFoundException	指示尝试打开由指定路径名表示的文件失败。
EOFException	表示在输入过程中意外地到达文件结束或流结束。

## 异常处理

异常的处理方式有两个：

方法1：通过throws抛出异常，由调用者处理该异常，例如：

```java
private static final Logger logger = LoggerFactory.getLogger(TestClass.class);

//调用者处理异常
public static void main(String[] args)  {
    try {
        String file = getFile();
        System.out.println(file);
    } catch (IOException e) {
       logger.error("读取文件异常",e);
    }
}

   //此方法通过throws方法将异常抛出
    private static String getFile() throws IOException {
        File file = new File("C:\\文件\\test.txt");
        FileReader fileReader = new FileReader(file);
        BufferedReader bufferedReader = new BufferedReader(fileReader);
        StringBuilder stringBuilder = new StringBuilder();
        String readLine ;
        while ((readLine = bufferedReader.readLine()) != null){
            stringBuilder.append(readLine);
        }
        return stringBuilder.toString();
    }
```

方法2：通过try…catch…捕获异常，并处理

```java
private static final Logger logger = LoggerFactory.getLogger(TestClass.class);

public static void main(String[] args)  {
    String file = getFile();
    System.out.println(file);
}

private static String getFile()  {
    File file = new File("C:\\文件\\test.txt");
    try (FileReader  fileReader = new FileReader(file);
         BufferedReader bufferedReader = new BufferedReader(fileReader);
        ) {
        StringBuilder stringBuilder = new StringBuilder();
        String readLine ;
        while ((readLine = bufferedReader.readLine()) != null){
            stringBuilder.append(readLine);
        }
        return stringBuilder.toString();
    }catch (Exception e){
        logger.error("读取文件异常",e);
    }
    return null;
}
```




## 注释类型

Override
override表示方法声明旨在覆盖父类的方法

小知识：

重载和重写

对比项	   重载	                      重写
关键字	overload	               override
概念	   一个方法的多样性	子类重新实现父类已有的方法
环境	   同一个类中	父类，子类的继承关系中
方法名	必须相同	             必须相同
参数列表	必须不同	          必须相同
返回值	  没有要求	            必须相同
访问权限	没有要求	          不能比父类更加严格
异常	        没有要求	          不能比父类抛出更大的异常