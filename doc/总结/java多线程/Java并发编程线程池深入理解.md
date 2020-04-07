# Java并发编程:线程池深入理解

-----------------------https://blog.csdn.net/evankaka/article/details/51489322

摘要： 本文主要讲了Java当中的线程池的使用方法、注意事项及其实现源码实现原理，并辅以实例加以说明,对加深Java线程池的理解有很大的帮助。

​     首先，讲讲什么是线程池？照笔者的简单理解，其实就是一组线程实时处理休眠状态，等待唤醒执行。那么为什么要有线程池这个东西呢？可以从以下几个方面来考虑：其一、减少在创建和销毁线程上所花的时间以及系统资源的开销 。其二、是将当前任务与主线程隔离，能实现和主线程的异步执行，特别是很多可以分开重复执行的任务。但是，一味的开线程也不一定能带来性能上的，线池休眠也是要占用一定的内存空间，所以合理的选择线程池的大小也是有一定的依据。

## 一、Executors的API介绍


Java类库提供了许多静态方法来创建一个线程池：

a、newFixedThreadPool 创建一个固定长度的线程池，当到达线程最大数量时，线程池的规模将不再变化。
b、newCachedThreadPool 创建一个可缓存的线程池，如果当前线程池的规模超出了处理需求，将回收空的线程；当需求增加时，会增加线程数量；线程池规模无限制。
c、newSingleThreadPoolExecutor 创建一个单线程的Executor，确保任务对了，串行执行
d、newScheduledThreadPool 创建一个固定长度的线程池，而且以延迟或者定时的方式来执行，类似Timer；

小结一下：在线程池中执行任务比为每个任务分配一个线程优势更多，通过重用现有的线程而不是创建新线程，可以在处理多个请求时分摊线程创建和销毁产生的巨大的开销。当请求到达时，通常工作线程已经存在，提高了响应性；通过配置线程池的大小，可以创建足够多的线程使CPU达到忙碌状态，还可以防止线程太多耗尽计算机的资源。

### 创建线程池基本方法：

(1)定义线程类

 

```java
class Handler implements Runnable{
}
```


(2)建立ExecutorService线程池

 

```java
ExecutorService executorService = Executors.newCachedThreadPool();
```


或者

```Java
int cpuNums = Runtime.getRuntime().availableProcessors();  //获取当前系统的CPU 数目
ExecutorService executorService =Executors.newFixedThreadPool(cpuNums * POOL_SIZE); //ExecutorService通常根据系统资源情况灵活定义线程池大小
```


(3)调用线程池操作

循环操作，成为daemon,把新实例放入Executor池中

```java
  while(true){
    executorService.execute(new Handler(socket)); 
       // class Handler implements Runnable{
    或者
    executorService.execute(createTask(i));
        //private static Runnable createTask(final int taskID)
  }
```
execute(Runnable对象)方法其实就是对Runnable对象调用start()方法（当然还有一些其他后台动作，比如队列，优先级，IDLE timeout，active激活等）

## 二、几种不同的ExecutorService线程池对象



### 1.newCachedThreadPool() 	

-缓存型池子，先查看池中有没有以前建立的线程，如果有，就reuse.如果没有，就建一个新的线程加入池中
-缓存型池子通常用于执行一些生存期很短的异步型任务
 因此在一些面向连接的daemon型SERVER中用得不多。
-能reuse的线程，必须是timeout IDLE内的池中线程，缺省timeout是60s,超过这个IDLE时长，线程实例将被终止及移出池。
  注意，放入CachedThreadPool的线程不必担心其结束，超过TIMEOUT不活动，其会自动被终止。

### 2.newFixedThreadPool	

