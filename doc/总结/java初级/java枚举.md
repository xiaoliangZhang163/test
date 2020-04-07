# java枚举

## 一、枚举的定义

​       枚举是一种特殊的数据类型，之所以特殊是因为它既是一种类（class）类型却又比类型多了些特殊的约束，但是这些约束的存在也造就了枚举类型的简洁，安全性以及便捷性。创建枚举类型要使用enum关键字，隐含了所创建的类型都是java.lang.Enum类的子类（java.lang.Enum是一个抽象类）。枚举类型符合通用模式Class Enum<E extends Enum<E>>，而E表示枚举类型的名称。枚举类型的每一个值都映射到protected Enum(String name,int ordinal)构造函数中，在这里，每个值的名称都转换成一个字符串，并且序数设置表示了此设置被创建的顺序。

## 二、枚举的使用

​    创建一个枚举类：EnumTest 

```java
public enum EnumTest {
	//星期一，星期二，星期三，星期四，星期五，星期六
	MON(1), TUE(2),WED(3),THU(4),FRI(5),SAT(6){	
		public boolean isRest(){
			return true;
		}
	},
	//星期日
	SUN(0){
		public boolean isRest(){
			return true;
		}
	};
	private int value;
	private  EnumTest(int value){
		this.value=value;
	}
	public int getValue(){
		return value;
	}
	public boolean isRest(){
		return  false;
	}
}
使用EnumTest枚举类：

public class EnumMain {
	public static void main(String[] args) {
		for (EnumTest enumTest : EnumTest.values()) {
			System.out.println(enumTest + ":" + enumTest.getValue());
		}
		System.out.println("---------------我是分割线------------");
		EnumTest test = EnumTest.SAT;
		switch (test) {
		case MON:
			System.out.println("今天是星期一");
			break;
		case TUE:
			System.out.println("今天是星期二");
			break;
		case WED:
			System.out.println("今天是星期三");
			break;
		case THU:
			System.out.println("今天是星期四");
			break;
		case FRI:
			System.out.println("今天是星期五");
			break;
		case SAT:
			System.out.println("今天是星期六");
			break;
		case SUN:
			System.out.println("今天是星期日");
			break;
		default:
			System.out.println(test);
			break;
		}
	}
}
```

结果：

MON:1
TUE:2
WED:3
THU:4
FRI:5
SAT:6
SUN:0
---------------我是分割线------------
今天是星期六

## 三、枚举优缺点

   在枚举出现之前，如果想要表示一组特定的离散值，往往使用一些常量。例如：

 

```java
public class Entity {


public static final int VIDEO = 1;//视频

public static final int AUDIO = 2;//音频

public static final int TEXT = 3;//文字

public static final int IMAGE = 4;//图片


private int id;

private int type;


public int getId() {

return id;

}

public void setId(int id) {

this.id = id;

}

public int getType() {

return type;

}

public void setType(int type) {

this.type = type;

}
```



## 四、使用这个常量的方法的缺点

​      1、代码可读性差，易用性差，由于setType()方法的参数是int型的，

      2、类型不安全。在用户去调用的时候，必须保证类型完全一致，同时取值范围也要正确。像setType(-1);是合法的，但是不是合理的，今后会为程序带来种种问题。
    
       3、耦合性高，扩展性差。假如，因为某些原因，需要修改Entity类中常量的值，那么需要改的时候，修改漏了，那可不是开玩笑的。

枚举就是为了这样的问题而诞生的。它们给出了将一个任意项同另一项比较的能力。