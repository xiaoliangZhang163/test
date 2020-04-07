# Java并发编程ThreadFactory、ThreadLocal

--------------------------https://blog.csdn.net/evankaka/article/details/51705661

一、ThreadFactory
1.1 源码解读
ThreadFactory这个故名思义，就是一个线程工厂。用来创建线程。这里为什么要使用线程工厂呢？其实就是为了统一在创建线程时设置一些参数，如是否守护线程。线程一些特性等，如优先级。通过这个TreadFactory创建出来的线程能保证有相同的特性。下面来看看它的源码吧

它首先是一个接口类，而且方法只有一个。就是创建一个线程。

public interface ThreadFactory {

    Thread newThread(Runnable r);
}
在JDK中，有实现ThreadFactory就只有一个地方。而更多的时候，我们都是继承它然后自己来写这个线程工厂的。
下面的代码中在类Executors当中。默认的 我们创建线程池时使用的就是这个线程工厂

    static class DefaultThreadFactory implements ThreadFactory {
        private static final AtomicInteger poolNumber = new AtomicInteger(1);//原子类，线程池编号
        private final ThreadGroup group;//线程组
        private final AtomicInteger threadNumber = new AtomicInteger(1);//线程数目
        private final String namePrefix;//为每个创建的线程添加的前缀
     
        DefaultThreadFactory() {
            SecurityManager s = System.getSecurityManager();
            group = (s != null) ? s.getThreadGroup() :
                                  Thread.currentThread().getThreadGroup();//取得线程组
            namePrefix = "pool-" +
                          poolNumber.getAndIncrement() +
                         "-thread-";
        }
     
        public Thread newThread(Runnable r) {
            Thread t = new Thread(group, r,
                                  namePrefix + threadNumber.getAndIncrement(),
                                  0);//真正创建线程的地方，设置了线程的线程组及线程名
            if (t.isDaemon())
                t.setDaemon(false);
            if (t.getPriority() != Thread.NORM_PRIORITY)//默认是正常优先级
                t.setPriority(Thread.NORM_PRIORITY);
            return t;
        }
    }

在上面的代码中，可以看到线程池中默认的线程工厂实现是很简单的，它做的事就是统一给线程池中的线程设置线程group、统一的线程前缀名。以及统一的优先级。
1.2 应用实例
下面来看看自己写的一个线程工厂

package com.func.axc.threadfactory;

import java.util.ArrayList;
import java.util.Date;
import java.util.Iterator;
import java.util.List;
import java.util.concurrent.ThreadFactory;

/**
 * 功能概要：
 * 
 * @author linbingwen
 * @since 2016年6月18日
 */
public class ThreadFactoryTest {

	static class MyThreadFactory implements ThreadFactory {

		private int counter;
		private String name;
		private List<String> stats;
	 
		public MyThreadFactory(String name) {
			counter = 0;
			this.name = name;
			stats = new ArrayList<String>();
		}
	 
		@Override
		public Thread newThread(Runnable run) {
			Thread t = new Thread(run, name + "-Thread-" + counter);
			counter++;
			stats.add(String.format("Created thread %d with name %s on%s\n",t.getId(), t.getName(), new Date()));
			return t;
		}
	 
		public String getStas() {
			StringBuffer buffer = new StringBuffer();
			Iterator<String> it = stats.iterator();
			while (it.hasNext()) {
				buffer.append(it.next());
				buffer.append("\n");
			}
			return buffer.toString();
		}

	}
	
	static class MyTask implements Runnable {
		
		private int num;
		
		public MyTask(int num) {
			this.num = num;
		}
	 
		@Override
		public void run() {
			System.out.println("Task "+ num+" is running");
			try {
				Thread.sleep(2*10000);
			} catch (InterruptedException e) {
				// TODO Auto-generated catch block
				e.printStackTrace();
			}
			
		}
	
	}

	/**
	 * @author linbingwen
	 * @since 2016年6月18日
	 * @param args
	 */
	public static void main(String[] args) {
		System.out.println("main thread beging");
		MyThreadFactory factory = new MyThreadFactory("MyThreadFactory");  

        Thread thread = null;  
        for(int i = 0; i < 10; i++) {  
            thread = factory.newThread(new MyTask(i));  
            thread.start();  
        }  
        System.out.printf("Factory stats:\n");  
        System.out.printf("%s\n",factory.getStas());  
    	System.out.println("main thread end");
	}

}
输出结果：


这里通过线程工厂统一设置了线程前缀名，并将创建的线程放到一个list当中。

二、ThreadLocal源码与应用
 2.1 应用实例      
     其实这个和上面的ThreadFactoy基本没什么关联。ThreadFactory与ThreadGroup还有点关联。ThreadLocal基本上和这两个没什么联系的，但是在高并发场景，如果只考虑线程安全而不考虑延迟性、数据共享的话，那么使用ThreadLocal会是一个非常不错的选择。

     当使用ThreadLocal维护变量时，ThreadLocal为每个使用该变量的线程提供独立的变量副本，所以每一个线程都可以独立地改变自己的副本，而不会影响其它线程所对应的副本。

应用实例：

package com.func.axc.threadlocal;

import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;

/**
 * 功能概要：
 * 
 * @author linbingwen
 * @since  2016年6月18日 
 */
public class ThreadLocalTest {
	
    //创建一个Integer型的线程本地变量
	 static final ThreadLocal<Integer> local = new ThreadLocal<Integer>() {
		@Override
		protected Integer initialValue() {
			return 0;
		}
	};
	
	static class Task implements Runnable{
		private int num;
		
		public Task(int num) {
			this.num = num;
		}
	 
