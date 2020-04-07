# util包

java.util包含集合框架，旧集合类，事件模型，日期和时间设施，国际化和其他实用程序类（字符串tokenizer，随机数生成器和位数组）。

## 一. 集合

### 集合的概念：

　　是一种工具类，可以存储任意数量、任意类型的对象（所以后面需要用到泛型，以约束集合中元素的类型）

### 集合的作用：

　　1、在类的内部对属性进行组织

　　2、方便快速定位属性位置

　　3、某些集合接口，提供了一系列排列有序的元素，可以在序列中快速插入或删除

　　4、某些集合接口，提供了映射关系，可以通过关键字（key）快速查找到对应的唯一对象，而这个key可以是任意类型

### 集合与数组的差别：

　　1、数组长度固定，集合长度可变

　　2、数组只能通过下标访问具体元素，集合则可通过任意类型查找所映射的具体对象

 

### Java集合框架体系结构：

根接口——子接口——实现类（并未全部列出）

![img](C:\Users\zhangxiaoliang\Desktop\总结\java常用类库\Java集合框架体系结构.png)

 

**List接口及其常用实现类——ArrayList**

　　1、List是元素有序并且可以重复的集合，称为序列

　　2、List可以精确插入或删除某个位置的元素

　　3、ArrayList——数组序列，底层是由数组实现的

ArrayList实现的方法详见Java API文档：http://tool.oschina.net/apidocs/apidoc?api=jdk-zh

注意：对象存入集合都会变成Object类型，取出时需要进行类型转换

 

**泛型：**

　　集合中的元素可以是任意类型的对象，如果把某个对象放入集合，则会忽略他的类型，当作Object处理

　　1、泛型集合中，不能添加泛型规定的类型及其子类型以外的对象，编译期间会进行类型检查

　　2、泛型使用for each方法遍历集合时，不需要用Object，直接使用原类型即可

　　3、泛型集合中的限定类型不能使用基本数据类型，可以通过使用包装类限定允许存入的基本数据类型

 

Set接口及其实现类——HashSet

　　1、Set是元素无序并且不可以重复的集合，称为“集”

　　2、HashSet——哈希集

　　3、Set中，同一个对象无论添加多少次，只有第一次会添加生效

　　4、Set中可以添加null

 

 

Map

　　Map提供了一种映射关系，其中的元素是以键值对（key-value）的形式存储的，能够根据key查找value，key和value可以是任意类型的对象

　　key和value属于Entry类的对象实例

　　key值不能重复

　　一个value可以对应多个key，一个key只能对应一个value

　　Map的泛型：Map<K， V>　　 //K为key值的类型，V为value值的类型

 

HashMap

　　HashMap中的Entry对象是无序排列的

　　key值和value值可以为null，但是只能有一个key为null，因为key不可重复

Collection


集合：是java提供的一种容器，可以用来存储多个数据


集合和数组都是容器，那么他们的区别是什么？

数组长度是固定的。集合的长度是可变的。

int[ ] arr = new int[10]

Student[ ] arr = new Student[3]

List list = new ArrayList<>();

数组中存储的是同一类型的元素，可以存储基本数据类型值，集合存储的都是对象。而且对象的类型可以不一致。在开发中一般当对象多的时候，使用集合存储。

注意：集合不能存储基本数据类型，比如：存储的是1,2等数字，其实存储的是Integer对象

### Collection

Collection接口是集合层次结构中的根节点，Collection接口继承自超级接口Iterable,是Collection层次结构中的根接口。Collection表示一组对象，这些对象也被称为Collection的元素。一些Collection允许有重复的元素（例如List），但是另一些则不允许有重复的元素，即为无序的（如Set）。JDK不提供此接口的任何直接实现—它会提供更为具体的子接口(如Set和List),。此接口用来传递Collection,并在需要时最大普遍性的地方操作这些Collection。

注意：因为Collection继承了Iterable接口，所以实现Collection的类可以通过迭代器遍历，如：list的子类，set的子类，但是Map就不可以，若使用迭代器遍历，可以一边遍历一边通过迭代器对象删除元素。

Collection是所有单列集合的父接口，因此Collection中定义单列集合List和Set通用的一些方法，这些方法可用于操作所有单列集合

Collection提供了一些通用的方法：

