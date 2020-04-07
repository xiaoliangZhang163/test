# java 多线程 锁

-------------------------------https://blog.csdn.net/evankaka/article/details/51866242

一、基础知识
       在Java并发编程里头，锁是一个非常重要的概念。就如同现实生活一样，如果房子上了锁。别人就进不去。Java里头如果一段代码取得了一个锁，其它地方再想去这个锁（或者再执行这个相同的代码）就都得等待锁释放。锁其实分成非常多。比如有互斥锁、读写锁、乐观锁、悲观锁、自旋锁、公平锁、非公平锁等。包括信号量其实都可以认为是一个锁。

 1、什么时需要锁呢？
     其实非常多的场景，如共享实例变量、共享连接资源时以及包括并发包中BlockingQueue、ConcurrentHashMap等并发集合中都大量使用了锁。基体上使用同步的地方都可以改成锁来用，但是使用锁的地方不一定能改成同步来用。

2、 锁和同步的对比
1）同步synchronized算是一个关键词，是来来修饰方法的，但是锁lock是一个实例变量，通过调用lock()方法来取得锁

2）、只能同步方法，而不能同步变量和类，锁也是一样

3）、同步无法保证线程取得方法执行的先后顺序。锁可以设置公平锁来确保。
4）、不必同步类中所有的方法，类可以同时拥有同步和非同步方法。
5）、如果线程拥有同步和非同步方法，则非同步方法可以被多个线程自由访问而不受锁的限制。锁也是一样。
6）、线程睡眠时，它所持的任何锁都不会释放。
7）、线程可以获得多个锁。比如，在一个对象的同步方法里面调用另外一个对象的同步方法，则获取了两个对象的同步锁。
8）、同步损害并发性，应该尽可能缩小同步范围。同步不但可以同步整个方法，还可以同步方法中一部分代码块。
9）、在使用同步代码块时候，应该指定在哪个对象上同步，也就是说要获取哪个对象的锁。例如：

最后，还需要说的一点是。如果使用锁，那么一定的注意编写代码，但不很容易出现死锁！避免方法后文后讲。

 3、简单实例
在看锁的源码时，首先来看个锁的实例，从而对锁有一个简单的理解。由线程A输出1、2、3.接着线程B输出4、5、6.最后线程A再输出7、8、9

package com.func.axc.reentrantlock;

import java.util.concurrent.locks.Condition;
import java.util.concurrent.locks.Lock;
import java.util.concurrent.locks.ReentrantLock;

/**
 * 功能概要：
 * 
 * @author linbingwen
 * @since 2016年5月27日
 */
public class ReenTrantLockTest {

	static class NumberWrapper {
		public int value = 1;
	}

	public static void main(String[] args) {
		// 初始化可重入锁
		final Lock lock = new ReentrantLock();

		// 第一个条件当屏幕上输出到3
		final Condition reachThreeCondition = lock.newCondition();
		// 第二个条件当屏幕上输出到6
		final Condition reachSixCondition = lock.newCondition();
	 
		// NumberWrapper只是为了封装一个数字，一边可以将数字对象共享，并可以设置为final
		// 注意这里不要用Integer, Integer 是不可变对象
		final NumberWrapper num = new NumberWrapper();
		// 初始化A线程
		Thread threadA = new Thread(new Runnable() {
			@Override
			public void run() {
				// 需要先获得锁
				lock.lock();
				try {
					System.out.println("threadA start write");
					// A线程先输出前3个数
					while (num.value <= 3) {
						System.out.println(num.value);
						num.value++;
					}
					// 输出到3时要signal，告诉B线程可以开始了
					reachThreeCondition.signal();
				} finally {
					lock.unlock();
				}
				lock.lock();
				try {
					// 等待输出6的条件
					reachSixCondition.await();
					System.out.println("threadA start write");
					// 输出剩余数字
					while (num.value <= 9) {
						System.out.println(num.value);
						num.value++;
					}
	 
				} catch (InterruptedException e) {
					e.printStackTrace();
				} finally {
					lock.unlock();
				}
			}
	 
		});
	 
		Thread threadB = new Thread(new Runnable() {
			@Override
			public void run() {
				try {
					lock.lock();
	 
					while (num.value <= 3) {
						// 等待3输出完毕的信号
						reachThreeCondition.await();
					}
				} catch (InterruptedException e) {
					e.printStackTrace();
				} finally {
					lock.unlock();
				}
				try {
					lock.lock();
					// 已经收到信号，开始输出4，5，6
					System.out.println("threadB start write");
					while (num.value <= 6) {
						System.out.println(num.value);
						num.value++;
					}
					// 4，5，6输出完毕，告诉A线程6输出完了
					reachSixCondition.signal();
				} finally {
					lock.unlock();
				}
			}
	 
		});
	 
		// 启动两个线程
		threadB.start();
		threadA.start();
	}
}

