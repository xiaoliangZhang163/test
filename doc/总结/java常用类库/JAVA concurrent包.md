# concurrent包

----------- https://blog.csdn.net/weixin_43915808/article/details/95871021 

java.util.concurrent包中包含了并发编程需要的接口和类

为了学习的流畅性，所以将lang包中的关于线程的常用类在这里介绍。

## 一.  线程类型

### 1.1 接口：Runnable

接口Runnable是函数式接口，只有一个方法run()，且通过注解@FunctionalInterface注入

```java
@FunctionalInterface
public interface Runnable {
    /**

 * When an object implementing interface <code>Runnable</code> is used

   * to create a thread, starting the thread causes the object's

   * <code>run</code> method to be called in that separately executing

     * thread.

     * <p>
       * The general contract of the method <code>run</code> is that it may
       * take any action whatsoever.
       *
       * @see     java.lang.Thread#run()
       */
        public abstract void run();

}
```

### 1.2 类：Thread

小知识：进程和线程

进程：是执行中一段程序，即一旦程序被载入到内存中并准备执行，它就是一个进程。进程是表示资源分配的的基本概念，又是调度运行的基本单位，是系统中的并发执行的单位。

线程：单个进程中执行中每个任务就是一个线程。线程是进程中执行运算的最小单位

### 1.3 两种创建线程的方式

​	(其实还有通过使用Callable和Future创建线程、通过使用线程池例如用Executor框架)

#### 1.3.1 方法一、通过实现Runnable接口创建并启动线程

首先需要定义Runnable接口的实现类，重写run()方法，run()方法就是线程的执行体
其次创建Runnable的实现类的实例，并用这个实例作为Thread的target创建Thread对象
通过Thread对象调用start()方法启动线程

```java
public class MyThread implements Runnable {  
    @Override
    public void run() {
        System.out.println("线程执行");
    }
}
public class TestMyThread {
    public static void main(String[] args) {
        MyThread myThread = new MyThread();
        Thread thread = new Thread(myThread);
        thread.start();
        System.out.println("线程已经启动");
    }
}
```

#### 1.3.2 方法二、通过继承Thread类来创建并启动线程

定义Thread的子类，重写run()方法，执行线程体
创建Thread子类的实例
通过实例启动线程

```java 
public class MyThread extends Thread {

    public static void main(String[] args) {
        MyThread myThread = new MyThread();
        myThread.start();
        System.out.println("线程已经启动");
    }
    
    @Override
    public void run(){
        System.out.println("线程开始执行");
    }

}
```

### 1.4 线程的状态：Thread.State

线程状态。 线程可以处于以下状态之一：

NEW （new）尚未启动的线程处于此状态。
RUNNABLE （runnable）在Java虚拟机中执行的线程处于此状态。
BLOCKED （blocked）被阻塞等待监视器锁定的线程处于此状态。
WAITING （waiting）正在等待另一个线程执行特定动作的线程处于此状态。
TIMED_WAITING （timed_waiting）正在等待另一个线程执行动作达到指定等待时间的线程处于此状态。
TERMINATED (terminated) 已退出的线程处于此状态。
如下：

```java
public class MyThread extends Thread {

    public static void main(String[] args){
        MyThread myThread = new MyThread();
        System.out.println("线程状态0："+myThread.getState());
        myThread.start();
        System.out.println("线程已经启动");
        System.out.println("线程状态1："+myThread.getState());
    }
    
    @Override
    public void run(){
        System.out.println("线程开始执行");
    }

}
```

线程状态0：NEW
线程已经启动
线程状态1：RUNNABLE
线程开始执行

## 二. 线程生命周期

### 2.1 线程生命周期

线程的生命周期就是线程的5种状态：

 新建(New)、就绪(Runnable)、运行(Running)、阻塞(Blocked)、死亡(Dead)

新建状态：当程序使用new 创建一个线程，那么该线程就处于新建状态，仅由JVM为其分配内存，并初始化其成员变量；

就绪状态：当线程对象调用了start()方法，该线程就进入了就绪状态，随时可以被cpu调用。

运行状态：如果被cpu调用那么就进入了运行状态，若执行顺利就会到达死亡状态，当然也可能被阻塞

阻塞状态：如果线程在执行过程中sleep(),wait(),IO阻塞，那么就进入到了阻塞状态，若是睡眠，当睡眠时间到了结束睡眠，若是等待，可以通过notify()结束等待，或者知道IO阻塞解除，结束阻塞状态后线程回到就绪状态等待调用