返回值	    方法名	                                              描述
boolean	add(E e)	集合添加元素
boolean	addAll(Collection<? extends E> c)	将指定集合的全部元素添加到新的集合中
void	       clear()	                                                删除集合中全部元素
boolean	contains(Object o)	                          集合中是否包含某个元素
boolean	containsAll(Collection<?> c)	          判断集合中是否包含指定集合中的全部元素
boolean	equals(Object o)	                             比较两个对象是否相等
boolean	isEmpty()	                                          判断集合是否为空
Iterator	 iterator()	                                           返回此集合中元素的迭代器
boolean	remove(Object o)	                           从集合中移除指定元素
boolean	removeAll(Collection<?> c)	           从集合中移除指定集合中的所有元素
int	          size()	                                                返回集合中元素的数量
Object[ ]	toArray()	                                         将集合中的元素，存储到数组中

#### Iterator

Collection继承了Iterable接口，那么就像上面提到了只要是实现了Collection接口的集合都可以使用迭代器，迭代器就是循环遍历集合中元素。查看Iterable接口jdk中只有一句：实现此接口允许对象成为“for-each loop”语句的目标。其实for-eache循环底层还是迭代器。

如何获取迭代器？

 我们发现Collection接口中提供了一个方法Iterator()返回值就是一个Iterator，它的泛型与集合的泛型一致。

迭代器Iterator常用方法：

返回值	   方法名			  描述
boolean	hasNext()		是否有更多元素
E				next()			    获取下一个元素
void		   remove()		 移除元素
注意：集合在for循环中边遍历边删除的情况必须使用迭代器循环实现，如果使用for循环边遍历边删除元素那么就会报ConcurrentModificationException异常

增强for循环底层仍然是迭代器

#### 集合的比较

##### 子接口比较

List	                            Set	                              Map<k,v>
Collection的子接口	Collection的子接口	
1.有序，有下标
2.元素可重复	          1.无序
2.元素不可重复	1.键值对一一对应
2.key不可重复，value可以重复
List和Set都继承了Collection接口，都是是单列集合，Map是双列集合。

Map集合中有一些操作键值对的方法：

put(key, value) 向map中添加元素

get(key) 通过key获取value的值

keySet() 获取所有key的Set集合

values() 获取Collection即value的集合

entrySet() 返回值是键值对的set集合Set<Map.Entry<k,v>>

java.util.Map.Entry<K,V>接口,就是键值对，我们常用于遍历map集合时可以通过entrySet获取到键值对的集合Map.Entry<k,v>,再通过接口提供的两个方法分别获取键值对的key和value，即getKey() 和 getValue()

#### List实现类比较

ArrayList	LinkedList	Vector	Queue
底层是数组结构：查询快，增删慢
因为数组有下标，且是连续的，所以查询速度较快
查看ArrayList集合底层源码发现，每次添加 元素都是数组复制的过程，每次添加一个元素都会创建一个比原数组长度大一的新数组，将原有数组的元素复制到新数组中，并添加新元素，所以ArrayList增删慢	底层双向链表结构：查询速度较慢，但是增删快；
因为LinkedList实现了Deque接口，所以可以模拟队列(先进先出)，因此有offer()入列，poll()出列，也可以模拟堆栈(先进后出)，因此有压栈push(),弹栈pop()，当然因为LinkedList集合是双向列表那么就有一些操作列表首尾元素的方法，比如：addFrist()，addLast()，getFirst()，getLast()等方法	1.0版本单列表集合
类似ArrayList，但是因为线程安全，效率低，被ArrayList替代	1,0版本，继承了Vector栈：特点：后进先出，常用的方法就是pop()出栈也叫弹栈，即删除顶部对象，push()压栈，将对象添加到
多线程，不安全，效率高	多线程，不安全，效率高	单线程，安全，效率低	单线程，线程安全，效率低
建议：如果程序中查询比较多，那么使用ArrayList

#### Set实现类比较

哈希值：是一个十进制的整数，由系统随机给出（就是对象的地址值，是一个逻辑地址，是模拟出来的地址，不是数据实际存储的物理地址）

在Object类有一个方法，可以获取对象哈希值——int hashCode 返回对象的哈希码值

hashCode方法的源码：public native int hashCode(); 其中native代表该方法调用的是本地操作系统的方法

