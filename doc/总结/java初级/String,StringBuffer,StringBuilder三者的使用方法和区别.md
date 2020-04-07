# String,StringBuffer,StringBuilder三者的使用方法和区别


在java中，我们常常用到String类型来操作字符串，但是用来操作字符串变量的不仅仅只有String类型，还有StringBuilder和StringBuffer类型，尽管我们平时使用String类型比较多，但是在实际的开发中，这三种类型完全是三足鼎立的局面，根据不同的使用场景，我们要使用不同的类型，在很多情况下，使用StringBuilder和StringBuffer比使用String类型更快，还会有其它优点。

我们先来了解一下StringBuilder和StringBuffer的简单用法，至于String的用法，我们就不再赘述了，后面拿他们三个作比较的时候我们会提到，因为StringBuffer和StringBuilder的方法基本上都一样，所以我们共同介绍它们的方法

1.StringBuffer，StringBuilder的用法
toString()方法
将StringBuffer，StringBuilder对象转换为String字符串，常用在需要输出的时候，因为StringBuffer和StringBuilder的对象不能直接输出，例如：

StringBuffer s1 = new StringBuffer();
s1.toString();
1
2
append()方法
用于在字符串的后面追加字符串，当StringBuffer,StringBuilder中没有字符串的时候也可以append()，可以用来初始化，例如：

public static void main(String[] args)
    {
        StringBuffer s1 = new StringBuffer().append("bbb");
        s1.append("aaa");
        System.out.println(s1.toString());
    }
1
2
3
4
5
6
运行结果：


charaAt()方法
返回指定索引位置的字符，索引从0开始，例如：

public static void main(String[] args)
    {
        StringBuffer s1 = new StringBuffer().append("bbb");
        s1.append("aaa");
        System.out.println(s1.toString());
        System.out.println(s1.charAt(3));
    }
1
2
3
4
5
6
7
结果：


deleteCharAt()方法
删除指定索引位置的字符，例如：

public static void main(String[] args)
    {
        StringBuffer s1 = new StringBuffer().append("bbb");
        s1.append("aaa");
        System.out.println(s1.toString());
        s1.deleteCharAt(3);
        System.out.println(s1.toString());
    }
1
2
3
4
5
6
7
8
结果：


delete()方法
删除从开始索引到结束索引的字符串，例如：

public static void main(String[] args)
    {
        StringBuffer s1 = new StringBuffer().append("bbb");
        s1.append("aaa");
        System.out.println(s1.toString());
        s1.delete(2,4);
        System.out.println(s1.toString());

    }
1
2
3
4
5
6
7
8
9
结果：


insert()方法
在指定索引位置之前插入字符串，例如：

public static void main(String[] args)
    {
        StringBuffer s1 = new StringBuffer().append("bbb");
        s1.append("aaa");
        System.out.println(s1.toString());
        s1.insert(2,"cc");
        System.out.println(s1.toString());
    }
1
2
3
4
5
6
7
8
结果：


indexOf()方法
返回指定字符串的开始字符索引位置，还可以从某个字符索引位置开始向后匹配，没有找到匹配的就会返回-1,例如：

public static void main(String[] args)
    {
        StringBuffer s1 = new StringBuffer().append("bbb");
        s1.append("aaa");
        System.out.println(s1.toString());
        System.out.println(s1.indexOf("ba"));
        System.out.println(s1.indexOf("ba",2));
        System.out.println(s1.indexOf("mn"));
    }
1
2
3
4
5
6
7
8
9
结果：


lastIndexOf()方法
和indexOf()的用法一样，只不过是从后往前匹配，也支持从指定索引开始从后往前去匹配，例如：

 public static void main(String[] args)
    {
        StringBuffer s1 = new StringBuffer().append("bbb");
        s1.append("aaa");
        System.out.println(s1.toString());
        System.out.println(s1.lastIndexOf("b",5));
    }
1
2
3
4
5
6
7
结果：


reverse()方法
反转字符串，例如：

public static void main(String[] args)
    {
        StringBuffer s1 = new StringBuffer().append("bbb");
        s1.append("aaa");
        System.out.println(s1.toString());
        System.out.println(s1.reverse());
        System.out.println(s1.toString());
        System.out.println(s1.length());
    }
