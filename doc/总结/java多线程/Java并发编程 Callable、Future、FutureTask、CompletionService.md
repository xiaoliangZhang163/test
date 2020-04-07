# Java并发编程 Callable、Future、FutureTask、CompletionService


---------------------------https://blog.csdn.net/evankaka/article/details/51610635

         在上一文章中，笔者介绍了线程池及其内部的原理。今天主要讲的也是和线程相关的内容。一般情况下，使用Runnable接口、Thread实现的线程我们都是无法返回结果的。但是如果对一些场合需要线程返回的结果。就要使用用Callable、Future、FutureTask、CompletionService这几个类。Callable只能在ExecutorService的线程池中跑，但有返回结果，也可以通过返回的Future对象查询执行状态。Future 本身也是一种设计模式，它是用来取得异步任务的结果，

一、基本源码
所以来看看它们的源码信息

1、Callable

看看其源码：

public interface Callable<V> {
    V call() throws Exception;
}
它只有一个call方法，并且有一个返回V，是泛型。可以认为这里返回V就是线程返回的结果。

ExecutorService接口：线程池执行调度框架

<T> Future<T> submit(Callable<T> task);
<T> Future<T> submit(Runnable task, T result);
Future<?> submit(Runnable task);
2、Future

Future是我们最常见的

public interface Future<V> {
    //试图取消对此任务的执行。如果任务已完成、或已取消，或者由于某些其他原因而无法取消，则此尝试将失败。当调用 cancel 时，如果调用成功，而此任务尚未启动     //，则此任务将永不运行。如果任务已经启动，则 
    //mayInterruptIfRunning 参数确定是否应该以试图停止任务的方式来中断执行此任务的线程。此方法返回后，对 isDone() 的后续调用将始终返回 true。如果此方法返    //回 true，则对 isCancelled() 
    //的后续调用将始终返回 true。 
    boolean cancel(boolean mayInterruptIfRunning);

    //如果在任务正常完成前将其取消，则返回 true。 
    boolean isCancelled();

   //如果任务已完成，则返回 true。 可能由于正常终止、异常或取消而完成，在所有这些情况中，此方法都将返回 true。
    boolean isDone();

   //等待线程结果返回，会阻塞
    V get() throws InterruptedException, ExecutionException;

   //设置超时时间
    V get(long timeout, TimeUnit unit)
        throws InterruptedException, ExecutionException, TimeoutException;
}

3、FutureTask
从源码看其继承关系如下：







其源码如下：

public class FutureTask<V> implements RunnableFuture<V> {
    //真正用来执行线程的类
    private final Sync sync;

    //构造方法1，从Callable来创建FutureTask
    public FutureTask(Callable<V> callable) {
        if (callable == null)
            throw new NullPointerException();
        sync = new Sync(callable);
    }
     
    //构造方法2，从Runnable来创建FutureTask，V就是线程执行返回结果
    public FutureTask(Runnable runnable, V result) {
        sync = new Sync(Executors.callable(runnable, result));
    }
    //和Futrue一样
    public boolean isCancelled() {
        return sync.innerIsCancelled();
    }
    //和Futrue一样
    public boolean isDone() {
        return sync.innerIsDone();
    }
    //和Futrue一样
    public boolean cancel(boolean mayInterruptIfRunning) {
        return sync.innerCancel(mayInterruptIfRunning);
    }
     
    //和Futrue一样
    public V get() throws InterruptedException, ExecutionException {
        return sync.innerGet();
    }
     
    //和Futrue一样
    public V get(long timeout, TimeUnit unit)
        throws InterruptedException, ExecutionException, TimeoutException {
        return sync.innerGet(unit.toNanos(timeout));
    }
     
    //线程结束后的操作
    protected void done() { }
     
    //设置结果
    protected void set(V v) {
        sync.innerSet(v);
    }
     
    //设置异常
    protected void setException(Throwable t) {
        sync.innerSetException(t);
    }
    //线程执行入口
    public void run() {
        sync.innerRun();
    }
     
    //重置
    protected boolean runAndReset() {
        return sync.innerRunAndReset();
    }
     
    //这个类才是真正执行、关闭线程的类
    private final class Sync extends AbstractQueuedSynchronizer {
        private static final long serialVersionUID = -7828117401763700385L;
        //线程运行状态
        private static final int RUNNING   = 1;
        private static final int RAN       = 2;
        private static final int CANCELLED = 4;


​        
        private final Callable<V> callable;
        private V result;
        private Throwable exception;
     