-newFixedThreadPool与cacheThreadPool差不多，也是能reuse就用，但不能随时建新的线程
-其独特之处:任意时间点，最多只能有固定数目的活动线程存在，此时如果有新的线程要建立，只能放在另外的队列中等待，直到当前的线程中某个线程终止直接被移出池子
-和cacheThreadPool不同，FixedThreadPool没有IDLE机制（可能也有，但既然文档没提，肯定非常长，类似依赖上层的TCP或UDP IDLE机制之类的），所以FixedThreadPool多数针对一些很稳定很固定的正规并发线程，多用于服务器
-从方法的源代码看，cache池和fixed 池调用的是同一个底层池，只不过参数不同:
fixed池线程数固定，并且是0秒IDLE（无IDLE）
cache池线程数支持0-Integer.MAX_VALUE(显然完全没考虑主机的资源承受能力），60秒IDLE  

### 3.ScheduledThreadPool	

-调度型线程池
-这个池子里的线程可以按schedule依次delay执行，或周期执行

### 4.SingleThreadExecutor	

-单例线程，任意时间池中只能有一个线程
-用的是和cache池和fixed池相同的底层池，但线程数目是1-1,0秒IDLE（无IDLE）


应用实例：

1.CachedThreadPool

CachedThreadPool首先会按照需要创建足够多的线程来执行任务(Task)。随着程序执行的过程，有的线程执行完了任务，可以被重新循环使用时，才不再创建新的线程来执行任务。我们采用《Thinking In Java》中的例子来分析。客户端线程和线程池之间会有一个任务队列。当程序要关闭时，你需要注意两件事情：入队的这些任务的情况怎么样了以及正在运行的这个任务执行得如 何了。令人惊讶的是很多开发人员并没能正确地或者有意识地去关闭线程池。正确的方法有两种：一个是让所有的入队任务都执行完毕（shutdown()）， 再就是舍弃这些任务（shutdownNow())——这完全取决于你。比如说如果我们提交了N多任务并且希望等它们都执行完后才返回的话，那么就使用 shutdown()：

 

 * ```java
 import java.util.Date;
import java.util.concurrent.ExecutorService;
	import java.util.concurrent.Executors;
	import java.util.concurrent.ScheduledThreadPoolExecutor;
	import java.util.concurrent.TimeUnit;
	
	/**
	
	 * 功能概要：缓冲线程池实例-execute运行
     * 
     * @author linbingwen
	 * @since  2016年5月24日 
	   */
	   class Handle implements Runnable {
	   private String name;
	   public Handle(String name) {
	   	this.name = "thread"+name;
	   }	
	   @Override
	   public void run() {
	   	System.out.println( name +" Start. Time = "+new Date());
	       processCommand();
	       System.out.println( name +" End. Time = "+new Date());
   }
     private void processCommand() {
            try {
                Thread.sleep(1000);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
     @Override
        public String toString(){
            return this.name;
        }	
    }
 ```
 
 

验证实例：

 

 

```java
public static void testCachedThreadPool() {
	 System.out.println("Main: Starting at: "+ new Date());  
	 ExecutorService exec = Executors.newCachedThreadPool();   //创建一个缓冲池，缓冲池容量大小为Integer.MAX_VALUE
     for(int i = 0; i < 10; i++) {   
            exec.execute(new Handle(String.valueOf(i)));   
     }   
     exec.shutdown();  //执行到此处并不会马上关闭线程池,但之后不能再往线程池中加线程，否则会报错
     System.out.println("Main: Finished all threads at"+ new Date());
}
```

执行结果：

 ![img](C:\Users\zhangxiaoliang\Desktop\总结\java多线程\CacheThreadPool实例结果图.png)



从上面的结果可以看出：

1、主线程的执行与线程池里的线程分开，有可能主线程结束了，但是线程池还在运行

2、放入线程池的线程并不一定会按其放入的先后而顺序执行

 

2.FixedThreadPool
FixedThreadPool模式会使用一个优先固定数目的线程来处理若干数目的任务。规定数目的线程处理所有任务，一旦有线程处理完了任务就会被用来处理新的任务(如果有的话)。这种模式与上面的CachedThreadPool是不同的，CachedThreadPool模式下处理一定数量的任务的线程数目是不确定的。而FixedThreadPool模式下最多 的线程数目是一定的。

应用实例：

 

```java
public static void testFixThreadPool() {
	System.out.println("Main Thread: Starting at: "+ new Date());  
	 ExecutorService exec = Executors.newFixedThreadPool(5);   
     for(int i = 0; i < 10; i++) {   
            exec.execute(new Handle(String.valueOf(i)));   
     }   
     exec.shutdown();  //执行到此处并不会马上关闭线程池
     System.out.println("Main Thread: Finished at:"+ new Date());
}
```

运行结果：

 ![img](C:\Users\zhangxiaoliang\Desktop\总结\java多线程\FixedThreadPool实例结果图)