#### 哈希表：



注意：如果同一个哈希值下面挂载的对象超过8个就将此哈希值下的对象转换成红黑树。哈希表=数组+链表或者数组+红黑树

HashSet	LinkedHashSet	TreeSet
底层是哈希表结构：查询速度快
特点：元素不重复，且没有顺序，可能存取顺序不同	底层是哈希表+链表
特点：元素不重复，保存了元素的存储顺序	底层是二叉树
特点：可以排序
实体类实现Comparable接口重写compareTo方法；
接口Comparator自定义比较器
线程不安全，效率高	线程不安全，效率高	线程不安全，效率高
HashSet去重原理：首先比较元素的hashCode()哈希值是否存在，如果不存在，那么将元素存储，如果存在，再通过equals()方法表元素内容是否一致，如果一致不存储，不一致则存储。HashSet底层使用的HashMap，只取了hashMap的key。

注意：jdk提供的数据类型String，Integer等类都重写了hashCode和equals方法，所以当这些数据存入HashSet集合中时都会去重，但是如果存储自定的类型的数据需要去重的话，就必须重写hashCode()和equals()方法。比如：hashSet集合中要存储Student对象（名字和年龄都相同就是同一个人），且需要去重，那么就需要Student类重写hashCode和equals方法。

如下图描述：


LinkedHashSet继承了HashSet，只是底层结构不同，LinkedHashSet底层结构为哈希表+链表，链表记录了元素的存储顺序，保证元素有序，LinkedHashSet去重原理与HashSet相同。

#### TreeSet的两种排序和去重方法

方法一：Comparable

TreeSet会对存储的元素按照自然顺序排序，也就是存入TreeSet的元素必须实现Comparable这个接口，我们常用的数据类型如：String，Integer，Date等都实现了此接口，所以如果我们要存储自定义的实体，那么该实体必须实现Comparable接口。

什么是自然顺序？
	java.lang.Comparable<T>  public interface Comparable<T>该接口对实现它的每个类的对象强加一个整体排序。 这个排序被称为类的自然排序 ，类的compareTo方法被称为其自然比较方法 。 因为String实现了接口Comparable<T>，那么String中的compareTo方法按照字典顺序比较字符串，基于字符串中每个字符的Unicode值； 因为Integer实现了Comparable<T>接口，那么Integer中的compareTo方法比较的是Integer对象，按照数值从小到大排序。
	接口Comparable<T>中有一个方法compareTo(T o)，将此对象(即调用compareTo方法的对象)与指定的对象(即o)进行比较以进行排序。返回一个负整数，零或正整数，因为该对象小于，等于或大于指定对象。 我们一般分别返回-1，0, 1。 
如果没有实现那么就会报异常：java.lang.ClassCastException: com.xrds.springbootdemo.bean.PersonBean cannot be cast to java.lang.Comparable 例如：

//PersonBean没有实现Comparable接口   
public static void main(String[] args) {
        PersonBean p1 = new PersonBean("张三",20);
        PersonBean p2 = new PersonBean("李四",25);
        PersonBean p3 = new PersonBean("王五",30);
        PersonBean p4 = new PersonBean("王五",30);

        Set<PersonBean> treeSet = new TreeSet<>();
        treeSet.add(p1);
        treeSet.add(p2);
        treeSet.add(p3);
        treeSet.add(p4);
        System.out.println(treeSet.size());
    }
结果：
Exception in thread "main" java.lang.ClassCastException: com.xrds.springbootdemo.bean.PersonBean cannot be cast to java.lang.Comparable
	at java.util.TreeMap.compare(TreeMap.java:1294)
	at java.util.TreeMap.put(TreeMap.java:538)
	at java.util.TreeSet.add(TreeSet.java:255)
	at com.xrds.springbootdemo.business.collection.MyTreeSet.main(MyTreeSet.java:25)
Process finished with exit code 1
自定义实体类实现了Comparable接口，那么就需要重写compareTo方法，可以根据实体类的属性进行排序和去重。例如学生实体类通过年龄从小到大排序和去重。

    @Override
    public int compareTo(Student o) {
        if (this.age > o.age){
            return 1;
        }else if (this.age < o.age)){
            return  -1;
        }
        return 0;
    }

方法二：Comparator

