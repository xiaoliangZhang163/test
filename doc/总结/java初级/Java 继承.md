# Java 继承

继承是所有OOP语言和Java语言不可缺少的组成部分。当创建一个类时，总是在继承，因此，除非已明确指出要从其他类中继承，否则就是在隐式地从Java的标准根类Object进行继承。

## 1.什么是继承

继承就是子类继承父类的特征和行为，使得子类对象（实例）具有父类的实例域和方法，或子类从父类继承方法，使得子类具有父类相同的行为。

## 2.继承的语法

关键字 extends 表明正在构造的新类派生于一个已存在的类。已存在的类被称为超类（super class）、基类（base class）或父类（parent class）；新类被称为子类（subclass）、派生类（derived class）或孩子类（child class）。

```java
public class Employee {
    private String name;
    private double salary;
    private Date hireDay;

public Employee(String n, double s, int year, int month, int day)
{
    name = n;
    salary = s;
    GregorianCalendar calendar = new GregorianCalendar(year, month, day);
    hireDay = calendar.getTime();
}

public String getName() {
    return name;
}

//more method
......
```

}
尽管 Employee 类是一个超类，但并不是因为它位于子类之上或者拥有比子类更多的功能。恰恰相反，子类比超类拥有的功能更加丰富。在 Manager 类中，增加了一个用于存储奖金信息的域，以及一个用于设置这个域的方法：

```java
public class Manager extends Employee{
    ...
    private double bonus;

    public void setBonus(double bonus) {
        this.bonus = bonus;
    }

}
```


然而，尽管在 Manager 类中没有显式地定义 getName 和 getHireDay 等方法，但属于 Manager 类的对象却可以使用它们，这是因为 Manager 类自动地继承了超类 Employee 中的这些方法。同样，从超类中还继承了 name、salary 和 hireDay 这 3 个域。这样一来，每个 Manager 类对象就包含了4个域：name、salary、hireDay 和 bonus。

在通过扩展超类定义子类的时候，仅需要指出子类与超类的不同之处。因此在设计类的时候，应该将通用的方法放到超类中，而将具有特色用途的方法放在子类中，这种将通用的功能放到超类的做法，在面向对象程序设计中十分普遍。

然而，超类中的有些方法对子类 Manager 并不一定适用。例如，在 Manager 类中的 getSalary 方法应该返回薪水和奖金的总和。为此，需要提供一个新的方法来覆盖（override）超类中的这个方法：

```java
@Override
public double getSalary() {
    return salary + bonus;//won't work
}
```


然而，这个方法并不能运行。这是因为 Manager 类的 getSalary 方法不能直接地访问超类的私有域。也就是说，尽管每个 Manager 对象都拥有一个名为 salary 的域，但在 Manager 类的 getSalary 方法中并不能够直接地访问 salary 域。只有 Employee 类的方法才能够访问私有部分。如果 Manager 类的方法一定要访问私有域，就必须借助共有的接口，Employee 类中的共有方法正式这样一个接口。

```java
@Override
public double getSalary() {
    return super.getSalary() + bonus;
}
```


super 关键字有两个用途：一是调用超类的方法，二是调用超类的构造器。super 不是一个对象的引用，不能将 super 赋给另一个对象变量，它只是一个指示编译器调用超类方法的特有关键字。

由于 Manager 类的构造器不能访问 Employee 类的私有域，所以必须利用 Employee 类的构造器对这部分私有域进行初始化，我们可以通过 super 实现对超类构造器的调用。使用 super 调用构造器的语句必须是子类构造器的第一条语句。

## 3.继承初始化过程

在继承关系中，子类具有父类相同的行为，当子类调用父类方法时，如何确保父类的实例域是正确初始化的？当初始化子类过程中，如何确保父类也得到正确的初始化？

```java
public class Animal {
    private static int A = printInit("static Animal region init");

    public Animal()
    {
        System.out.println("--Animal--");
    }
     
    public static int printInit(String s)
    {
        System.out.println(s);
        return 30;
    }

}

public class Bird extends Animal{
    private static int B = Animal.printInit("static Bird region init");
    

    public Bird()
    {
        System.out.println("--Bird--");
    }

}

public class Parrot extends Bird{
    private static int P = Animal.printInit("static Parrot region init");
    

    public Parrot()
    {
        System.out.println("--Parrot--");
    }
     
    public static void main(String[] arg)
    {
        Parrot parrot = new Parrot();
    }

}
```

运行结果：

```java
static Animal region init
static Bird region init
static Parrot region init
--Animal--
--Bird--
--Parrot--
```


接下来看看 Parrot、Bird、Animal 类构造器字节码：