死亡状态：当线程被使用完后就死亡

![](C:\Users\zhangxiaoliang\Desktop\总结\java常用类库\线程状态转换图.png)

注意：线程从阻塞状态只能进入就绪状态，无法直接进入运行状态。而就绪和运行状态之间的转换通常不受程序控制，而是由系统线程调度所决定。当处于就绪状态的线程获得处理器资源时，该线程进入运行状态；当处于运行状态的线程失去处理器资源时，该线程进入就绪状态。但有一个方法例外，调用yield()方法可以让运行状态的线程转入就绪状态。关于yield()方法后面有更详细的介纽。

### 2.2 线程常用方法

返回值	                                          方法	                                                      功能
static Thread	                       currentThread()	                          返回对当前正在执行的线程对象的引用。
long	                                       getId()	                                          返回此线程的标识符。
String	                                    getName()	                                  返回此线程的名称。
int											getPriority()	                                 返回此线程的优先级。
Thread.State	                      getState()	                                     返回此线程的状态。
ThreadGroup	                     getThreadGroup()	                       返回此线程所属的线程组。
void	                                     interrupt()	                                   中断这个线程。
static boolean	                   interrupted()	                               测试当前线程是否中断。
boolean	                                 sAlive()	                                      测试这个线程是否活着
void	                                       join()	                                          等待这个线程死亡。
void	                                  join(long millis)	                             等待这个线程死亡最多 millis毫秒。
void	                                  run()	                                             如果这个线程使用单独的Runnable运行对象构造，                                                    则调用该Runnable对象 的run方法; 否则，此方法不执行任何操作并返回。
void	                                 setName()	                                      将此线程的名称更改为等于参数 name
void	                               setPriority(int new Priority)	           更改此线程的优先级(1-10)
static void	                         sleep(long millis)	                          使当前正在执行的线程以指定的毫秒数暂停（暂时停止执行），具体取决于系统定时器和调度程序的精度和准确性。
void	                                   start()	                                  导致此线程开始执行; Java虚拟机调用此线程的run方法。
属性		
static int						MAX_PRIORITY							线程可以拥有的最大优先级。
static int						MIN_PRIORITY							线程可以拥有的最小优先级。
static int						NORM_PRIORITY						分配给线程的默认优先级。

## 三. 线程池

我们一般使用线程的时候就会创建一个线程，至于创建线程的方式在学习lang包的时候就介绍了，创建线程的方式可以通过实现Runnable，或者继承Thread，那么如果我们遇到并发时可能需要多个线程同时执行，若每次都创建一个新的线程，用完之后再销毁，这样会大大降低系统效率，因为创建线程和销毁线程需要时间。

我们可以通过线程池来解决，线程池就是一个存放线程的容器，在线程池中创建多个线程，有任务执行就到线程池中获取线程，用完之后归还线程到线程池中，后面的任务就可以使用空闲的线程执行。

![](C:%5CUsers%5Czhangxiaoliang%5CDesktop%5C%E6%80%BB%E7%BB%93%5Cjava%E5%B8%B8%E7%94%A8%E7%B1%BB%E5%BA%93%5CJAVA%20concurrent%E5%8C%85.assets%5C%E7%BA%BF%E7%A8%8B%E6%B1%A0.png)

合理利用线程池的好处：

降低资源消耗。减少创建和销毁线程的次数，每个工作线程都可以重复利用，可执行多个任务。
提高响应速度。当任务到达时，任务可以不需要等到线程创建就可以立即执行。
提高线程的可管理性。可以根据系统承受能力，调整线程池中工作线程的数目，防止因为消耗过多的内存，而把服务器累趴(每个线程需要大约1MB内存，线程开的越多，消耗内存越多)

### 创建线程池

#### Executor & ExecutorService

 		Executor框架便是Java 5中引入的，其内部使用了线程池机制，它在java.util.concurrent 包下，通过该框架来控制线程的启动、执行和关闭，可以简化并发编程的操作。因此，在Java 5之后，通过Executor来启动线程比使用Thread的start方法更好，除了更易管理，效率更好（用线程池实现，节约开销）外，还有关键的一点：有助于避免this逃逸问题——如果我们在构造器中启动一个线程，因为另一个任务可能会在构造器结束之前开始执行，此时可能会访问到初始化了一半的对象用Executor在构造器中。Eexecutor作为灵活且强大的异步执行框架，其支持多种不同类型的任务执行策略，提供了一种标准的方法将任务的提交过程和执行过程解耦开发，基于生产者-消费者模式，其提交任务的线程相当于生产者，执行任务的线程相当于消费者，并用Runnable来表示任务，Executor的实现还提供了对生命周期的支持，以及统计信息收集，应用程序管理机制和性能监视等机制。