上面创建了一个固定大小的线程池，大小为5.也就说同一时刻最多只有5个线程能运行。并且线程执行完成后就从线程池中移出。它也不能保证放入的线程能按顺序执行。这要看在等待运行的线程的竞争状态了。

3、newSingleThreadExecutor

其实这个就是创建只能运行一条线程的线程池。它能保证线程的先后顺序执行，并且能保证一条线程执行完成后才开启另一条新的线程

 

	public static void testSingleThreadPool() {
		 System.out.println("Main Thread: Starting at: "+ new Date());  
		 ExecutorService exec = Executors.newSingleThreadExecutor();   //创建大小为1的固定线程池
	     for(int i = 0; i < 10; i++) {   
	            exec.execute(new Handle(String.valueOf(i)));   
	     }   
	     exec.shutdown();  //执行到此处并不会马上关闭线程池
	     System.out.println("Main Thread: Finished at:"+ new Date());
	}

运行结果：

 ![img](C:\Users\zhangxiaoliang\Desktop\总结\java多线程\newSingleThreadExecutor实例结果图)



其实它也等价于以下：

 

 ExecutorService exec = Executors.newFixedThreadPool(1);   






4、newScheduledThreadPool

这是一个计划线程池类，它能设置线程执行的先后间隔及执行时间等，功能比上面的三个强大了一些。

以下实例：

 

```java
public static void testScheduledThreadPool() {
	System.out.println("Main Thread: Starting at: "+ new Date());  
	ScheduledThreadPoolExecutor  exec = (ScheduledThreadPoolExecutor) Executors.newScheduledThreadPool(10);   //创建大小为10的线程池
     for(int i = 0; i < 10; i++) {   
            exec.schedule(new Handle(String.valueOf(i)), 10, TimeUnit.SECONDS);//延迟10秒执行
     }   
     exec.shutdown();  //执行到此处并不会马上关闭线程池
     while(!exec.isTerminated()){
            //wait for all tasks to finish
     }
     System.out.println("Main Thread: Finished at:"+ new Date());
}
```
实现每个放入的线程延迟10秒执行。
结果：

 ![img](C:\Users\zhangxiaoliang\Desktop\总结\java多线程\newScheduledThreadPool实例结果图.png)



ScheduledThreadPoolExecutor的定时方法主要有以下四种：

![img](C:\Users\zhangxiaoliang\Desktop\总结\java多线程\ScheduledThreadPoolExcutor的四种定时方法.png)

下面将主要来具体讲讲scheduleAtFixedRate和scheduleWithFixedDelay

scheduleAtFixedRate 按指定频率周期执行某个任务
public ScheduledFuture<?> scheduleAtFixedRate(Runnable command,
long initialDelay,
long period,
TimeUnit unit);
command：执行线程
initialDelay：初始化延时
period：两次开始执行最小间隔时间
unit：计时单位

scheduleWithFixedDelay 周期定时执行某个任务/按指定频率间隔执行某个任务(注意)
public ScheduledFuture<?> scheduleWithFixedDelay(Runnable command,
long initialDelay,
long delay,
TimeUnit unit);
command：执行线程
initialDelay：初始化延时
period：前一次执行结束到下一次执行开始的间隔时间（间隔执行延迟时间）
unit：计时单位

使用实例：

 

```java
class MyHandle implements Runnable {

	@Override
	public void run() {
		System.out.println(System.currentTimeMillis());
		try {
			Thread.sleep(1 * 1000);
		} catch (InterruptedException e) {
			// TODO Auto-generated catch block
			e.printStackTrace();
		}
	}

}
```

1.按指定频率周期执行某个任务

下面实现每隔2秒执行一次，注意，如果上次的线程还没有执行完成，那么会阻塞下一个线程的执行。即使线程池设置得足够大。

 

```java
/**
 * 初始化延迟0ms开始执行，每隔2000ms重新执行一次任务
 * @author linbingwen
 * @since  2016年6月6日
 */
public static void executeFixedRate() {  
    ScheduledExecutorService executor = Executors.newScheduledThreadPool(10);  
    executor.scheduleAtFixedRate(  
            new MyHandle(),  
            0,  
            2000,  
            TimeUnit.MILLISECONDS);  
}  
```




间隔指的是连续两次任务开始执行的间隔。对于scheduleAtFixedRate方法，当执行任务的时间大于我们指定的间隔时间时，它并不会在指定间隔时开辟一个新的线程并发执行这个任务。而是等待该线程执行完毕。