        //线程实例
        private volatile Thread runner;
        //构造函数
        Sync(Callable<V> callable) {
            this.callable = callable;
        }
     
     。。。。
    }
}
FutureTask类是Future 的一个实现，并实现了Runnable，所以可通过Excutor(线程池) 来执行,也可传递给Thread对象执行。如果在主线程中需要执行比较耗时的操作时，但又不想阻塞主线程时，可以把这些作业交给Future对象在后台完成，当主线程将来需要时，就可以通过Future对象获得后台作业的计算结果或者执行状态。 Executor框架利用FutureTask来完成异步任务，并可以用来进行任何潜在的耗时的计算。一般FutureTask多用于耗时的计算，主线程可以在完成自己的任务后，再去获取结果。FutureTask类既可以使用new Thread(Runnable r)放到一个新线程中跑，也可以使用ExecutorService.submit(Runnable r)放到线程池中跑，而且两种方式都可以获取返回结果，但实质是一样的，即如果要有返回结果那么构造函数一定要注入一个Callable对象。


二、应用实例
1、Future实例

package com.func.axc.futuretask;

import java.util.Random;
import java.util.concurrent.Callable;
import java.util.concurrent.ExecutionException;
import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;
import java.util.concurrent.Future;

/**
 * 功能概要：
 * 
 * @author linbingwen
 * @since  2016年6月8日 
 */
public class FutureTest {

	/**
	 * @author linbingwen
	 * @since  2016年6月8日 
	 * @param args    
	 */
	public static void main(String[] args) {
		   System.out.println("main Thread begin at:"+ System.nanoTime());
		    ExecutorService executor = Executors.newCachedThreadPool();
		    HandleCallable task1 = new HandleCallable("1");
		    HandleCallable task2 = new HandleCallable("2");
		    HandleCallable task3 = new HandleCallable("3");
	        Future<Integer> result1 = executor.submit(task1);
	        Future<Integer> result2 = executor.submit(task2);
	        Future<Integer> result3 = executor.submit(task3);
	        executor.shutdown();
	        try {
	            Thread.sleep(1000);
	        } catch (InterruptedException e1) {
	            e1.printStackTrace();
	        }
	        try {
	            System.out.println("task1运行结果:"+result1.get());
	            System.out.println("task2运行结果:"+result2.get());
	            System.out.println("task3运行结果:"+result3.get());
	        } catch (InterruptedException e) {
	            e.printStackTrace();
	        } catch (ExecutionException e) {
	            e.printStackTrace();
	        }
	        System.out.println("main Thread finish at:"+ System.nanoTime());
	}

}

class HandleCallable implements Callable<Integer>{
	private String name;
	public HandleCallable(String name) {
		this.name = name;
	}
	
    @Override
    public Integer call() throws Exception {
    	System.out.println("task"+ name + "开始进行计算");
    	Thread.sleep(3000);
    	int sum = new Random().nextInt(300);
    	int result = 0;
    	for (int i = 0; i < sum; i++)
    		result += i;
    	return result;
    }
}
执行结果：


2、FutureTask

方法一、直接通过New Thread来启动线程

package com.func.axc.futuretask;

import java.util.Random;
import java.util.concurrent.Callable;
import java.util.concurrent.ExecutionException;
import java.util.concurrent.FutureTask;

import org.springframework.scheduling.config.Task;

/**
 * 功能概要：
 * 
 * @author linbingwen
 * @since 2016年5月31日
 */
public class FutrueTaskTest {

	public static void main(String[] args) {
		//采用直接启动线程的方法
		System.out.println("main Thread begin at:"+ System.nanoTime());
		MyTask task1 = new MyTask("1");
        FutureTask<Integer> result1 = new FutureTask<Integer>(task1);
        Thread thread1 = new Thread(result1);
        thread1.start();
        
		MyTask task2 = new MyTask("2");
    	FutureTask<Integer> result2 = new FutureTask<Integer>(task2);
    	Thread thread2 = new Thread(result2);
    	thread2.start();
	 
		try {
			Thread.sleep(1000);
		} catch (InterruptedException e1) {
			e1.printStackTrace();
		}
		
		try {
			System.out.println("task1返回结果:"  + result1.get());
			System.out.println("task2返回结果:"  + result2.get());
		} catch (InterruptedException e) {
			e.printStackTrace();
		} catch (ExecutionException e) {
			e.printStackTrace();
		}
	 
		System.out.println("main Thread finish at:"+ System.nanoTime());
	
	}
}

