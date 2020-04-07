# Java 流程控制

## 一、复合语句

　　Java语言的复合语句是以整个块区为单位的语句，又称块语句。复合语句由“{”开始，“}”结束。

　　对于复合语句，我们只需要知道，复合语句为局部变量创建了一个作用域，该作用域为程序的一部分，在该作用域中某个变量被创建并能够被使用，如果在某个变量的作用域外使用该变量，则会发生错误。并且复合语句中可以嵌套复合语句。

## 二、条件语句

　　条件语句可根据不同的条件执行不同的语句。包括if条件语句与switch多分支语句。这是学习Java的一个基础与重点。

### 　　1. if条件语句

　　使用if条件语句，可选择是否要执行紧跟在条件之后的那个语句。关键字if之后是作为条件的“布尔表达式”，如果该表达式返回true，则执行其后的语句；若为false，则不执行if后的语句。可分为简单的if条件语句、if···else语句和if···else if多分支语句。

```
int a = 100;
if(a == 100) {
    System.out.println(a);
}
```

　　如上方代码，｛｝之间为复合语句，if为条件语句，翻译过来就是如果a等于100，则输出a的值，否则不执行。

　　如果if后只有一条语句，比如上述代码只有一条输出，可以不加｛｝，但为了代码的可读性，以及防止代码过多出现不必要的错误，建议所有的if、else后都加上相应的｛｝。

### 　　2. if···else语句

　　if···else语句是条件语句中最常用的一种形式，它会针对某种条件有选择的作出处理。通常表现为“如果满足某种条件，就进行某种处理，否则就进行另一种处理”。

　　if后的()内的表达式必须是boolean型的。如果为true，则执行if后的复合语句；如果为false，则执行else后的复合语句。

　　如：

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
 1 public class Getifelse {
 2 
 3     public static void main(String[] args) {
 4         int math = 80;        // 声明，数学成绩为80（及格）
 5         int english = 50;    // 声明，英语成绩为50（不及格）
 6         
 7         if(math >= 60) {    // if判断语句判断math是否大于等于60
 8             System.out.println("math has passed");
 9         } else {            // if条件不成立
10             System.out.println("math has not passed");
11         }
12         
13         if(english >= 60) {    // if判断语句判断english是否大于等于60
14             System.out.println("english has passed");
15         } else {            // if条件不成立
16             System.out.println("english has not passed");
17         }
18     }
19 
20 }
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

　　运行结果为：