2、按指定频率间隔执行某个任务

 

```java
/** 
 * 以固定延迟时间进行执行 
 * 本次任务执行完成后，需要延迟设定的延迟时间，才会执行新的任务 
 */  
public static void executeFixedDelay() {  
    ScheduledExecutorService executor = Executors.newScheduledThreadPool(10);  
    executor.scheduleWithFixedDelay(  
            new MyHandle(),  
            0,  
            2000,  
            TimeUnit.MILLISECONDS);  
}  
```


间隔指的是连续上次执行完成和下次开始执行之间的间隔。

3.周期定时执行某个任务

周期性的执行一个任务，可以使用下面方法设定每天在固定时间执行一次任务。

 

```java
/** 
 * 每天晚上9点执行一次 
 * 每天定时安排任务进行执行 
 */  
public static void executeEightAtNightPerDay() {  
    ScheduledExecutorService executor = Executors.newScheduledThreadPool(1);  
    long oneDay = 24 * 60 * 60 * 1000;  
    long initDelay  = getTimeMillis("21:00:00") - System.currentTimeMillis();  
    initDelay = initDelay > 0 ? initDelay : oneDay + initDelay;  
  
    executor.scheduleAtFixedRate(  
            new MyHandle(),  
            initDelay,  
            oneDay,  
            TimeUnit.MILLISECONDS);  
}  
 
/** 
 * 获取指定时间对应的毫秒数 
 * @param time "HH:mm:ss" 
 * @return 
 */  
private static long getTimeMillis(String time) {  
    try {  
        DateFormat dateFormat = new SimpleDateFormat("yy-MM-dd HH:mm:ss");  
        DateFormat dayFormat = new SimpleDateFormat("yy-MM-dd");  
        Date curDate = dateFormat.parse(dayFormat.format(new Date()) + " " + time);  
        return curDate.getTime();  
    } catch (ParseException e) {  
        e.printStackTrace();  
    }  
    return 0;  
} 
```

## 三、线程池一些常用方法

1、submit()

   将线程放入线程池中，除了使用execute，也可以使用submit，它们两个的区别是一个使用有返回值，一个没有返回值。submit的方法很适应于生产者-消费者模式，通过和Future结合一起使用，可以起到如果线程没有返回结果，就阻塞当前线程等待线程 池结果返回。

它主要有三种方法：

一般用第一种比较多

![img](C:\Users\zhangxiaoliang\Desktop\总结\java多线程\submit三种方法.png)

如下实例。注意，submit中的线程要实现接口Callable 

```java
package com.func.axc.executors;

import java.util.ArrayList;
import java.util.List;
import java.util.concurrent.Callable;
import java.util.concurrent.ExecutionException;
import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;
import java.util.concurrent.Future;

/**

 * 功能概要：缓冲线程池实例-submit运行

 * 

 * @author linbingwen

 * @since  2016年5月25日 
   */
   class TaskWithResult implements Callable<String> { 
   private int id; 

   public TaskWithResult(int id) { 
           this.id = id; 
   } 

   /** 

    * 任务的具体过程，一旦任务传给ExecutorService的submit方法，则该方法自动在一个线程上执行。 
    * 
    * @return 
    * @throws Exception 
      */
      public String call() throws Exception { 
          System.out.println("call()方法被自动调用,干活！！！             " + 				  			Thread.currentThread().getName()); 
          //一个模拟耗时的操作
          for (int i = 999999; i > 0; i--) ; 
          return"call()方法被自动调用，任务的结果是：" + id + "    " + 				                   Thread.currentThread().getName(); 
      } 
 }

public class ThreadPool2 {
	  public static void main(String[] args) { 
          ExecutorService executorService = Executors.newCachedThreadPool(); 
          List<Future<String>> resultList = new ArrayList<Future<String>>(); 

          //创建10个任务并执行
          for (int i = 0; i < 10; i++) { 
                  //使用ExecutorService执行Callable类型的任务，并将结果保存在future变量中
                  Future<String> future = executorService.submit(new TaskWithResult(i)); 
                  //将任务执行结果存储到List中
                  resultList.add(future); 
          } 
        //启动一次顺序关闭，执行以前提交的任务，但不接受新任务。如果已经关闭，则调用没有其他作用。
          executorService.shutdown(); 
          
          //遍历任务的结果
          for (Future<String> fs : resultList) { 
                  try { 
                          System.out.println(fs.get());     //打印各个线程（任务）执行的结果
                  } catch (InterruptedException e) { 
                          e.printStackTrace(); 
                  } catch (ExecutionException e) { 
                          e.printStackTrace(); 
                  } finally { 
                          
                  } 
          } 

  } 
}
```