class MyTask implements Callable<Integer> {
	private String name;
	
	public MyTask(String name) {
		this.name = name;
	}
	
	@Override
	public Integer call() throws Exception {
		System.out.println("task"+ name + "开始进行计算");
		Thread.sleep(3000);
		int sum = new Random().nextInt(300);
		int result = 0;
		for (int i = 0; i < sum; i++)
			result += i;
		return result;
	}

}

执行结果：


方法二、通过线程池来启动线程

package com.func.axc.futuretask;

import java.util.Random;
import java.util.concurrent.Callable;
import java.util.concurrent.ExecutionException;
import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;
import java.util.concurrent.Future;

/**
 * 功能概要：
 * 
 * @author linbingwen
 * @since 2016年5月31日
 */
public class FutrueTaskTest2 {

	public static void main(String[] args) {
		System.out.println("main Thread begin at:"+ System.nanoTime());
		ExecutorService executor = Executors.newCachedThreadPool();
		MyTask2 task1 = new MyTask2("1");
		MyTask2 task2 = new MyTask2("2");
		Future<Integer> result1 = executor.submit(task1);
		Future<Integer> result2 = executor.submit(task2);
		executor.shutdown();

		try {
			Thread.sleep(1000);
		} catch (InterruptedException e1) {
			e1.printStackTrace();
		}
		
		try {
			System.out.println("task1返回结果:"  + result1.get());
			System.out.println("task2返回结果:"  + result2.get());
		} catch (InterruptedException e) {
			e.printStackTrace();
		} catch (ExecutionException e) {
			e.printStackTrace();
		}
	 
		System.out.println("main Thread finish at:"+ System.nanoTime());
	
	}
}

class MyTask2 implements Callable<Integer> {
	private String name;
	
	public MyTask2(String name) {
		this.name = name;
	}
	
	@Override
	public Integer call() throws Exception {
		System.out.println("task"+ name + "开始进行计算");
		Thread.sleep(3000);
		int sum = new Random().nextInt(300);
		int result = 0;
		for (int i = 0; i < sum; i++)
			result += i;
		return result;
	}

}
执行结果：


三、CompletionService
这个光看其单词，就可以猜到它应该是一个线程执行完成后相关的服务，没错。它就是一个将线程池执行结果放入到一个Blockqueueing的类。那么它和Future或FutureTask有什么不同呢？其实在上面的例子中，笔者用的实例可能不太好。如果在线程池中我们使用Future或FutureTask来取得返回结果，比如。我们开了100条线程。但是这些线程的执行时间是未知的。但是我们又需要返回结果。每执行一条线程就根据结果做一次相应的操作。如果是Future或FutureTask。我们只能通过一个循环，不断的遍历线程池里的线程。取得其执行状态。然后再来取结果。这样效率就太低了，有可能发生一条线程执行完毕了，但我们不能立刻知道它处理完成了。还得通过一个循环来判断。基本上面的这种问题，所以出了CompletionService。

     CompletionService原理不是很难，它就是将一组线程的执行结果放入一个BlockQueueing当中。这里线程的执行结果放入到Blockqueue的顺序只和这个线程的执行时间有关。和它们的启动顺序无关。并且你无需自己在去写很多判断哪个线程是否执行完成，它里面会去帮你处理。

首先看看其源码：

package java.util.concurrent;

public interface CompletionService<V> {
    //提交线程任务
    Future<V> submit(Callable<V> task);
    //提交线程任务
    Future<V> submit(Runnable task, V result);
   //阻塞等待
    Future<V> take() throws InterruptedException;
   //非阻塞等待
    Future<V> poll();
   //带时间的非阻塞等待
    Future<V> poll(long timeout, TimeUnit unit) throws InterruptedException;
}
上面只是一个接口类，其实现类如下：
package java.util.concurrent;

public class ExecutorCompletionService<V> implements CompletionService<V> {
    private final Executor executor;//线程池类
    private final AbstractExecutorService aes;
    private final BlockingQueue<Future<V>> completionQueue;//存放线程执行结果的阻塞队列

    //内部封装的一个用来执线程的FutureTask
    private class QueueingFuture extends FutureTask<Void> {
        QueueingFuture(RunnableFuture<V> task) {
            super(task, null);
            this.task = task;
        }
        protected void done() { completionQueue.add(task); }//线程执行完成后调用此函数将结果放入阻塞队列
        private final Future<V> task;
    }
     