TreeSet不仅可以通过实体类实现Comparable接口实现排序和去重，也可以自定义比较器，即TreeSet有参构造函数TreeSet(Comparator<? super E> comparator) 参数就是Comparator接口的实现类，可以创建一个Comparator的实现类，也可以通过匿名内部类实现排序和去重。例如：

java.util.Comparator<T>接口，有比较功能，对一些对象的集合施加了一个整体排序 。可以将比较器传递给排序方法（如Collections.sort或Arrays.sort ），以便对排序顺序进行精确控制。
1
//通过人物的年龄进行从小到大排序和去重
public static void main(String[] args) {
        PersonBean p1 = new PersonBean("张三",20);
        PersonBean p2 = new PersonBean("李四",25);
        PersonBean p3 = new PersonBean("王五",30);
        PersonBean p4 = new PersonBean("王五",30);
//匿名内部类
        Set<PersonBean> treeSet = new TreeSet<>(new Comparator<PersonBean>() {
            @Override
            public int compare(PersonBean o1, PersonBean o2) {
                if (o1.getAge() > o2.getAge()){
                    return 1;
                }else if (o1.getAge()  < o2.getAge()){
                    return -1;
                }
                return 0;
            }
        });
        treeSet.add(p1);
        treeSet.add(p2);
        treeSet.add(p3);
        treeSet.add(p4);
        for (PersonBean p:treeSet){
            System.out.println(p);
        }
    }

注意：TreeSet存储元素类型必须一致，即使泛型设置为Object，虽然add可以将元素存储，但是运行会报异常：java.lang.ClassCastException: java.lang.Integer cannot be cast to java.lang.String

    public static void main(String[] args) {
        TreeSet<Object> treeSet = new TreeSet<>();
        treeSet.add(1);
        treeSet.add("a");
        treeSet.add("c");
        treeSet.add("1");
        treeSet.add("3");
        System.out.println(treeSet.size());
    }
结果：
Exception in thread "main" java.lang.ClassCastException: java.lang.Integer cannot be cast to java.lang.String
	at java.lang.String.compareTo(String.java:111)
	at java.util.TreeMap.put(TreeMap.java:568)
	at java.util.TreeSet.add(TreeSet.java:255)
	at com.xrds.springbootdemo.business.collection.MyTreeSet1.main(MyTreeSet1.java:19)

注意：Map/Set的key为自定义对象时，必须重写hashCode和equals。 关于hashCode和equals的处理，遵循如下规则：

只要重写equals，就必须重写hashCode。
因为Set存储的是不重复的对象，依据hashCode和equals进行判断，所以Set存储的对象必须重写这两个方法。
如果自定义对象做为Map的键，那么必须重写hashCode和equals。
Map实现类比较
Map实现类的比较类似Set实现类比较

HashMap	TreeMap	LinkedHashMap	HashTable
底层是哈希表：查询速度快	底层是红黑树	底层是哈希表+链表	被HashMap取代
key无序且不重复	key排序和去重	key有序	key无序且不重复
多线程，不安全，效率高	多线程，不安全，效率高	多线程，不安全，效率高	线程安全，效率低
HashMap的key去重原理与HashSet相同，都是通过hashCode()方法和equals()方法；

LinkedHashMap继承了HashMap所以key的去重原理和HashMap相同。

TreeMap的key去重原理与TreeSet相同，都是通过比较器排序和去重。

HashTable被HashMap取代，因为HashTable是线程安全的效率低。

注意：上面我们曾经说过，HashSet的底层是通过HashMap实现的，HashSet只是用了HashMap的key集合，所以自定义的实体类如果要作为Map集合的key，那么就需要重写hashCode()方法和equals()方法，对Map集合的key进行去重。当然TreeMap的key排序和去重与TreeSet类似。

#### Collections

我们发现Collections和Collection非常相似，Collection接口是集合层次结构中的根节点，Collections是包装器，就是集合的一个工具类，发现Collections中的方法都是static静态的。

##### Collections的属性

返回值	       属性名	             描述
static List	  EMPTY_LIST	   空list集合对象
static Map	EMPTY_MAP	  空map集合对象
static Set	  EMPTY_SET	    空set集合对象
这三个属性与常用方法中emptyList()、emptyMap()和emptySet()效果一样。