结果如下：



 

从上面可以看到，输出结果的依次的。说明每次get都 阻塞了的。

看了下它的源码，其实它最终还是调用 了execute方法

 

    public <T> Future<T> submit(Callable<T> task) {
        if (task == null) throw new NullPointerException();
        RunnableFuture<T> ftask = newTaskFor(task);
        execute(ftask);
        return ftask;
    }
2、execute()

 

表示往线程池添加线程，有可能会立即运行，也有可能不会。无法预知线程何时开始，何时线束。

主要源码如下：

 

    public void execute(Runnable command) {
        if (command == null)
            throw new NullPointerException();
        if (poolSize >= corePoolSize || !addIfUnderCorePoolSize(command)) {
            if (runState == RUNNING && workQueue.offer(command)) {
                if (runState != RUNNING || poolSize == 0)
                    ensureQueuedTaskHandled(command);
            }
            else if (!addIfUnderMaximumPoolSize(command))
                reject(command); // is shutdown or saturated
        }
    }


3、shutdown()

通常放在execute后面。如果调用 了这个方法，一方面，表明当前线程池已不再接收新添加的线程，新添加的线程会被拒绝执行。另一方面，表明当所有线程执行完毕时，回收线程池的资源。注意，它不会马上关闭线程池！

4、shutdownNow()

不管当前有没有线程在执行，马上关闭线程池！这个方法要小心使用，要不可能会引起系统数据异常！


四、ThreadPoolExecutor技术内幕
经过上面的过程，基本上可以掌握线程池的一些基本用法。下面再来看看JAVA中线程池的源码实现。

首先是其继承关系如下：



通过观察上面四种线程池的源码：

如：newFixedThreadPool

 

    public static ExecutorService newFixedThreadPool(int nThreads) {
        return new ThreadPoolExecutor(nThreads, nThreads,
                                      0L, TimeUnit.MILLISECONDS,
                                      new LinkedBlockingQueue<Runnable>());
    }


如：newCachedThreadPool

 

 

    public static ExecutorService newCachedThreadPool() {
        return new ThreadPoolExecutor(0, Integer.MAX_VALUE,
                                      60L, TimeUnit.SECONDS,
                                      new SynchronousQueue<Runnable>());
    }
如：newSingleThreadExecutor

 

 

    public static ExecutorService newSingleThreadExecutor() {
        return new FinalizableDelegatedExecutorService
            (new ThreadPoolExecutor(1, 1,
                                    0L, TimeUnit.MILLISECONDS,
                                    new LinkedBlockingQueue<Runnable>()));
    }




可以发现，其实它们调用的都是同一个接口ThreadPoolExecutor方法，只不过传入参数不一样而已。下面就来看看这个神秘的ThreadPoolExecutor。
首先来看看它的一些基本参数：

 

public class ThreadPoolExecutor extends AbstractExecutorService {

    //运行状态标志位
    volatile int runState;
    static final int RUNNING    = 0;
    static final int SHUTDOWN   = 1;
    static final int STOP       = 2;
    static final int TERMINATED = 3;
     
    //线程缓冲队列，当线程池线程运行超过一定线程时并满足一定的条件，待运行的线程会放入到这个队列
    private final BlockingQueue<Runnable> workQueue;
    //重入锁，更新核心线程池大小、最大线程池大小时要加锁
    private final ReentrantLock mainLock = new ReentrantLock();
    //重入锁状态
    private final Condition termination = mainLock.newCondition();
    //工作都set集合
    private final HashSet<Worker> workers = new HashSet<Worker>();
    //线程执行完成后在线程池中的缓存时间
    private volatile long  keepAliveTime;
    //核心线程池大小 
    private volatile int   corePoolSize;
    //最大线程池大小 
    private volatile int   maximumPoolSize;
    //当前线程池在运行线程大小 
    private volatile int   poolSize;
    //当缓冲队列也放不下线程时的拒绝策略
    private volatile RejectedExecutionHandler handler;
    //线程工厂，用来创建线程
    private volatile ThreadFactory threadFactory;   
    //用来记录线程池中曾经出现过的最大线程数
    private int largestPoolSize;   
   //用来记录已经执行完毕的任务个数
   private long completedTaskCount;   