Executor框架包括：线程池，Executor，Executors，ExecutorService，CompletionService，Future，Callable等。

 **Executor：**一个接口，其定义了一个接收Runnable对象的方法executor，其方法签名为executor(Runnable command),该方法接收一个Runnable实例，它用来执行一个任务，任务即一个实现了Runnable接口的类，一般来说，Runnable任务开辟在新线程中的使用方法为：new Thread(new RunnableTask())).start()，但在Executor中，可以使用Executor而不用显示地创建线程：executor.execute(new RunnableTask()); // 异步执行

 **ExecutorService：**ExecutorService继承了Executor接口，是一个比Executor使用更广泛的子类接口，其提供了生命周期管理的方法，返回 Future 对象，以及可跟踪一个或多个异步任务执行状况返回Future的方法；可以调用ExecutorService的shutdown（）方法来平滑地关闭 ExecutorService，调用该方法后，将导致ExecutorService停止接受任何新的任务且等待已经提交的任务执行完成(已经提交的任务会分两类：一类是已经在执行的，另一类是还没有开始执行的)，当所有已经提交的任务执行完毕后将会关闭ExecutorService。因此我们一般用该接口来实现和管理多线程。

通过 ExecutorService.submit() 方法返回的 Future 对象，可以调用isDone（）方法查询Future是否已经完成。当任务完成时，它具有一个结果，你可以调用get()方法来获取该结果。你也可以不用isDone（）进行检查就直接调用get()获取结果，在这种情况下，get()将阻塞，直至结果准备就绪，还可以取消任务的执行。Future 提供了 cancel() 方法用来取消执行 pending 中的任务。

ExecutorService的生命周期包括三种状态：运行、关闭、终止。创建后便进入运行状态，当调用了shutdown（）方法时，便进入关闭状态，此时意味着ExecutorService不再接受新的任务，但它还在执行已经提交了的任务，当有已经提交了的任务执行完后，便到达终止状态。如果不调用shutdown（）方法，ExecutorService会一直处在运行状态，不断接收新的任务，执行新的任务，服务器端一般不需要关闭它，保持一直运行即可。

在Java 5之后，任务分两类：一类是实现了Runnable接口的类，一类是实现了Callable接口的类。两者都可以被ExecutorService执行，但是Runnable任务没有返回值，而Callable任务有返回值。并且Callable的call()方法只能通过ExecutorService的submit(Callable task) 方法来执行，并且返回一个 Future，是表示任务等待完成的 Future。

java.util.concurrent interface Exector接口只有一个方法：

execute(Runnable command) 调用线程

java.util.concurrent interface ExecutorService 常用方法：

isShutdown() 判断ExecutorService是否已经关闭，如果关闭返回true

isTerminated() 判断所有任务在关闭后是否完成，如果完成返回true

shutdown() 启动有序关闭，已经提交的任务继续执行直到任务完成，但是不接受新的任务

submit(Callable task) 返回任务处理结果future,执行的任务是Callable

submit(Runnable task) 返回任务处理结果future，执行的任务是Runnable

#### interface ScheduledExecutorService

ScheduledExecutorService接口继承了ExecutorService接口，该接口提供了一些方法可以使线程延迟执行或者定时执行

schedule方法创建具有各种延迟的任务，并返回可用于取消或检查执行的任务对象。 scheduleAtFixedRate和scheduleWithFixedDelay方法创建并执行定期运行的任务，直到取消。

常用方法：

scheduleAtFixedRate(Runnable command, long initialDelay, long period, TimeUnit unit)

API描述：创建并执行在给定的初始延迟之后，随后以给定的时间段首先启用的周期性动作; 那就是执行将在initialDelay之后开始，然后initialDelay+period ，然后是initialDelay + 2 * period ，等等。 如果任务的执行遇到异常，则后续的执行被抑制。 否则，任务将仅通过取消或终止执行人终止。 如果任务执行时间比其周期长，则后续执行可能会迟到，但不会同时执行。

 创建并执行Runnable线程任务，按照设置的时间延迟，周期性执行任务，方法参数分别为:

 Runnable command —— 要执行的线程任务

 long initialDelay —— 第一次执行延迟时间

 long period —— 任务执行间隔时间

 TimeUnit unit —— long initialDelay和long period的时间单位

