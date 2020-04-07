# Java Lambda表达式的使用

 我们可以在任意函数式接口上使用 @FunctionalInterface 注解，这样做可以检查它是否是一个函数式接口，同时 javadoc 也会包含一条声明，说明这个接口是一个函数式接口。像上面的Consumer接口就是一个函数式接口。
Java内置的四大函数式接口分别是：

![image-20191024151251569](C:\Users\zhangxiaoliang\Desktop\总结\image-函数式接口)



```java
@Test
    public void test3() {
          //这个e就代表所实现的接口的方法的参数，
       Consumer<String> consumer = e->System.out.println("ghijhkhi"+e);
       consumer.accept("woojopj");
    }


@Test    
	public void test6() {        
    	Supplier<String> supplier = ()->"532323".substring(0, 2);
        System.out.println(supplier.get());
    }   
@Test    
public void test7() {       
    Function<String, String> function = (x)->x.substring(0, 2);        		    System.out.println(function.apply("我是中国人"));    
}    
@Test    
public void test8() {        
    Predicate<String> predicate = (x)->x.length()>5;            			    System.out.println(predicate.test("12345678"));        System.out.println(predicate.test("123"));    
} 
```

![image-20191024151805937](C:\Users\zhangxiaoliang\Desktop\总结\image-方法引用)