    ................
}






初始化线程池大小 有以下四种方法：



从源码中可以看到其实最终都是调用了以下的方法：

 

    public ThreadPoolExecutor(int corePoolSize,
                              int maximumPoolSize,
                              long keepAliveTime,
                              TimeUnit unit,
                              BlockingQueue<Runnable> workQueue,
                              ThreadFactory threadFactory,
                              RejectedExecutionHandler handler) {
        if (corePoolSize < 0 ||
            maximumPoolSize <= 0 ||
            maximumPoolSize < corePoolSize ||
            keepAliveTime < 0)
            throw new IllegalArgumentException();
        if (workQueue == null || threadFactory == null || handler == null)
            throw new NullPointerException();
        this.corePoolSize = corePoolSize;
        this.maximumPoolSize = maximumPoolSize;
        this.workQueue = workQueue;
        this.keepAliveTime = unit.toNanos(keepAliveTime);
        this.threadFactory = threadFactory;
        this.handler = handler;
    }
这里很简单，就是设置一下各个参数，并校验参数是否正确，然后抛出对应的异常。

 

接下来我们来看看最重要的方法execute，其源码如下：

 

    public void execute(Runnable command) {
        if (command == null)
            throw new NullPointerException();
        if (poolSize >= corePoolSize || !addIfUnderCorePoolSize(command)) { //  判断1
            if (runState == RUNNING && workQueue.offer(command)) { // 判断2
                if (runState != RUNNING || poolSize == 0)  //  判断3
                    ensureQueuedTaskHandled(command);
            }
            else if (!addIfUnderMaximumPoolSize(command)) //  判断4
                reject(command); // is shutdown or saturated
        }
    }

笔者在上面加了点注释。下面我们一个一个判断来看

 

首先判断1

 if (poolSize >= corePoolSize || !addIfUnderCorePoolSize(command)) 

（1）当poolSize >= corePoolSize 不成立时，表明当前线程数小于核心线程数目，左边返回fasle.接着执行右边判断!addIfUnderCorePoolSize(command)

它做了如下操作

 

    private boolean addIfUnderCorePoolSize(Runnable firstTask) {
        Thread t = null;
        final ReentrantLock mainLock = this.mainLock;//加锁
        mainLock.lock();
        try {
            if (poolSize < corePoolSize && runState == RUNNING)//线程池在运行且当前线程小于核心线程（外面已做了一次相同的判断，确保和外面的一样）
                t = addThread(firstTask);//加入线程
        } finally {
            mainLock.unlock();
        }
        return t != null;
    }

发现它又调用addTread

 

 

    private Thread addThread(Runnable firstTask) { //调用这个方法之前加锁
        Worker w = new Worker(firstTask);//线程包装成一个work
        Thread t = threadFactory.newThread(w);//线程工厂从work创建线程
        boolean workerStarted = false;
        if (t != null) {
            if (t.isAlive()) // 线程应该是未激活状态
                throw new IllegalThreadStateException();
            w.thread = t;
            workers.add(w);//全局set添加一个work
            int nt = ++poolSize;//当前运行线程数目加1
            if (nt > largestPoolSize)
                largestPoolSize = nt;
            try {
                t.start();//注意，这里线程执行了，但是其实真正调用的是Worker类的run方法！！！！！！！！！
                workerStarted = true;
            }
            finally {
                if (!workerStarted)
                    workers.remove(w);
            }
        }
        return t;
    }
Worker类的run方法！！！！！！！！！
                workerStarted = true;
            }
            finally {
                if (!workerStarted)
                    workers.remove(w);
            }
        }
        return t;
    }