//创建一个线程池，设置线程数量为1
ScheduledExecutorService scheduledExecutorService = Executors.newScheduledThreadPool(1);

//开始执行线程，设置第一次延迟时间为0，即不延迟，设置任务执行间隔为1s，时间单位为秒
scheduledExecutorService.scheduleAtFixedRate(new PollingThread(this), 0, 1, TimeUnit.SECONDS);

ScheduledFuture<?> scheduleWithFixedDelay(Runnable command, long initialDelay, long delay, TimeUnit unit)

API描述：创建并执行在给定的初始延迟之后首先启用的定期动作，随后在一个执行的终止和下一个执行的开始之间给定的延迟。 如果任务的执行遇到异常，则后续的执行被抑制。 否则，任务将仅通过取消或终止执行人终止。

 创建并执行Runnable线程任务，按照设置的时间延迟，方法参数分别为：

 Runnable command —— 要执行的线程任务

 long initialDelay —— 第一次执行延迟时间

 long delay —— 前一个任务执行终止与下一个任务执行开始之间的延迟时间

 TimeUnit unit —— long initialDelay和long delay 的时间单位

schedule(Callable callable, long delay, TimeUnit unit) 和 schedule(Runnable command, long delay, TimeUnit unit)

 以上两个方法类似就是创建并执行任务，设置每次任务延迟时间，方法参数分别为：

 Runnable command —— 要执行的线程任务

 long delay —— 从现在开始延迟执行的时间

 TimeUnit unit —— long initialDelay和long delay 的时间单位、

#### interface ThreadFactory

根据需要创建新线程的对象。 使用线程工厂可以删除new Thread的硬连线 ，使应用程序能够使用特殊的线程子类，优先级等。

它有一个方法：new Thread(Runnable r) 返回Thread构建一个新的Thread

#### class Executors

 java.util.concurrent.Executors 主要提供线程池相关操作。

defaultThreadFactory() 返回ThreadFactory 用于创建新线程的默认线程工程

**java通过Executors提供四种线程池**

newCachedThreadPool创建一个可缓存线程池，如果线程池长度超过处理需要，可灵活回收空闲线程，若无可回收，则新建线程。

newCachedThreadPool() 返回ExecutorService，创建一个线程池，如果没有空闲线程执行任务，就会创建新线程

newCachedThreadPool(ThreadFactory threadFactory) 使用该方法可以通过ThreadFactory创建线程

newFixedThreadPool 创建一个定长线程池，可控制线程最大并发数，超出的线程会在队列中等待。

newFixedThreadPool(int nThreads) 返回ExecutorService，创建一个线程池，该线程池中线程数量固定。 在任何时候，最多nThreads线程将处于主动处理任务。 如果所有线程处于活动状态时，提交的其他任务将在等待队列中直到有线程可用。 如果任何线程由于在关闭之前的执行期间发生故障而终止，则如果需要执行后续任务，需要等待新线程，本线程会被新任务占用。 池中的线程将一直存在，直到它明确地使用shutdown 关闭线程。

newFixedThreadPool(int nThreads, ThreadFactory threadFactory) 设置线程最大数量，线程的创建可以只用ThreadFactory。

newScheduledThreadPool 创建一个定长线程池，支持定时及周期性任务执行。

newScheduledThreadPool(int corePoolSize) 返回ScheduleExecutorService，创建一个线程池，可以设置保留在线程池中线程的数量，即使它们处于空闲状态。

newScheduledThreadPool(int corePoolSize, ThreadFactory threadFactory) 通过ThreadFactory创建线程

newSingleThreadExecutor 创建一个单线程化的线程池，它只会用唯一的工作线程来执行任务，保证所有任务按照指定顺序(FIFO, LIFO, 优先级)执行

newSingleThreadExecutor() 返回ExecutorService，创建一个单线程线程池

newSingleThreadScheduledExecutor(ThreadFactory threadFactory) 通过ThreadFactory创建线程

#### class CountDownLatch

java.util.concurrent CountDownLatch是一种通用的同步工具，类似一个线程计数器，例如：我们有多个线程同时执行任务，但是有一个线程在执行任务过程中也许某个过程，需要等其他线程都完成之后才能继续执行，那么就需要有一个类似监听其他线程是否完成的工具，当其他线程完成后通知那个等待的线程继续执行任务。这种场景可能就会用到CountDownLatch。如下：