```java
/** Parrot **/
public com.mdj.test.Parrot();
    descriptor: ()V
    flags: ACC_PUBLIC
    Code:
      stack=1, locals=1, args_size=1
         0: aload_0
         1: invokespecial #1                  // Method com/mdj/test/Bird."<init>":()V
         4: return

/** Bird **/
  public com.mdj.test.Bird();
    descriptor: ()V
    flags: ACC_PUBLIC
    Code:
      stack=1, locals=1, args_size=1
         0: aload_0
         1: invokespecial #1                  // Method com/mdj/test/Animal."<init>":()V
         4: return

/** Animal **/
  public com.mdj.test.Animal();
    descriptor: ()V
    flags: ACC_PUBLIC
    Code:
      stack=1, locals=1, args_size=1
         0: aload_0
         1: invokespecial #1                  // Method java/lang/Object."<init>":()V
         4: return
```

当创建 Parrot 对象实例时，会生成构造函数链 Parrot -> Bird -> Animal -> Object，最后才执行 Parrot 构造函数初始化：

![1571646475348](C:\Users\zhangxiaoliang\Desktop\总结\1571646475348.png)

从运行结果可以看出，根基类的 static 会首先执行，然后是下一个导出类，以此类推。这种方式很重要，因为导出类的 static 初始化可能会依赖于基类成员能否被正确初始化。

基类的构造器总是在导出类的构造过程中被调用，而且按照继承层次逐渐向上链接，以使每个基类的构造器都能得到调用。这样做是有意义的，因为构造器具有一项特殊的任务：检查对象是否被正确的构造。导出类只能访问它自己的成员，不能访问基类的成员（基类成员通常是 private）。只有基类的构造器才具有恰当的知识和权限来对自己的元素进行初始化。因此必须令所有的构造器都得到调用，否则就不可能正确构造完整对象。这正是编译器为什么要强制每个导出类都必须调用构造器的原因。在构造器内部，我们必须确保所要使用的成员都已经构建完毕。为确保这一目的，唯一的办法就是首先调用基类构造器。

Object是所有的类的基类，Java会自动在导出类的构造器中插入对基类构造器的调用(调用类的实例构造器<init>)。

## 4.继承的分类

根据继承的特性可以分为纯继承与扩展。

纯继承关系是纯粹的“is-a”(是一种)关系，因为一个类的接口已经确定了它应该是什么。继承可以确保所有的导出类具有基类的接口，且绝对不会少。基类可以接收发送给导出类的任何消息，因为二者有着完全相同的接口。

扩展关系“is-like-a”(像一个)关系，导出类就像是一个基类——它有着相同基类的基本接口，但是它还具有由额外方法实现的其他特性。导出类中接口扩展部分不能被基类访问，因此，一旦向上转型，就不能调用那些新方法。

## 5.继承的特性

Java 语言的继承是单继承，不允许一个类继承多个父类。

继承最重要的方面是用来表现新类和基类之间的关系。这种关系可以用“新类是现有类的一种类型”。

继承可以把新类向上转换成基类，这是向上转型的一种表现。
由导出类转型成基类，在继承图上是向上移动的。
向上转型是从一个较专用类型向较通用类型转换。
导出类是基类的一个超集。它可能比基类含有更多的方法，但它必须至少具备基类中所有的方法。

## 6.继承的优缺点

在面向对象语言中，继承是必不可少的、非常优秀的语言机制，它有如下优点：

代码共享，减少创建类的工作量，每个子类都拥有父类的方法和属性；
提高代码的重用性；
子类可以形似父类，但又异于父类；
提高代码的可扩展性，实现父类的方法就可以“为所欲为”了。
提高产品或项目的开放性。
自然界的所有事物都是优点和缺点并存的，继承的缺点如下：

继承是侵入性的。只要继承，就必须拥有父类的所有属性和方法；
降低代码的灵活性。子类必须拥有父类的属性和方法，让子类自由的世界中多了些约束；
增强了耦合性。当父类的常量、变量和方法被修改时，需要考虑子类的修改，而且在缺乏规范的环境下，这种修改可能带来非常糟糕的结果——大段的代码需要重构。

## 7.继承的扩展

在上面提到过：当创建了一个导出类的一个对象时，这个子对象和你直接用基类创建的对象是一样的。二者区别在于，后者来自外部，而基类的子对象被包装在导出类对象内部。那是不是真的会在导出类的内部再 new 一个父类的对象？

```java
public class Widget {
    public synchronized void doSomething() {}
}

public class LoggingWidget extends Widget {
    public synchronized void doSomething() {
        super.doSomething();
    }

    public static void main(String[] args){
        Widget widget = new LoggingWidget();
        widget.doSomething();
    }

}
```

LoggingWidget.class 字节码：