		@Override
		public void run() {
			//获取当前线程的本地变量，然后累加10次
			Integer i = local.get();
			while(++i<10);
			System.out.println("Task " + num + "local num resutl is " + i);
		}
	}
	
	static void Test1(){
		System.out.println("main thread begin");
		ExecutorService executors = Executors.newCachedThreadPool();
		for(int i =1;i<=5;i++) {
			executors.execute(new Task(i));
		}
		executors.shutdown();
		System.out.println("main thread end");
	}
	
	public static void main(String[] args){
		Test1();
	}

}
输出结果：
可以看到各个线程之间的变量是独门的，不会相影响。



通过上面的一个实例，简单的了解了ThreadLocal的用法，下面再来看下其源码实现。

2.2 源码解析
首先是其包含的方法：



它的构造函数不做什么：

    public ThreadLocal() {
    }
其实主要的也就下面几个方法：

public T get() { }
public void set(T value) { }
public void remove() { }
protected T initialValue() { }
（1）get
    public T get() {
        Thread t = Thread.currentThread();
        ThreadLocalMap map = getMap(t);
        if (map != null) {
            ThreadLocalMap.Entry e = map.getEntry(this);
            if (e != null) {
                @SuppressWarnings("unchecked")
                T result = (T)e.value;
                return result;
            }
        }
        return setInitialValue();
    }

这个上方法就是用来取得变量的副本的，注意到它先取得了当前线程对象，接下来使用了getMap返回一个ThreadLocalMap
    ThreadLocalMap getMap(Thread t) {
        return t.threadLocals;
    }

然后可以知道ThreadLocalMap这个竟然是从线程中取到的，好，再打开线程类看看
发现Thread类中有这样一个变量：

    ThreadLocal.ThreadLocalMap threadLocals = null;
也变是说每一个线程都有自己一个ThreadLocalMap。
在我们第一次调用get()函数 时，getMap函数返回的是一个null的map.接着就调用setInitialValue()

看看setInitialValue，它才是真正去初始化map的地方！

    private T setInitialValue() {
        T value = initialValue();
        Thread t = Thread.currentThread();
        ThreadLocalMap map = getMap(t);
        if (map != null)
            map.set(this, value);
        else
            createMap(t, value);
        return value;
    }

其中initialValue这个方法就是我们要重写的，一般我们在这里通过一个new 方法返回一个新的变量实例
    protected T initialValue() {
        return null;
    }
因为是第一次调用get()，所以getMap后的map还是为null。这时就调用到createMap

    void createMap(Thread t, T firstValue) {
        t.threadLocals = new ThreadLocalMap(this, firstValue);
    }
终于创建ThreadLocalMap！
        ThreadLocalMap(ThreadLocal<?> firstKey, Object firstValue) {
            table = new Entry[INITIAL_CAPACITY];
            int i = firstKey.threadLocalHashCode & (INITIAL_CAPACITY - 1);
            table[i] = new Entry(firstKey, firstValue);
            size = 1;
            setThreshold(INITIAL_CAPACITY);
        }
这里就将Thread和我们的ThreadLocal通过一个map关联起来。意思是每个Thread中都有一个ThreadLocal.ThreadLocalMap。其中Key为ThreadLocal这个实例，value为每次initialValue（）得到的变量！

接下来如果我们第二次调用get()函数，这里就会进入if方法中去！

    public T get() {
        Thread t = Thread.currentThread();
        ThreadLocalMap map = getMap(t);
        if (map != null) {
            ThreadLocalMap.Entry e = map.getEntry(this);
            if (e != null) {
                @SuppressWarnings("unchecked")
                T result = (T)e.value;
                return result;
            }
        }
        return setInitialValue();
    }
进入If方法中后。就会根据当前的thradLocal实例为Key，取得thread中对应map的vale.其中getEntry方法只是取得我们的key-value对。注意，这时的table其实就是在ThreadLocal实例中都会记录着每个和它关联的Thread类中的ThreadLocalMap变量
        private Entry getEntry(ThreadLocal<?> key) {
            int i = key.threadLocalHashCode & (table.length - 1);
            Entry e = table[i];
            if (e != null && e.get() == key)
                return e;
            else
                return getEntryAfterMiss(key, i, e);
        }
是取得我们的key-value对之后就可取value了，然后就是返回result.如果这时取不到entry，那么又会调用到setInitalValue()方法，过程又和上面的一样了。这里就不说了！
（2）set

这个方法就是重新设置每一个线程的本地ThreadLocal变量的值

    public void set(T value) {
        Thread t = Thread.currentThread();
        ThreadLocalMap map = getMap(t);
        if (map != null)
            map.set(this, value);
        else
            createMap(t, value);
    }
这里取得当前线程，然后根据当前的thradLocal实例取得其map。然后重新设置 map.set(this, value);这时这个线程的thradLocal里的变量副本就被重新设置值了！
（3）remove

就是清空ThreadLocalMap里的value，这样一来。下次再调用get时又会调用 到initialValue这个方法返回设置的初始值

     public void remove() {
         ThreadLocalMap m = getMap(Thread.currentThread());
         if (m != null)
             m.remove(this);
     }

总结：

1、每个线程都有自己的局部变量
每个线程都有一个独立于其他线程的上下文来保存这个变量，一个线程的本地变量对其他线程是不可见的（有前提，后面解释）
2、独立于变量的初始化副本
ThreadLocal可以给一个初始值，而每个线程都会获得这个初始化值的一个副本，这样才能保证不同的线程都有一份拷贝。
3、状态与某一个线程相关联
ThreadLocal 不是用于解决共享变量的问题的，不是为了协调线程同步而存在，而是为了方便每个线程处理自己的状态而引入的一个机制，理解这点对正确使用ThreadLocal至关重要。