```java
//创建一个CountDownLatch设置开始线程数量
CountDownLatch countDownLatch = new CountDownLatch(hosts.size());
        try {
            for (String host : hosts) {
                newCachedThreadPool.execute(new SendThread(this, host, countDownLatch));
            }
            //主线程等待，当线程数量为0时，主线程开始继续执行
            countDownLatch.await();
            sendTemporaryCache.deleteRulesList();
        } catch (InterruptedException e) {
            logger.error("执行更新规则线程异常", e);

​        }

//这一部分是每个线程都要执行的部分
try {
            URL getUrl = new URL("http://" + host + RULEAPI_GET_NEW_RULES);
            String result = HttpClientUtils.post(getUrl.toString(), 
                                                 new HashMap<String, Object>(16) {
                {
                    put("rule", JSON.toJSONString(sendTemporaryCache.getRulesList()));
                }
            }, null, ContentType.APPLICATION_JSON);
            if (FAILD.equals(result)) {
                while (number < MAXNUMBER) {
                    number++;
                    String rs = sendRules(host, countDownLatch);
                    if (StringUtils.equals(SUCCESS, rs)) {
                        number = 0;
                        return SUCCESS;
                    }
                }
                logger.error("服务器{}更新规则失败",host);
                return FAILD;
            }
        } catch (Exception e) {
            logger.error("发送新规则异常");
        } finally {
    //每次线程执行完都要调用countDown()方法，减少1个线程
            countDownLatch.countDown();
        }
        return SUCCESS;
    }
```


常用的方法：

await() 线程等待，直到线程锁存器计数为零才继续执行。

await(long timeout, TimeUnit unit) 类似上面的方法，线程等待，直到线程锁存器计数为零或者达到等待时间才继续执行

countDown() 减少锁存器的计数，如果计数达到零，释放所有等待的线程

getCount() 返回当前计数。

#### interface Future

API描述：A Future计算的结果。 提供方法来检查计算是否完成，等待其完成，并检索计算结果。 结果只能在计算完成后使用方法get进行检索，如有必要，阻塞，直到准备就绪。 取消由cancel方法执行。 提供其他方法来确定任务是否正常完成或被取消。 计算完成后，不能取消计算。 如果您想使用Future ，以便不可撤销，但不提供可用的结果，则可以声明Future<?>表格的类型，并返回null作为基础任务的结果。

常用方法：

boolean cancel(boolean mayInterruptIfRunning) 取消执行的任务，若mayInterruptIfRunning为true，则执行的任务应该被中断，反之，允许完成。

get() 等待计算完成，然后检索其结果。如果任务没有完成那就等待，直到获取到结果

get(long timeout, TimeUnit unit) 如果需要等待最多在给定的时间计算完成，然后检索其结果（如果可用）。

isCancelled() 如果此任务在正常完成之前被取消，则返回 true 。

isDone() 返回 true如果任务已完成。

Future模式，实现多个线程同时执行任务，其中有某个线程需要用到其他线程的结果。类似CountDownLatch使用场景

https://www.cnblogs.com/cz123/p/7693064.html 场景介绍不错

https://blog.csdn.net/u014209205/article/details/80598209 结构分析

当然类似场景也可以使用join()方法

```java
public static void main(String[] args) throws InterruptedException {
	long start = System.currentTimeMillis();
	
	// 等凉菜 -- 必须要等待返回的结果，所以要调用join方法
	Thread t1 = new ColdDishThread();
	t1.start();
	t1.join();
	
	// 等包子 -- 必须要等待返回的结果，所以要调用join方法
	Thread t2 = new BumThread();
	t2.start();
	t2.join();
	
	long end = System.currentTimeMillis();
    System.out.println("准备完毕时间："+(end-start));
}
```


还有可以通过Fork/Join

 ![在这里插入图片描述](C:\Users\zhangxiaoliang\Desktop\总结\java常用类库\Fork-Join.png) 

### 自定义线程池

注意：最近阿里发布的 Java开发手册中强制线程池不允许使用 Executors 去创建，而是通过 ThreadPoolExecutor 的方式，这样的处理方式让写的同学更加明确线程池的运行规则，规避资源耗尽的风险

https://www.cnblogs.com/dolphin0520/p/3932921.html java并发编程：线程池的使用

如果并发的线程数量很多，并且每个线程都是执行一个时间很短的任务就结束了，这样频繁创建线程就会大大降低系统的效率，因为频繁创建线程和销毁线程需要时间。

那么有没有一种办法使得线程可以复用，就是执行完一个任务，并不被销毁，而是可以继续执行其他的任务？

在Java中可以通过线程池来达到这样的效果。今天我们就来详细讲解一下Java的线程池，首先我们从最核心的ThreadPoolExecutor类中的方法讲起，然后再讲述它的实现原理，接着给出了它的使用示例，最后讨论了一下如何合理配置线程池的大小。

