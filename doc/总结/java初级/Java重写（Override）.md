## Java重写（Override）



### 1.重写出现的原因

先看一个例子

```java
class Animal{
    public void printWhoIAm(){System.out.println("Animal")}
}
public class Dog extends Animal{
    public static void main(String args[]){
        Dog dog = new Dog();
        dog.printWhoIAm();
    }
} 
/*
output:
Animal
*/
```

当dog调用printWhoIAm()方法时，其实希望的是输出“dog”，而不是Animal。要实现输出“Dog”，想到了重载，可是重载要求被重载的方法具有不同的形参列表。这个方法不得行。dog调用的printWhoIAm()是父类中的，在子类中若是可以重写这个方法，那么就可以实现目的了。于是，重写（覆写）便产生了，**为了解决父类方法在子类中不适用，而让子类重写方法的方式**。我们解决以上代码需求如下：

```java
class Animal{
    public void printWhoIAm(){System.out.println("Animal")}
}
public class Dog extends Animal{
    //加上注解@Override可以强制进行重写的检查 防止自己重写错误
    @Override
    public void printWhoIAm(){System.out.println("Dog")}
    public static void main(String args[]){
        Dog dog = new Dog();
        dog.printWhoIAm();
    }
} 
/*
output:
Dog
*/
```



### 2.重写的规则

- 重写是对父类允许访问的方法的实现过程进行重新编写，返回值不变或者为子类、形参列表不能改变并且访问控制权限不能严于父类。父类为default包访问权限，则子类就为public或者default；若父类是public，则子类必须为public。
- **子类可以重写父类的除了构造器的任何方法**。构造器是和类名相同的，不能被子类继承，因此也不可以被重写。
- 重写方法不能抛出新的检查异常或者比被重写方法申明更加宽泛的异常。例如： 父类的一个方法申明了一个检查异常 IOException，但是在重写这个方法的时候不能抛出 Exception 异常，因为 Exception 是 IOException 的父类，只能抛出 IOException 的子类异常。



### 3.子类可以重写父类中访问控制权限为private的方法吗？

答案是不可以。父类中的private方法对子类来说是不可见的，就算子类中完全按照重写要求定义方法，也不能算重写父类中的方法，实际上只是子类新增的一个方法。所以，只有非private方法才可以被重写。



### 4.super.方法()和this.方法()的区别

子类若是重写了父类的方法，那么父类原来的这个方法还可以被调用吗？答案是可以的，使用super对父类的方法进行调用。

```java
class Animal{
    public void printWhoIAm(){System.out.println("Animal")}
}
public class Dog extends Animal{
    //加上注解@Override可以强制进行重写的检查 防止自己重写错误
    @Override
    public void printWhoIAm(){System.out.println("Dog")}
    public void print(){
        super.printWhoIAm();
        printWhoIAm();// this.printWhoIAm();
    }
    public static void main(String args[]){
        Dog dog = new Dog();
        dog.print();
    }
} 
/*
output:
Animal
Dog
*/
```

使用`this.方法()`会先在本类中查找是否存在要调用的方法，如果不存在则查找父类中是否具备此方法。如果有则调用，否则出现编译时错误。使用`super.方法()`会明确表示调用父类中的方法，直接去父类寻找要调用的方法。