　　![img](https://images2017.cnblogs.com/blog/1018770/201801/1018770-20180122141435444-1528693407.png)

### 　　3. if···else if多分支语句

　　if···else if多分支语句用于针对某一事件的多种情况进行处理。通常表现为“如果满足某种条件”，就进行某种处理，否则，如果满足另一种条件，则进行另一种处理。

　　如：

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
 1 public class GetTerm {
 2 
 3     public static void main(String[] args) {
 4         int x = 40;
 5         
 6         if(x > 60) {
 7             System.out.println("x的值大于60");
 8         } else if (x > 30) {
 9             System.out.println("x的值大于30但小于60");
10         } else if (x > 0) {
11             System.out.println("x的值大于0但小于30");
12         } else {
13             System.out.println("x的值小于等于0");
14         }
15     }
16 
17 }
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

　　在本例中，由于x为40，条件x>60为false，程序向下执行判断；条件x>30为真，所以执行条件x>30后的程序块中的语句。输出结果如下：

　　![img](https://images2017.cnblogs.com/blog/1018770/201801/1018770-20180122142341194-1193261688.png)

　　所以，if语句只执行条件为真的语句，其它语句都不会执行。

### 　　4. switch多分支语句

　　switch语句是一种比较简单明了的多选一的选择，在Java语言中，可以用switch语句将动作组织起来进行多选一。语法格式如下：

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
switch(表达式)
{ 
 case 常量值1:
        语句块1
        [break;]
...
case 常量值n:
        语句块2
        [break;]
default:
        语句块 n+1;
        [break;]
}　　
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

　　switch语句中表达式的值必须是**整型或字符型**，常量值1~n必须也是整型或字符型。

　　简单说一下switch语句是怎么执行的（重点，初学的朋友要注意）。首先switch语句先计算表达式的值，如果表达式的值与case后的常量值相同，则执行该case后的若干个语句，直到遇到break语句为止。如果没有break，则继续执行下一case中的若干语句，直到遇到break为止。若没有一个常量的值与表达式的值相同，则执行default后面的语句。default语句可选，如果不存在default语句，而且switch语句中的表达式的值与任何case的常量值都不相同，则switch不做任何处理。并且，同一个switch语句，case的常量值必须互不相同。

　　例：用switch语句打印出星期的英文单词

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
 1 import java.util.Scanner;
 2 
 3 public class GetSwitch {
 4 
 5     public static void main(String[] args) {
 6         Scanner scan = new Scanner(System.in);
 7         System.out.print("请输入今天星期几：");
 8         int week = scan.nextInt();
 9         
10         switch (week) {
11         case 1:
12             System.out.println("Monday");
13             break;
14         case 2:
15             System.out.println("Tuesday");
16             break;
17         case 3:
18             System.out.println("Wednesday");
19             break;
20         case 4:
21             System.out.println("Thursday");
22             break;
23         case 5:
24             System.out.println("Friday");
25             break;
26         case 6:
27             System.out.println("Saturday");
28             break;
29         case 7:
30             System.out.println("Sunday");
31             break;
32         default:
33             System.out.println("Sorry,I don't konw");
34             break;
35         }
36     }
37 
38 }
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

　　这里Scanner是一个扫描器，用来输入的，使用时需在package下面用import语句导入Scanner类，我们可以在控制台中输入一个数字，然后用nextInt()来接收，这样week的值就是我们输入的数字，接下来执行switch语句，有7个case分别对应周一到周末，default在输入1~7以外的数据时执行。假设在控制台中输入1，则控制台会执行case 1后的语句，输出了Monday，结果如下：

　　![img](https://images2017.cnblogs.com/blog/1018770/201801/1018770-20180122144231475-272236334.png)

　　要注意的是case后的常量表达式的值可以为整数和字符，但不可以是实数后字符串，比如case 1.1，case “ok”都是非法的。

## 三、循环语句

　　循环语句就是在满足一定条件的情况下反复执行某一个操作。包括while循环语句、do···while循环语句和for循环语句。

### 　　1. while循环语句

　　while循环语句的循环方式为利用一个条件来控制是否要继续反复执行这个语句。

　　假设现在有1~10十个数字，我们要将它们相加求和，在学习while之前可能会直接用+运算符从1加到10，也就是1+2+3+4+5+6+7+8+9+10，但如果现在需要从1加到1万呢？10万？所以，我们要引入while循环来进行循环相加，如下：

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
 1 public class GetSum {
 2 
 3     public static void main(String[] args) {
 4         int x = 1;            // 定义初值
 5         int sum = 0;        // 定义求和变量，用于存储相加后的结果
 6         
 7         while(x <= 10) {
 8             sum += x;        // 循环相加，也即    sum = sum + x;
 9             x++;
10         }
11         System.out.println(sum);
12     }
13 
14 }
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

　　这就是一个从1加到10的代码，首先定义一个初值x为1，然后定义一个存储相加结果的变量sum为0，循环条件为x<=10，也就是每次判断x<=10是否成立，成立则继续循环。循环内第一句“sum +=x;”其实就是“sum = sum +x;”的另一种写法，是在sum的基础上加x，并赋给sum，那么此时sum的值为0+1=1了，然后x++，x自增1为2，判断x<=10，则继续循环，sum的值变为1+2=3，然后x++变为3，如此循环下去，直到x为11时退出循环，此时sum的值就是1+2+3+4+5+6+7+8+9+10最后的结果55了。

　　在while循环语句中，如果while语句后直接加分号，如while(a == 5);代表当前while为空语句，进入无线循环。

### 　　2. do···while循环语句

　　do···while循环语句与while循环语句的区别是，while循环语句先判断条件是否成立再执行循环体，而do···while循环语句则先执行一次循环后，再判断条件是否成立。也即do···while至少执行一次。语法格式如下：

```
do
{
    执行语句
}  while (条件表达式);
```

　　下面对while循环语句与do···while循环语句进行一个对比：

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
 1 public class Cycle {
 2 
 3     public static void main(String[] args) {
 4         int a = 10;
 5         int b = 10;
 6         
 7         // while循环语句
 8         while(a == 8) {
 9             System.out.println("a == " + a);
10             a--;
11         }
12         
13         // do···while循环语句
14         do {
15             System.out.println("b == " + b);
16             b--;
17         } while(b == 8);
18     }
19 
20 }
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

　　运行结果为：![img](https://images2017.cnblogs.com/blog/1018770/201801/1018770-20180122152130475-1168030271.png)

　　这里，a、b都为10，先看while循环语句，进入while下语句块的条件是a == 8，很明显不成立，所以不执行，结果中没有关于a的结果，然后再看do···while循环语句，先执行一次do后的语句块，输出“b == 10”，然后判断while条件b == 8不成立，循环结束，所以结果只有一个do···while语句中执行的第一步b == 10。

### 　　3. for循环语句

　　for循环语句是Java程序设计中最有用的循环语句之一。一个for循环可以用来重复执行某条语句，知道某个条件得到满足。语法格式如下：

```
for(表达式1; 表达式2; 表达式3)
{
    语句序列
}
```

　　其中，表达式1为初始化表达式，负责完成变量的初始化；表达式2为循环条件表达式，指定循环条件；表达式3为循环后操作表达式，负责修整变量，改变循环条件。三个表达式间用分号隔开

　　例：用for循环语句求100以内所有偶数的和。

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
 1 public class Circulate {
 2 
 3     public static void main(String[] args) {
 4         int sum = 0;
 5         
 6         for(int i=2; i<=100; i+=2) {
 7             sum += i;
 8         }
 9         
10         System.out.println(sum);
11     }
12 
13 }
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

　　for循环内，首先定义一个变量并赋初值，表示循环中i从2开始进行，然后条件为i<=100，即i<=100时进行循环并执行语句块中的语句，第三个表达式“i+=2”表示每次循环执行i=i+1，即没循环一次，i的值为在原来的基础上加2后的值。然后循环求sum的值，即2+4+6+8+···+100，当i=102时退出循环，执行输出语句，输出结果为2550。

　　说到for循环语句就不得提到foreach语句了，它是Java5后新增的for语句的特殊简化版本，并不能完全替代for语句，但所有foreach语句都可以改写为for语句。foreach语句在遍历数组等时为程序员提供了很大的方便。语法格式如下：

```
for(元素变量x : 遍历对象obj) {
    引用了x的Java语句;
}
```

　　下面举一个例子说明foreach怎么遍历的：

```
int array[] = {7, 8, 9};

for (int arr : array) {
     System.out.println(arr);
}
```

　　array是一个一维数组，其中有7、8、9三个值，现在要将这三个值打印到控制台上，用foreach语句相比for语句会简单很多。其中，在for的条件中，先定义了一个整型变量arr（只要和要遍历的数组名不同即可），冒号后则是要遍历的数组名，那么｛｝间就是要循环的内容了。

## 四、跳转语句

　　Java语言提供了三种跳转语句，分别是break语句、continue语句和return语句。

### 　　1. break语句

　　break语句刚刚在switch中已经见过了，是用来中止case的。实际上break语句在for、while、do···while循环语句中，用于强行退出当前循环，为什么说是当前循环呢，因为break只能跳出离它最近的那个循环的循环体，假设有两个循环嵌套使用，break用在内层循环下，则break只能跳出内层循环，如下：

```
for(int i=0; i<n; i++) {    // 外层循环
    for(int j=0; j<n ;j++) {    // 内层循环
        break;
    }
}
```

### 　　2. continue语句

　　continue语句只能用于for、while、do···while循环语句中，用于让程序直接跳过其后面的语句，进行下一次循环。

　　例：输出10以内的所有奇数

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```java
 1 public class ContinueDemo {
 2 
 3     public static void main(String[] args) {
 4         int i = 0;
 5         
 6         while(i < 10) {
 7             i++;
 8             
 9             if(i%2 == 0) {    // 能被2整除，是偶数
10                 continue;    // 跳过当前循环
11             }
12             
13             System.out.print(i + " ");    
14         }
15     }
16 
17 }
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

　　这里if条件判断是否为偶数，如果是偶数则执行continue，直接跳出本次循环，不进行continue后的步骤（即不执行输出语句），然后下一次循环为奇数，输出i，运行结果如下：

　　![img](https://images2017.cnblogs.com/blog/1018770/201801/1018770-20180122155834444-755538791.png)

### 　　3. return语句

　　return语句可以从一个方法返回，并把控制权交给调用它的语句。

```
public void getName() {
    return name;
}
```

　　例如上方代码，这是一个方法用于获取姓名，当调用这个方法时将返回姓名值。