##### Java中的ThreadPoolExecutor类

 java.uitl.concurrent.ThreadPoolExecutor类是线程池中最核心的一个类，因此如果要透彻地了解Java中的线程池，必须先了解这个类。下面我们来看一下ThreadPoolExecutor类的具体实现源码。

```java
public class ThreadPoolExecutor extends AbstractExecutorService {
    .....
    public ThreadPoolExecutor(int corePoolSize,int maximumPoolSize,long keepAliveTime,TimeUnit unit,
            BlockingQueue<Runnable> workQueue);

public ThreadPoolExecutor(int corePoolSize,int maximumPoolSize,long keepAliveTime,TimeUnit unit,
        BlockingQueue<Runnable> workQueue,ThreadFactory threadFactory);
 
public ThreadPoolExecutor(int corePoolSize,int maximumPoolSize,long keepAliveTime,TimeUnit unit, BlockingQueue<Runnable> workQueue,RejectedExecutionHandler handler);
 
public ThreadPoolExecutor(int corePoolSize,int maximumPoolSize,long eepAliveTime,TimeUnit unit,BlockingQueue<Runnable> workQueue,ThreadFactory threadFactory,RejectedExecutionHandler handler);
...
}
```


从上面的代码可以得知，ThreadPoolExecutor继承了AbstractExecutorService类，并提供了四个构造器，事实上，通过观察每个构造器的源码具体实现，发现前面三个构造器都是调用的第四个构造器进行的初始化工作。

从上面的代码可以得知，ThreadPoolExecutor继承了AbstractExecutorService类，并提供了四个构造器，事实上，通过观察每个构造器的源码具体实现，发现前面三个构造器都是调用的第四个构造器进行的初始化工作。

下面解释下一下构造器中各个参数的含义：

corePoolSize：核心池的大小，这个参数跟后面讲述的线程池的实现原理有非常大的关系。在创建了线程池后，默认情况下，线程池中并没有任何线程，而是等待有任务到来才创建线程去执行任务，除非调用了*prestartAllCoreThreads()或者prestartCoreThread()方法*，从这2个方法的名字就可以看出，是预创建线程的意思，即在没有任务到来之前就创建corePoolSize个线程或者一个线程。默认情况下，在创建了线程池后，线程池中的线程数为0，当有任务来之后，就会创建一个线程去执行任务，当线程池中的线程数目达到corePoolSize后，就会把到达的任务放到缓存队列当中；

maximumPoolSize：线程池最大线程数，这个参数也是一个非常重要的参数，它表示在线程池中最多能创建多少个线程；当队列满时，会创建线程执行任务直到线程池中的数量等于maximumPoolSize。

keepAliveTime：表示线程没有任务执行时最多保持多久时间会终止。默认情况下，只有当线程池中的线程数大于corePoolSize时，keepAliveTime才会起作用，直到线程池中的线程数不大于corePoolSize，即当线程池中的线程数大于corePoolSize时，如果一个线程空闲的时间达到keepAliveTime，则会终止，直到线程池中的线程数不超过corePoolSize。但是如果调用了allowCoreThreadTimeOut(boolean)方法，在线程池中的线程数不大于corePoolSize时，keepAliveTime参数也会起作用，直到线程池中的线程数为0；

unit：参数keepAliveTime的时间单位，有7种取值，在TimeUnit类中有7种静态属性：

TimeUnit.DAYS;               //天
TimeUnit.HOURS;             //小时
TimeUnit.MINUTES;           //分钟
TimeUnit.SECONDS;           //秒
TimeUnit.MILLISECONDS;      //毫秒
TimeUnit.MICROSECONDS;      //微妙
TimeUnit.NANOSECONDS;       //纳秒

workQueue：一个阻塞队列，用来存储等待执行的任务，这个参数的选择也很重要，会对线程池的运行过程产生重大影响，一般来说，这里的阻塞队列有以下几种选择：　ArrayBlockingQueue和PriorityBlockingQueue使用较少，一般使用LinkedBlockingQueue和SynchronousQueue。线程池的排队策略与BlockingQueue有关。
	ArrayBlockingQueue ：一个由数组结构组成的有界阻塞队列。
	LinkedBlockingQueue ：一个由链表结构组成的有界阻塞队列。
	PriorityBlockingQueue ：一个支持优先级排序的无界阻塞队列。
	DelayQueue： 一个使用优先级队列实现的无界阻塞队列。
	SynchronousQueue： 一个不存储元素的阻塞队列。
	LinkedTransferQueue： 一个由链表结构组成的无界阻塞队列。
	LinkedBlockingDeque： 一个由链表结构组成的双向阻塞队列。