我们在程序中经常为了防止出现空指针异常NullPointerException，会对集合进行判null操作，可能会通过条件语句进行判断，但是程序中条件语句增多的话，会提高代码的复杂度。

```java
//我们发现这段代码我们有两次判空 
public static void main(String[] args) {
        List<String> list = getList();
        if (CollectionUtils.isEmpty(list)){
            System.out.println("list集合是空的");
            return;
        }
        for (String s:list){
            System.out.println(s);
        } 
    }



private static  List<String> getList(){
    List<String> list = StudentDao.selectStudentList();
    if (CollectionUtils.isEmpty(list)){
        return null;
    }
    //TODO对list进行一番操作,然后返回最终的list
    return list;
}
```


其实我们可以减少判空次数，如下：

```java
public static void main(String[] args) {
        List<String> list = getList();
        for (String s:list){
            System.out.println(s);
        } 
    }

private static  List<String> getList(){
    List<String> list = StudentDao.selectStudentList();
    if (CollectionUtils.isEmpty(list)){
        return Collections.EMPTY_LIST;
    }
    //TODO对list进行一番操作,然后返回最终的list
    return list;
}
```



##### 常用方法：

binarySearch(List<? extends Comparable<? super T>> list, T key) 通过二分查找方式从集合中查询key的下标

binarySearch(List<? extends T> list, T key, Comparator<? super T> c)

 以上两个方法类似，二分查找必须集合中的元素是升序排序，否则结果有误，方法二在查询前需要对指定列表进行升序排序。

copy(List<? super T> dest, List<? extends T> src) 将一个集合(src)中所有元素复制到目标集合中(dest)

reverse(List<?> list) 将集合中的元素顺序反转排序

sort(List list) 集合中所有元素必须实现Comparable接口，实现升序排序

sort(List list, Comparator<? super T> c) 根据指定的比较器引起的顺序对指定的列表进行排序。

 我们常用的创建集合的方式，创建的都是线程不安全的集合，比如：new ArrayList<>()，Collections提供了一些方法可以创建线程安全的集合。如下：

synchronizedList(List list) ，synchronizedMap(Map<K,V> m) ，synchronizedSet(Set s)

#### 数组

Arrays
java.util.Arrays类，该类是操作数组的一个工具类，该类包含用于操作数组的各种方法（如排序和搜索）。 该类还包含一个静态工厂，可以将数组视为列表。 如果指定的数组引用为空，则该类中的方法都抛出一个NullPointerException。Arrays类似Collections，其方法都是static静态类。

##### 常用的方法：

asList(T… a) 将数组转换成List集合，例如：List list = Arrays.asList(“a”, “b”, “c”) 或者 List list = Arrays.asList(new String[ ]{“a”, “b”, “c”})

sort(byte[ ] a) 和 sort(byte[] a, int fromIndex, int toIndex)

 对数组进行排序，基本数据类型按照数字顺序升序排序，自定义数据类型类似TreeSet排序方式，通过接口Comparable和接口Comparator，方法二可以指定排序范围

binarySearch(byte[ ] a, byte[ ] key) 和 binarySearch(char[] a, int fromIndex, int toIndex, char key)

 以上两个方法是通过二分法查询数组，数组类型不仅仅只有byte还有char，double，int等基本数据类型，以及Object指定对象数组，其中第二个方法可以指定从数组起始下标到结束下标进行二分法查询，值得注意的是使用二分法查询必须排序。

copyOf(byte[ ] a, int newLength) 和 copyOfRange(byte[ ] int from, int to)

 以上两个方法是复制数组，方法一将指定数组，从下标零开始复制，长度为newLength，如果数组长度小于newLength，那么用零填充，方法二将指定数组复制，并且指定复制的范围

当然Arrays也提供了equals()方法比较数组是否相同，toString()方法将指定数组转成字符串等方法

## 二. 时间

### 2.1 Date

关于时间的首先是日期Date

常用的方法：

compareTo(Date anotherDate) 比较两个日期

equals() 比较两个日期是否相同

getTime() 获取时间的毫秒数

关于时间我们常做的就是日期格式转成，之间通过代码学习

将日期Date转换成字符串