其实Work是真实去调用线程方法的地方,它是对Thread类的一个包装，每次Thread类调用其start方法时，就会调用到work的run方法。其代码如下，

 

 

        private void runTask(Runnable task) { //真正发起线程方法的地方
            final ReentrantLock runLock = this.runLock;
            runLock.lock();
            try {
                if ((runState >= STOP || //判断的判断
                    (Thread.interrupted() && runState >= STOP)) &&
                    hasRun)
                    thread.interrupt();
             
                boolean ran = false;
                beforeExecute(thread, task);//处理前
                try {
                    task.run();//执行真正的原始线程的run方法
                    ran = true;
                    afterExecute(task, null);//处理后
                    ++completedTasks;
                } catch (RuntimeException ex) {
                    if (!ran)
                        afterExecute(task, ex);
                    throw ex;
                }
            } finally {
                runLock.unlock();
            }
        }
     
        //这里执行线程的方法
        public void run() {
            try {
                hasRun = true;
                Runnable task = firstTask;
                firstTask = null;
                while (task != null || (task = getTask()) != null) {
                    runTask(task);
                    task = null;
                }
            } finally {
                workerDone(this);
            }
        }

发现要执行一个线程真的很不容易，如果addIfUnderCorePoolSize返回true，刚表明成功添加一条线程，并调用了其start方法，那么整个调用到此结束。如果返回fasle.那么就进入判断2.

 

（2）当poolSize >= corePoolSize成立时，整个判断返回true。接着执行判断2

判断2

 