threadFactory：线程工厂，主要用来创建线程；
handler：表示当拒绝处理任务时的策略，有以下四种取值
	ThreadPoolExecutor.AbortPolicy:丢弃任务并抛出RejectedExecutionException异常。 
	ThreadPoolExecutor.DiscardPolicy：也是丢弃任务，但是不抛出异常。 
	ThreadPoolExecutor.DiscardOldestPolicy：丢弃队列最前面的任务，然后重新尝试执行任务（重复此过程）
	ThreadPoolExecutor.CallerRunsPolicy：由调用线程处理该任务 

Executor是一个顶层接口，在它里面只声明了一个方法execute(Runnable)，返回值为void，参数为Runnable类型，从字面意思可以理解，就是用来执行传进去的任务的；然后ExecutorService接口继承了Executor接口，并声明了一些方法：submit、invokeAll、invokeAny以及shutDown等；抽象类AbstractExecutorService实现了ExecutorService接口，基本实现了ExecutorService中声明的所有方法；然后ThreadPoolExecutor继承了类AbstractExecutorService。在ThreadPoolExecutor类中有几个非常重要的方法：

​	execute()
​	submit()
​	shutdown()
​	shutdownNow()

execute()方法实际上是Executor中声明的方法，在ThreadPoolExecutor进行了具体的实现，这个方法是ThreadPoolExecutor的核心方法，通过这个方法可以向线程池提交一个任务，交由线程池去执行。

submit()方法是在ExecutorService中声明的方法，在AbstractExecutorService就已经有了具体的实现，在ThreadPoolExecutor中并没有对其进行重写，这个方法也是用来向线程池提交任务的，但是它和execute()方法不同，它能够返回任务执行的结果，去看submit()方法的实现，会发现它实际上还是调用的execute()方法，只不过它利用了Future来获取任务执行结果（Future相关内容将在下一篇讲述）。

shutdown()和shutdownNow()是用来关闭线程池的。

##### 深入剖析线程池实现原理

1.线程池状态

2.任务的执行

3.线程池中的线程初始化

4.任务缓存队列及排队策略

5.任务拒绝策略

6.线程池的关闭

7.线程池容量的动态调整

###### 1.线程池状态

在ThreadPoolExecutor中定义了一个volatile变量，另外定义了几个static final变量表示线程池的各个状态：

```java
volatile int runState;
static final int RUNNING    = 0;
static final int SHUTDOWN   = 1;
static final int STOP       = 2;
static final int TERMINATED = 3;
```

 runState表示当前线程池的状态，它是一个volatile变量用来保证线程之间的可见性；

下面的几个static final变量表示runState可能的几个取值。

当创建线程池后，初始时，线程池处于RUNNING状态；

如果调用了shutdown()方法，则线程池处于SHUTDOWN状态，此时线程池不能够接受新的任务，它会等待所有任务执行完毕；

如果调用了shutdownNow()方法，则线程池处于STOP状态，此时线程池不能接受新的任务，并且会去尝试终止正在执行的任务；

当线程池处于SHUTDOWN或STOP状态，并且所有工作线程已经销毁，任务缓存队列已经清空或执行结束后，线程池被设置为TERMINATED状态。

###### 2.任务的执行

在了解将任务提交给线程池到任务执行完毕整个过程之前，我们先来看一下ThreadPoolExecutor类中其他的一些比较重要成员变量：

```java
private final BlockingQueue<Runnable> workQueue;              //任务缓存队列，用来存放等待执行的任务
private final ReentrantLock mainLock = new ReentrantLock();   //线程池的主要状态锁，对线程池状												//态（比如线程池大小、runState等）的改变都要使用这个锁
private final HashSet<Worker> workers = new HashSet<Worker>();  //用来存放工作集

private volatile long  keepAliveTime;    //线程存货时间   
private volatile boolean allowCoreThreadTimeOut;   //是否允许为核心线程设置存活时间
private volatile int   corePoolSize;     //核心池的大小（即线程池中的线程数目大于这个参数时，提交的											//任务会被放进任务缓存队列）
private volatile int   maximumPoolSize;   //线程池最大能容忍的线程数

private volatile int   poolSize;       //线程池中当前的线程数

private volatile RejectedExecutionHandler handler; //任务拒绝策略

private volatile ThreadFactory threadFactory;   //线程工厂，用来创建线程

private int largestPoolSize;   //用来记录线程池中曾经出现过的最大线程数

private long completedTaskCount;   //用来记录已经执行完毕的任务个数
```