```java
public static void main(String[] args) throws ParseException {
    //获取当前时间
    Date date = new Date();
    System.out.println("当前时间：" + date);
    //当前时间转成毫秒
    System.out.println("当前时间毫秒："+date.getTime());

    //将当前时间根据指定格式转成字符串形式
    SimpleDateFormat simpleDateFormat1 = new SimpleDateFormat();
    String format1 = simpleDateFormat1.format(date);
    System.out.println("当前时间默认字符串格式："+format1);

    SimpleDateFormat simpleDateFormat = new SimpleDateFormat("yyyy-MM-dd HH:mm:ss");
    String format = simpleDateFormat.format(date);
    System.out.println("当前时间指定格式：" + format);

    //将字符串转成Date类型,注意：字符串格式必须与SimpleDateFormat指定日期格式一致
    String date1 = "2019-04-14";
    String date2 = "2019-04-15";
    SimpleDateFormat simpleDateFormat2 = new SimpleDateFormat("yyyy-MM-dd");
    Date parse1 = simpleDateFormat2.parse(date1);
    Date parse2 = simpleDateFormat2.parse(date2);
    System.out.println(parse1);
    System.out.println(parse2);

    //通过毫秒数比较两个日期
    if (parse1.getTime() > parse2.getTime()){
        System.out.println(date1+"比"+date2+"晚");
    }else {
        System.out.println(date1+"比"+date2+"早");

    }
}
```
结果：
当前时间：Sun Apr 14 15:00:35 CST 2019
当前时间毫秒：1555225235266
当前时间默认字符串格式：19-4-14 下午3:00
当前时间指定格式：2019-04-14 15:00:35
Sun Apr 14 00:00:00 CST 2019
Mon Apr 15 00:00:00 CST 2019
2019-04-14比2019-04-15早

### 2.2 Calendar

Calendar即日历，java.util.Calendar类是一个抽象类，所以不能通过new来创建对象，而且通过getInstance()方法创建一个Calendar对象。通过Calendar操作时间更加灵活，我们可以通过代码学习Calendar。

```java
public static void main(String[] args) {
    Calendar instance = Calendar.getInstance();
    System.out.println(instance);

    //获取当前时间，相当于Date date = new Date();
    Date time = instance.getTime();
    System.out.println("当前时间："+time);

    //将日期Date转成Calendar
    instance.setTime(new Date());
    System.out.println("当前时间Calendar："+instance);

    System.out.println("=====================================");
    //注意：设置日期是月份需要减1
    instance.set(2019,4-1,14);
    System.out.println("自定义时间年月日："+instance.getTime());

    //通过set(int field, int value)给指定的属性赋值，我们常用的属性
    instance.set(Calendar.YEAR,2019);
    instance.set(Calendar.MONTH,4-1);//Calendar.MONTH实际月份比设置月份大1，即设置的是4，实际输出的是5月份
    instance.set(Calendar.DATE,2);
    instance.set(Calendar.HOUR_OF_DAY,23);//Calendar.HOUR实际时间比设置的时间超出12小时，即如果设置12，实际输出的日期是第二天的0点
    instance.set(Calendar.MINUTE,59);
    instance.set(Calendar.SECOND,59);
    instance.set(Calendar.WEEK_OF_MONTH,4);
    SimpleDateFormat simpleDateFormat = new SimpleDateFormat("yyyy-MM-dd HH:mm:ss");
    String format = simpleDateFormat.format(instance.getTime());
    System.out.println("分别自定义年月日时分秒："+format);

    System.out.println("=====================================");
    //通过field获取对应value值
    Calendar instance2 = Calendar.getInstance();
    System.out.println("当前时间年份："+instance2.get(Calendar.YEAR));
    System.out.println("当前时间月份："+instance2.get(Calendar.MONTH)+1);//获取当前月份需要加1，即Calendar.MONTH获得月份加1才是本月月份
    System.out.println("当前时间日期："+instance2.get(Calendar.DATE));
    System.out.println("当前时间小时："+instance2.get(Calendar.HOUR_OF_DAY));
    System.out.println("当前时间分钟："+instance2.get(Calendar.MINUTE));
    System.out.println("当前时间秒："+instance2.get(Calendar.SECOND));
    System.out.println("当前时间周几："+instance2.get(Calendar.DAY_OF_WEEK));//获取周几，注意：周日是1，周一是2 。。。
    System.out.println("当前时间是本月第几周："+instance2.get(Calendar.WEEK_OF_MONTH));//获取本月第几周
    System.out.println("当前时间是本年第几周："+instance2.get(Calendar.WEEK_OF_YEAR));//获取本年第几周
```