```java
Constant pool:
   #1 = Methodref          #5.#15         // com/java/widget/Widget."<init>":()V
   #2 = Methodref          #5.#16         // com/java/widget/Widget.doSomething:()V
   #3 = Class              #17            // com/java/widget/LoggingWidget
   #4 = Methodref          #3.#15         // com/java/widget/LoggingWidget."<init>":()V
   #5 = Class              #18            // com/java/widget/Widget
   #6 = Utf8               <init>
   #7 = Utf8               ()V
   #8 = Utf8               Code
   #9 = Utf8               LineNumberTable
  #10 = Utf8               doSomething
  #11 = Utf8               main
  #12 = Utf8               ([Ljava/lang/String;)V
  #13 = Utf8               SourceFile
  #14 = Utf8               LoggingWidget.java
  #15 = NameAndType        #6:#7          // "<init>":()V
  #16 = NameAndType        #10:#7         // doSomething:()V
  #17 = Utf8               com/java/widget/LoggingWidget
  #18 = Utf8               com/java/widget/Widget
{
  public com.java.widget.LoggingWidget();
    descriptor: ()V
    flags: ACC_PUBLIC
    Code:
      stack=1, locals=1, args_size=1
         0: aload_0
         1: invokespecial #1                  // Method com/java/widget/Widget."<init>":()V
         4: return
      LineNumberTable:
        line 9: 0

  public synchronized void doSomething();
    descriptor: ()V
    flags: ACC_PUBLIC, ACC_SYNCHRONIZED
    Code:
      stack=1, locals=1, args_size=1
         0: aload_0
         1: invokespecial #2                  // Method com/java/widget/Widget.doSomething:()V
         4: return
      LineNumberTable:
        line 11: 0
        line 13: 4

  public static void main(java.lang.String[]);
    descriptor: ([Ljava/lang/String;)V
    flags: ACC_PUBLIC, ACC_STATIC
    Code:
      stack=2, locals=2, args_size=1
         0: new           #3                  // class com/java/widget/LoggingWidget
         3: dup
         4: invokespecial #4                  // Method "<init>":()V
         7: astore_1
         8: aload_1
         9: invokevirtual #2                  // Method com/java/widget/Widget.doSomething:()V
        12: return
      LineNumberTable:
        line 16: 0
        line 17: 8
        line 18: 12
}
SourceFile: "LoggingWidget.java"
```

从字节码可知：new 一个 LoggingWidget 对象时，在 LoggingWidget 构造函数中会调用 Widget 的 <init> 实例构造器，正确的初始化父类的状态变量。实际上只是调用父类的实例构造器，不是在子类对象上 new 一个父类对象。

从 Java 程序的视角来看，对象创建才刚刚开始 —— <init> 方法还没有执行，所有的字段都还为零。所以，一般来说（由字节码中是否跟随 invokespecial 指令所决定），执行 new 指令之后会接着执行 <init> 方法，把对象按照程序员的意愿进行初始化，这样一个真正可用的对象才算完全产生出来。

![1571646639535](C:\Users\zhangxiaoliang\Desktop\总结\1571646639535.png)

从以上可知：在创建子类对象时，并非在内部也创建一个父类对象，只是调用父类的实例构造器来正确的初始化对应的父类状态。

## 8.组合与继承

继承和组合都能从现有类型生成新类型，组合一般是将现有类型作为新类型的底层实现的一部分加以复用，而继承复用的是接口。

组合在开发过程中常使用的手段，显示的在新类中放置子对象。组合的语法：只需将对象引用置于新类中即可。组合技术通常用于想在新类中使用现有类的功能而非它的接口的情景。在新类中嵌入某个对象，让其实现所需的功能，但用户看到的只是新类所定义的接口，而非所嵌入对象的接口。

```java
public class TestClass {
    private Animal mAnimal;
}
```


TestClass.class 字节码：

```java
 public com.java.test.TestClass();
    descriptor: ()V
    flags: ACC_PUBLIC
    Code:
      stack=1, locals=1, args_size=1
         0: aload_0
         1: invokespecial #1                  // Method java/lang/Object."<init>":()V
         4: return
      LineNumberTable:
        line 7: 0
```


尽管面向对象编程对继承极力强调，但在开始一个设计时，一般应优先选择使用组合，只有在确实必要时才使用继承，同时组合更具灵活性。

继承涉及到基类和导出类这两个类，而不是只有一个类，但从外部看来，它就像是一个和基类具有相同接口的新类，或许还会有额外的方法和域。但继承并不只是复制基类的接口。当创建了一个导出类的一个对象时，这个子对象和你直接用基类创建的对象是一样的。二者区别在于，后者来至于外部，而基类的子对象被包装在导出类对象内部。

继承与组合应选择哪个？一个最清晰的判断方法：是否需要从新类向基类进行向上转型。如果必须向上转型，则继承是必须的，如果不需要，则应当好好考虑。是否需要继承，只要记得自问一下“我真的需要向上转型吗？”就能较好的在这两种技术中作出决定。组合是一种强耦合关系，你和我都有共同的生命期。