if (runState == RUNNING && workQueue.offer(command)) { // 判断2
如果当前线程池在运行状态，并且将当前线程加入到缓冲队列中。workQueue的offer是一个非阻塞方法。如查缓冲队列满了的话，返回为false.否则返回true;

 

如果上面两个都 为true，表明线程被成功添加到缓冲队列中，并且当前线程池在运行。进入判断3

判断3

 

 if (runState != RUNNING || poolSize == 0)
   ensureQueuedTaskHandled(command);

当线程被加入到线程池中，进入判断3.如果这时线程池没有在运行或者运行的线程为为0。那么就调用ensureQueuedTaskHandled，它做的其实是判断下是否在拒绝这个线程的执行。

 

 

    private void ensureQueuedTaskHandled(Runnable command) {
        final ReentrantLock mainLock = this.mainLock;
        mainLock.lock();
        boolean reject = false;
        Thread t = null;
        try {
            int state = runState;
            if (state != RUNNING && workQueue.remove(command)) //线程池没有在运行，且缓冲队列中有这个线程
                reject = true;
            else if (state < STOP &&
                     poolSize < Math.max(corePoolSize, 1) &&
                     !workQueue.isEmpty())
                t = addThread(null);
        } finally {
            mainLock.unlock();
        }
        if (reject)
            reject(command); //根据拒绝策略处理线程
    }

判断4

 

 

 else if (!addIfUnderMaximumPoolSize(command))
                reject(command); // is shutdown or saturated

在判断2为false时执行，表明当前线程池没有在运行或者该线程加入缓冲队列中失败，那么就会尝试再启动下该线程，如果还是失败，那就根据拒绝策略来处理这个线程。其源码如下：

 

 

    private boolean addIfUnderMaximumPoolSize(Runnable firstTask) {
        Thread t = null;
        final ReentrantLock mainLock = this.mainLock;
        mainLock.lock();
        try {
            if (poolSize < maximumPoolSize && runState == RUNNING) //如果当前运行线程数目 小于最大线程池大小 并且 线程池在运行，那么启动该线程
                t = addThread(firstTask);
        } finally {
            mainLock.unlock();
        }
        return t != null;
    }

一般调用这个方法是发生在缓冲队列已满了，那么线程池会尝试直接启动该线程。当然，它要保存当前运行的poolSize一定要小于maximumPoolSize。否则，最后。还是会拒绝这个线程！

 

以上大概就是整个线程池启动一条线程的整体过程。

总结：

       ThreadPoolExecutor中，包含了一个任务缓存队列和若干个执行线程，任务缓存队列是一个大小固定的缓冲区队列，用来缓存待执行的任务，执行线程用来处理待执行的任务。每个待执行的任务，都必须实现Runnable接口，执行线程调用其run()方法，完成相应任务。
ThreadPoolExecutor对象初始化时，不创建任何执行线程，当有新任务进来时，才会创建执行线程。
构造ThreadPoolExecutor对象时，需要配置该对象的核心线程池大小和最大线程池大小：
当目前执行线程的总数小于核心线程大小时，所有新加入的任务，都在新线程中处理
当目前执行线程的总数大于或等于核心线程时，所有新加入的任务，都放入任务缓存队列中
当目前执行线程的总数大于或等于核心线程，并且缓存队列已满，同时此时线程总数小于线程池的最大大小，那么创建新线程，加入线程池中，协助处理新的任务。
当所有线程都在执行，线程池大小已经达到上限，并且缓存队列已满时，就rejectHandler拒绝新的任务

 

五、自定义线程池
再来看看它的方法

 

 

    public ThreadPoolExecutor(int corePoolSize,//核心线程大小
                              int maximumPoolSize,//最大线程大小 
                              long keepAliveTime,//线程缓存时间
                              TimeUnit unit,//前面keepAlive
                              BlockingQueue<Runnable> workQueue,//缓存队列
                              ThreadFactory threadFactory,//线程工大
                              RejectedExecutionHandler handler)//拒绝策略

block queue有以下几种实现：

 

ArrayBlockingQueue : 有界的数组队列
 LinkedBlockingQueue : 可支持有界/无界的队列，使用链表实现
 PriorityBlockingQueue : 优先队列，可以针对任务排序
SynchronousQueue : 队列长度为1的队列，和Array有点区别就是：client thread提交到block queue会是一个阻塞过程，直到有一个worker thread连接上来poll task。当线




当线程池的任务缓存队列已满并且线程池中的线程数目达到maximumPoolSize，如果还有任务到来就会采取任务拒绝策略，通常有以下四种策略

ThreadPoolExecutor.AbortPolicy:丢弃任务并抛出RejectedExecutionException异常。
ThreadPoolExecutor.DiscardPolicy：也是丢弃任务，但是不抛出异常。
ThreadPoolExecutor.DiscardOldestPolicy：丢弃队列最前面的任务，然后重新尝试执行任务（重复此过程）
ThreadPoolExecutor.CallerRunsPolicy：由调用线程处理该任务

比如定义如下一个线程池：

 

 

ThreadPoolExecutor threadPool = new ThreadPoolExecutor(2, 4, 3,TimeUnit.SECONDS, new ArrayBlockingQueue<Runnable>(3),Executors.defaultThreadFactory(),new ThreadPoolExecutor.DiscardOldestPolicy());






这里核心线程数为2，最大线程数为4，线程缓存时间为3秒，缓冲队列的容量设置为3。线程工厂设置为默认

下面是一个具体实例：

 

package com.func.axc.executors;

import java.io.Serializable;
import java.util.concurrent.ArrayBlockingQueue;
import java.util.concurrent.Executors;
import java.util.concurrent.ThreadPoolExecutor;
import java.util.concurrent.TimeUnit;

/**
 * 功能概要：
 * 
 * @author linbingwen
 * @since 2016年6月7日
 */
public class MyThreadPoolTest {

	private static int produceTaskSleepTime = 2;
	private static int produceTaskMaxNumber = 10;

	public static void main(String[] args) {
		// 构造一个线程池
		ThreadPoolExecutor threadPool = new ThreadPoolExecutor(2, 4, 3,TimeUnit.SECONDS, new ArrayBlockingQueue<Runnable>(3),Executors.defaultThreadFactory(),new ThreadPoolExecutor.DiscardOldestPolicy());

		for (int i = 1; i <= produceTaskMaxNumber; i++) {
			try {
				// 产生一个任务，并将其加入到线程池
				String task = "task@ " + i;
				System.out.println("put " + task);
				threadPool.execute(new ThreadPoolTask(task));
				// 便于观察，等待一段时间
				Thread.sleep(produceTaskSleepTime);
			} catch (Exception e) {
				e.printStackTrace();
			}
		}
	}
}

/**
 * 线程池执行的任务
 */
class ThreadPoolTask implements Runnable, Serializable {
	private static final long serialVersionUID = 0;
	private static int consumeTaskSleepTime = 2000;
	// 保存任务所需要的数据
	private Object threadPoolTaskData;

	ThreadPoolTask(Object tasks) {
		this.threadPoolTaskData = tasks;
	}

	public void run() {
		// 处理一个任务，这里的处理方式太简单了，仅仅是一个打印语句
		System.out.println(Thread.currentThread().getName());
		System.out.println("start .." + threadPoolTaskData);

		try {
			// //便于观察，等待一段时间
			Thread.sleep(consumeTaskSleepTime);
		} catch (Exception e) {
			e.printStackTrace();
		}
		threadPoolTaskData = null;
	}

	public Object getTask() {
		return this.threadPoolTaskData;
	}
}