```java
    System.out.println("=====================================");
    //两个日期比较，如果小于0，则调用对象比指定对象早，反之晚，等于0则两个时间相同
    Calendar instance1 = Calendar.getInstance();
    instance1.set(2019,4-1,18);
    int i = instance.compareTo(instance1);
    if (i > 0){
        System.out.println(instance.getTime()+"比"+instance1.getTime()+"晚");
    }else if(i < 0){
        System.out.println(instance.getTime()+"比"+instance1.getTime()+"早");
    }else {
        System.out.println(instance.getTime()+"和"+instance1.getTime()+"相等");

    }

    System.out.println("=====================================");
    //add方法是根据日历的规则，将指定的时间量添加或减去给定的日历字段。
    Calendar instance3 = Calendar.getInstance();
    instance3.add(Calendar.MONTH,2);//当前时间两个月之后的时间
    instance3.add(Calendar.DATE,1);//与上一行代码合起来就是当前时间两个月另一天之后的时间
    System.out.println(instance3.getTime());
}
```
结果：
java.util.GregorianCalendar[time=1555230036582,areFieldsSet=true,areAllFieldsSet=true,lenient=true,zone=sun.util.calendar.ZoneInfo[id="Asia/Shanghai",offset=28800000,dstSavings=0,useDaylight=false,transitions=19,lastRule=null],firstDayOfWeek=1,minimalDaysInFirstWeek=1,ERA=1,YEAR=2019,MONTH=3,WEEK_OF_YEAR=16,WEEK_OF_MONTH=3,DAY_OF_MONTH=14,DAY_OF_YEAR=104,DAY_OF_WEEK=1,DAY_OF_WEEK_IN_MONTH=2,AM_PM=1,HOUR=4,HOUR_OF_DAY=16,MINUTE=20,SECOND=36,MILLISECOND=582,ZONE_OFFSET=28800000,DST_OFFSET=0]
当前时间：Sun Apr 14 16:20:36 CST 2019

当前时间Calendar：java.util.GregorianCalendar[time=1555230036614,areFieldsSet=true,areAllFieldsSet=true,lenient=true,zone=sun.util.calendar.ZoneInfo[id="Asia/Shanghai",offset=28800000,dstSavings=0,useDaylight=false,transitions=19,lastRule=null],firstDayOfWeek=1,minimalDaysInFirstWeek=1,ERA=1,YEAR=2019,MONTH=3,WEEK_OF_YEAR=16,WEEK_OF_MONTH=3,DAY_OF_MONTH=14,DAY_OF_YEAR=104,DAY_OF_WEEK=1,DAY_OF_WEEK_IN_MONTH=2,AM_PM=1,HOUR=4,HOUR_OF_DAY=16,MINUTE=20,SECOND=36,MILLISECOND=614,ZONE_OFFSET=28800000,DST_OFFSET=0]

自定义时间年月日：Sun Apr 14 16:20:36 CST 2019

分别自定义年月日时分秒：2019-04-21 23:59:59

当前时间年份：2019
当前时间月份：31
当前时间日期：14
当前时间小时：16
当前时间分钟：20
当前时间秒：36
当前时间周几：1
当前时间是本月第几周：3

当前时间是本年第几周：16

Sun Apr 21 23:59:59 CST 2019比Thu Apr 18 16:20:36 CST 2019晚

Sat Jun 15 16:20:36 CST 2019
当然还有一些方法比如：

获取当前月份的最大日期：getActualMaximum(Calendar.DATE) 即方法instance.getActualMaximum(int field);

获取当前时区：getTimeZone()

在jdk1.8新特性中添加了一些关于时间的类：LocalDate，LocalTime等

## 三. UUID

java.util.UUID类一个表示不可变的通用唯一标识符（UUID）的类。 UUID表示128位值。

我们常用UUID获取32数字，比如：常见的数据库主键

```java
public static void main(String[] args) {
    String s = UUID.randomUUID().toString().replace("-","");
    System.out.println(s);
}
```
结果：0a523fff8cf545d29e9ea1238ee57e19