1
2
3
4
5
6
7
8
9
结果：


length()方法
返回字符串的长度，例如：

public static void main(String[] args)
    {
        StringBuffer s1 = new StringBuffer().append("bbb");
        s1.append("aaa");
        System.out.println(s1.toString());
        System.out.println(s1.length());
    }
1
2
3
4
5
6
7
结果：


2.String、StringBuffer、StringBuilder的区别
重头戏来了，前面的部分相信你即使没有在StringBuffer/StringBuilder中用过，也在别的类中用过，那么，这三个类型到底有什么区别呢？怎么选择它们的应用场景呢？

首先，从性能、速度方面来说：
StringBuilder > StringBuffer > String
1
我们来做一个测试，我们分别使用String和StringBuilder创建变量，然后分别对它们进行加字符串操作，由于时间太短，我们把这个过程使用for循环重复100000遍以放大差距：

public static void main(String[] args)
    {

        Long start1 = System.currentTimeMillis();//获取开始时间
        for (int i=0;i<100000;i++)//重复10万次进行String变量加操作
        {
            String str = "a";
            str+="b";
        }
        Long end1 = System.currentTimeMillis();//获取结束时间
        System.out.println("String花费时间："+(end1-start1));//打印出花费的时间
    
        Long start2 = System.currentTimeMillis();
        for (int i=0;i<100000;i++)//重复10万次进行StringBuilder变量加操作
        {
            StringBuilder str2 = new StringBuilder("a");
            str2.append("b");
        }
        Long end2 = System.currentTimeMillis();
        System.out.println("StringBuilder花费时间："+(end2-start2));
    
        Long start3 = System.currentTimeMillis();
        for (int i=0;i<100000;i++)//重复10万次进行StringBuffer变量加操作
        {
            StringBuffer str2 = new StringBuffer("a");
            str2.append("b");
        }
        Long end3 = System.currentTimeMillis();
        System.out.println("StringBuffer花费时间："+(end2-start2));
    }
1
2
3
4
5
6
7
8
9
10
11
12
13
14
15
16
17
18
19
20
21
22
23
24
25
26
27
28
29
30
实验结果：

可以看到，放大了差距之后，String和StringBuilder、StringBuffer两兄弟的差距还是蛮大的，那么是什么造成了这种差距呢？

在上面的程序中：

String str = "a";
str+="b";
1
2
这句话看似是对同一个String类型的str对象进行了加操作，但是实际上可不是同一个对象，事实上，我们先声明了一个String类型的对象，值是"a",把str这个句柄指向了这个对象，然后，当我们把这个对象进行+=操作的时候，实际上是又创建了一个String对象，这个对象的值是"a"+“b"也就是"ab”，然后改变句柄str让它指向了这个新的对象，原来的对象失去了引用，就被jvm垃圾回收了。而StringBuffer和StringBuilder可不是这样，这两兄弟是直接改变自己本身对象的值。
那么，当我们进行了10万次操作的时候，快慢差距自然就体现出来了。

这里有人会问了，如果我把这句代码：

String str = "a";
str+="b";
1
2
改为：

String str = "a"+"b";
1
呢？

让我们看一下执行效果：

哪怕是进行10万次操作，String所花费的时间也是极少的，这是为什么呢？

这是因为String和我们其它类型的变量不同，其它的非基本类型对象的值、数据都存储在java的堆中，而String类型的变量的值是存储jvm在方法区中的字符串常量池中的。当我们执行：String str = “a”+“b”;这句话的时候，String会自动把这个对象的值看成"ab"，然后在方法区中如果找到了值同样为"ab"的，就会直接让str句柄指向它，也就是说，我们的这句代码现在相当于：

String str = "ab";
1
这可比之前的String用法一遍遍反复地去创建对象回收对象快多了，因此，即使重复10万次依然还是很快。

从线程安全的角度去看
StringBuffer是线程安全的，而StringBuilder是线程不安全的，实验的具体方法见此链接：https://blog.csdn.net/litterfrog/article/details/76862435

总结
String适用于少量的字符串操作的情况
StringBuilder适用于单线程下在字符缓冲区进行大量操作的情况
StringBuffer适用多线程下在字符缓冲区进行大量操作的情况