每个变量的作用都已经标明出来了，这里要重点解释一下corePoolSize、maximumPoolSize、largestPoolSize三个变量。

corePoolSize在很多地方被翻译成核心池大小，其实我的理解这个就是线程池的大小。举个简单的例子：

假如有一个工厂，工厂里面有10个工人，每个工人同时只能做一件任务。

因此只要当10个工人中有工人是空闲的，来了任务就分配给空闲的工人做；

当10个工人都有任务在做时，如果还来了任务，就把任务进行排队等待；

如果说新任务数目增长的速度远远大于工人做任务的速度，那么此时工厂主管可能会想补救措施，比如重新招4个临时工人进来；

然后就将任务也分配给这4个临时工人做；

如果说着14个工人做任务的速度还是不够，此时工厂主管可能就要考虑不再接收新的任务或者抛弃前面的一些任务了。

当这14个工人当中有人空闲时，而新任务增长的速度又比较缓慢，工厂主管可能就考虑辞掉4个临时工了，只保持原来的10个工人，毕竟请额外的工人是要花钱的。

这个例子中的corePoolSize就是10，而maximumPoolSize就是14（10+4）。

也就是说corePoolSize就是线程池大小，maximumPoolSize在我看来是线程池的一种补救措施，即任务量突然过大时的一种补救措施。

不过为了方便理解，在本文后面还是将corePoolSize翻译成核心池大小。

largestPoolSize只是一个用来起记录作用的变量，用来记录线程池中曾经有过的最大线程数目，跟线程池的容量没有任何关系。

###### 3.线程池中的线程初始化

默认情况下，创建线程池之后，线程池中是没有线程的，需要提交任务之后才会创建线程。

在实际中如果需要线程池创建之后立即创建线程，可以通过以下两个方法办到：

prestartCoreThread()：初始化一个核心线程；
prestartAllCoreThreads()：初始化所有核心线程

```java
public boolean prestartCoreThread() {
    return addIfUnderCorePoolSize(null); //注意传进去的参数是null
}

public int prestartAllCoreThreads() {
    int n = 0;
    while (addIfUnderCorePoolSize(null))//注意传进去的参数是null
        ++n;
    return n;
}
```



###### 4.任务缓存队列及排队策略

在前面我们多次提到了任务缓存队列，即workQueue，它用来存放等待执行的任务。

workQueue的类型为BlockingQueue，通常可以取下面三种类型：

1）ArrayBlockingQueue：基于数组的先进先出队列，此队列创建时必须指定大小；

2）LinkedBlockingQueue：基于链表的先进先出队列，如果创建时没有指定此队列大小，则默认为Integer.MAX_VALUE；

3）synchronousQueue：这个队列比较特殊，它不会保存提交的任务，而是将直接新建一个线程来执行新来的任务。

###### 5.任务拒绝策略

当线程池的任务缓存队列已满并且线程池中的线程数目达到maximumPoolSize，如果还有任务到来就会采取任务拒绝策略，通常有以下四种策略：

```java
ThreadPoolExecutor.AbortPolicy:丢弃任务并抛出RejectedExecutionException异常。
ThreadPoolExecutor.DiscardPolicy：也是丢弃任务，但是不抛出异常。
ThreadPoolExecutor.DiscardOldestPolicy：丢弃队列最前面的任务，然后重新尝试执行任务（重复此过程）
ThreadPoolExecutor.CallerRunsPolicy：由调用线程处理该任务
```



###### 6.线程池的关闭

ThreadPoolExecutor提供了两个方法，用于线程池的关闭，分别是shutdown()和shutdownNow()，其中：

shutdown()：不会立即终止线程池，而是要等所有任务缓存队列中的任务都执行完后才终止，但再也不会接受新的任务
shutdownNow()：立即终止线程池，并尝试打断正在执行的任务，并且清空任务缓存队列，返回尚未执行的任务

###### 7.线程池容量的动态调整

ThreadPoolExecutor提供了动态调整线程池容量大小的方法：setCorePoolSize()和setMaximumPoolSize()，

​	setCorePoolSize：设置核心池大小
​	setMaximumPoolSize：设置线程池最大能创建的线程数目大小
​	当上述参数从小变大时，ThreadPoolExecutor进行线程赋值，还可能立即创建新的线程来执行任务。