输出结果：






这个题目用同步的方法也做其实也可以。但是用锁可能更好一点。在上面笔者使用了锁和条件从而完成 了要求。



二、说说源码
最基础的我们先来看看lock方法

package java.util.concurrent.locks;
import java.util.concurrent.TimeUnit;

public interface Lock {

    //取得锁，但是要注意lock()忽视interrupt(), 拿不到锁就 一直阻塞
    void lock();
     
    //同样也是取得锁，但是lockInterruptibly()会响应打扰 interrupt()并catch到InterruptedException，从而跳出阻塞
    void lockInterruptibly() throws InterruptedException;
     
    //尝试取得锁，成功返回true
    boolean tryLock();
     
    //在规定的时间等待里，如果取得锁就返回tre
    boolean tryLock(long time, TimeUnit unit) throws InterruptedException;
     
    //释放锁
    void unlock();
    //条件状态，非常有用，Blockingqueue阻塞队列就是用到它了
    Condition newCondition();
}
1、接下来看看它最常见的实现类，ReentrantLock可重入锁。
public class ReentrantLock implements Lock, java.io.Serializable {
    private static final long serialVersionUID = 7373984872572414699L;
    private final Sync sync; //就只有一个Sync变量，ReentrantLock的所有方法基本都是调用Sync的方法 
2、构造函数
    public ReentrantLock() {
        sync = new NonfairSync(); //默认非公平锁
    }

    public ReentrantLock(boolean fair) {
        sync = (fair)? new FairSync() : new NonfairSync();//公平锁
    }
其里的公平锁的意思是哪个线程先来等待，谁就先获得这个锁。而非公平锁则是看操作系统的调度，有不确定性。一般设置成非公平锁的性能会好很多。
3、然后看看lock方法
    public void lock() {
        sync.lock();
    }
还有这个
    public void lockInterruptibly() throws InterruptedException {
        sync.acquireInterruptibly(1);
    }

发现都 是调用 sync这个变量的方法，它其实是一个ReentrantLock的内部类。真实起作用的其实是它，所以直接看它源码：
首先是非公平锁：

    final static class NonfairSync extends Sync {
        private static final long serialVersionUID = 7316153563782823691L;
     
        final void lock() {
            if (compareAndSetState(0, 1)) //0未获取，1已经获取
                setExclusiveOwnerThread(Thread.currentThread());//设置独占模式，则一个锁只能被一个线程持有，其他线程必须要等待。
            else
                acquire(1);//如果没有取得锁，尝试使用信号量的方式
        }
     