    private RunnableFuture<V> newTaskFor(Callable<V> task) {
        if (aes == null)
            return new FutureTask<V>(task);
        else
            return aes.newTaskFor(task);
    }
     
    private RunnableFuture<V> newTaskFor(Runnable task, V result) {
        if (aes == null)
            return new FutureTask<V>(task, result);
        else
            return aes.newTaskFor(task, result);
    }
     
     //构造函数，这里一般传入一个线程池对象executor的实现类
    public ExecutorCompletionService(Executor executor) {
        if (executor == null)
            throw new NullPointerException();
        this.executor = executor;
        this.aes = (executor instanceof AbstractExecutorService) ?
            (AbstractExecutorService) executor : null;
        this.completionQueue = new LinkedBlockingQueue<Future<V>>();//默认的是链表阻塞队列
    }
     
    //构造函数，可以自己设定阻塞队列
    public ExecutorCompletionService(Executor executor,
                                     BlockingQueue<Future<V>> completionQueue) {
        if (executor == null || completionQueue == null)
            throw new NullPointerException();
        this.executor = executor;
        this.aes = (executor instanceof AbstractExecutorService) ?
            (AbstractExecutorService) executor : null;
        this.completionQueue = completionQueue;
    }
    //提交线程任务，其实最终还是executor去提交
    public Future<V> submit(Callable<V> task) {
        if (task == null) throw new NullPointerException();
        RunnableFuture<V> f = newTaskFor(task);
        executor.execute(new QueueingFuture(f));
        return f;
    }
    //提交线程任务，其实最终还是executor去提交
    public Future<V> submit(Runnable task, V result) {
        if (task == null) throw new NullPointerException();
        RunnableFuture<V> f = newTaskFor(task, result);
        executor.execute(new QueueingFuture(f));
        return f;
    }
     
    public Future<V> take() throws InterruptedException {
        return completionQueue.take();
    }
     
    public Future<V> poll() {
        return completionQueue.poll();
    }
     
    public Future<V> poll(long timeout, TimeUnit unit) throws InterruptedException {
        return completionQueue.poll(timeout, unit);
    }

}
从源码中可以知道。最终还是线程还是提交到Executor当中去运行，所以构造函数中需要Executor参数来实例化。而每次有线程执行完成后往阻塞队列添加一个Future。
这是上面的RunnableFuture，这是每次往线程池是放入的线程。

public interface RunnableFuture<V> extends Runnable, Future<V> {
    void run();
}

接下来以两个例子来说明其使用
1、与Future的区别使用：

自定义一个Callable

class HandleFuture<Integer> implements Callable<Integer> {
	
	private Integer num;
	
	public HandleFuture(Integer num) {
		this.num = num;
	}
	 
	@Override
	public Integer call() throws Exception {
		Thread.sleep(3*100);
		System.out.println(Thread.currentThread().getName());
		return num;
	}

}
首先是Futuer
	public static void FutureTest() throws InterruptedException, ExecutionException {
		System.out.println("main Thread begin:");
		ExecutorService executor = Executors.newCachedThreadPool();
		List<Future<Integer>> result = new ArrayList<Future<Integer>>();
		for (int i = 0;i<10;i++) {
			Future<Integer> submit = executor.submit(new HandleFuture(i));
			result.add(submit);
		}
		executor.shutdown();
		for (int i = 0;i<10;i++) {//一个一个等待返回结果
			System.out.println("返回结果："+result.get(i).get());
		}
		System.out.println("main Thread end:");
	}
执行结果：


从输出结果可以看出,我们只能一个一个阻塞的取出。这中间肯定会浪费一定的时间在等待上。如7返回了。但是前面1-6都没有返回。那么7就得等1-6输出才能输出。
接下来换成CompletionService来做：

	public static void CompleTest() throws InterruptedException, ExecutionException {
		System.out.println("main Thread begin:");
		ExecutorService executor = Executors.newCachedThreadPool();
		// 构建完成服务
		CompletionService<Integer> completionService = new ExecutorCompletionService<Integer>(executor);
		for (int i = 0;i<10;i++) {
			completionService.submit(new HandleFuture(i));
		}
		for (int i = 0;i<10;i++) {//一个一个等待返回结果
			System.out.println("返回结果："+completionService.take().get());
		}
		System.out.println("main Thread end:");
	}

输出结果：

可以看出，结果的输出和线程的放入顺序无关系。每一个线程执行成功后，立刻就输出。