        protected final boolean tryAcquire(int acquires) {
            return nonfairTryAcquire(acquires);
它使用到的方法如下：
    //设置状态
    protected final boolean compareAndSetState(int expect, int update) {
        return unsafe.compareAndSwapInt(this, stateOffset, expect, update);
    }

    //可以看到, compareAndSwapInt不是用Java实现的, 而是通过JNI调用操作系统的原生程序.注意它是原子方法(C++写的)
   public final native boolean compareAndSwapInt(Object o, long offset,int expected, int x);
最终取得锁的方法其实在java Unsafe类的compareAndSwap方法。compareAndSwap是个原子方法,原理是cas.就是说如果他是xx,那么就改为xxx. 这个是高效,而且是原子的,不用加锁. 也不用但是其他值改了而产生误操作,应为会先判断当前值,符合期望才去改变. 
4、tryLock()方法
上面是lock方法是的调用，如果是tryLock呢？

    public boolean tryLock() {
        return sync.nonfairTryAcquire(1);
    }
再看sync的方法
        final boolean nonfairTryAcquire(int acquires) {
            final Thread current = Thread.currentThread();
            int c = getState();//取得状态
            if (c == 0) {//0表示未获取锁
                if (compareAndSetState(0, acquires)) {//CAS设置状态
                    setExclusiveOwnerThread(current);//设置独占线程
                    return true;
                }
            }
            else if (current == getExclusiveOwnerThread()) {//当前线程已有这个锁了
                int nextc = c + acquires;//设置重入的次数，如果是一个线程在有锁的情况下多次调用tryLock就有可能进入这个方法
                if (nextc < 0) // 重入数溢出了
                    throw new Error("Maximum lock count exceeded");
                setState(nextc);
                return true;
            }
            return false;//如果到这里就是没有取到锁了，
        }

其中getState()方法是在AbstractQueuedSynchronizer类的就方法，取得就是下面这个变量
private volatile int state;
在互斥锁中它表示着线程是否已经获取了锁，0未获取，1已经获取了，大于1表示重入数。同时AQS提供了getState()、setState()、compareAndSetState()方法来获取和修改该值：

protected final int getState() {
return state;
}
protected final void setState(int newState) {
state = newState;
}
protected final boolean compareAndSetState(int expect, int update) {
return unsafe.compareAndSwapInt(this, stateOffset, expect, update);
}
这些方法需要java.util.concurrent.atomic包的支持，采用CAS操作，保证其原则性和可见性。
5、tryLock(long timeout, TimeUnit unit)方法
    public boolean tryLock(long timeout, TimeUnit unit) throws InterruptedException {
        return sync.tryAcquireNanos(1, unit.toNanos(timeout));
    }

带有超时时间等待获取锁的方法。真正调用 的其实是Sync父类AbstractQueuedSynchronizer的方法
    public final boolean tryAcquireNanos(int arg, long nanosTimeout) throws InterruptedException {
	if (Thread.interrupted())//检测到当前线程的中断标志为true
	    throw new InterruptedException();
	return tryAcquire(arg) ||
	    doAcquireNanos(arg, nanosTimeout);
    }

这里调用 了两个方法tryAcquire和doAcquireNanos，其实tryAcquire调用的方法就是Lock()调用的方法
        protected final boolean tryAcquire(int acquires) {
            return nonfairTryAcquire(acquires);
        }
这样就不再说明。下面直接来看doAcquireNanos方法，它才是一直在等待循环获取锁的方法。
    private boolean doAcquireNanos(int arg, long nanosTimeout)
        throws InterruptedException {
        long lastTime = System.nanoTime();
        final Node node = addWaiter(Node.EXCLUSIVE);//放入等待的节点，会组成 一个链表
        try {
            for (;;) { //死循环，时间到了才会跳出
                final Node p = node.predecessor();
                if (p == head && tryAcquire(arg)) { //当前节点是头节点。然后尝试获得锁
                    setHead(node);
                    p.next = null; // 把当前节点去掉
                    return true;
                }
                if (nanosTimeout <= 0) { //超出等待时间
                    cancelAcquire(node);
                    return false;
                }
                if (nanosTimeout > spinForTimeoutThreshold &&
                    shouldParkAfterFailedAcquire(p, node))
                    LockSupport.parkNanos(this, nanosTimeout);//还在等待时间内
                long now = System.nanoTime();
                nanosTimeout -= now - lastTime;
                lastTime = now;
                if (Thread.interrupted())//检测到中断信号，直接跳出
                    break;
            }
        } catch (RuntimeException ex) {
            cancelAcquire(node);
            throw ex;
        }
        cancelAcquire(node);//检测 到中断信号时才会执行到这里
        throw new InterruptedException();
    }