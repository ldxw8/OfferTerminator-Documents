# 技术面试必备基础知识-Java

> 在线阅读：[CS-Notes-Java](https://cyc2018.github.io/CS-Notes/#/README?id=%e2%98%95%ef%b8%8f-java)

## Java 基础

## Java 容器
### 容器概览
#### Collection
##### Set
##### List
##### Queue

#### Map

### 容器的设计模式
#### 迭代器模式
#### 适配器模式

### 容器的源码解析
#### ArrayList
#### LinkedList
#### HashMap
#### ConcurrentHashMap

## Java 并发
### 线程状态转换

### 使用线程
- Java 中有三种使用线程的方法： 实现 `Runnable` 接口；实现 `Callable` 接口；继承 `Thread` 类。

	> 实现 Runnable 和 Callable 接口的类只能当做一个可以在线程中运行的任务，不是真正意义上的线程。因此最后还需要通过 Thread 来调用，可以说任务是通过线程驱动从而执行的。

#### 实现 Runnable 接口

- 需要实现 run() 方法；通过 Thread 调用 start() 方法来启动线程。

	```java
	public class MyRunnable implements Runnable {
	    public void run() {
	        // ...
	    }
	}
	
	public static void main(String[] args) {
	    MyRunnable instance = new MyRunnable();
	    Thread thread = new Thread(instance);
	    thread.start();
	}
	```

#### 实现 Callable 接口
- 与 Runnable 相比，Callable 可以有返回值，返回值通过 FutureTask 进行封装。

	```java
	public class MyCallable implements Callable<Integer> {
	    public Integer call() {
	        return 123;
	    }
	}

	public static void main(String[] args) 
	    throws ExecutionException, InterruptedException {
	    MyCallable mc = new MyCallable();
	    FutureTask<Integer> ft = new FutureTask<>(mc);
	    Thread thread = new Thread(ft);
	    thread.start();
	    System.out.println(ft.get());
	}
	```

#### 继承 Thread 类
- 同样也是需要实现 run() 方法，因为 Thread 类也实现了 Runable 接口。

	当调用 start() 方法启动一个线程时，虚拟机会将该线程放入就绪队列中等待被调度，当一个线程被调度时会执行该线程的 run() 方法。

	```java	
	public class MyThread extends Thread {
	    public void run() {
	        // ...
	    }
	}
	
	public static void main(String[] args) {
	    MyThread mt = new MyThread();
	    mt.start();
	}
	```

#### 实现接口 / 继承 Thread
- Java 不支持多重继承，因此继承了 Thread 类就无法继承其它类，但是可以实现多个接口；
- 类可能只要求可执行就行，继承整个 Thread 类开销过大。

	> 为此实现接口会更好一些。

### 基础线程机制
### 中断机制

### 同步互斥
- Java 提供了两种锁机制来控制 `多个线程` 对共享资源的 `互斥访问`，第一个是 JVM 实现的 `synchronized`，而另一个是 JDK 实现的 `ReentrantLock`。

#### 参考资料
- [Aoho. 并发编程的锁机制 synchronized 和 lock. juejin.im](https://juejin.im/post/5a43ad786fb9a0450909cb5f)
- [Matrix海子. Java并发编程 Lock. cnblogs.com](https://www.cnblogs.com/dolphin0520/p/3923167.html)
- [Cyc2018. Java并发-互斥同步. cyc2018.github.io](https://cyc2018.github.io/CS-Notes/#/notes/Java%20并发?id=五、互斥同步)

####  锁的分类
- 锁的分类有很多种，比如自旋锁、自旋锁的其他种类、阻塞锁、可重入锁、读写锁、互斥锁、悲观锁、乐观锁、公平锁、可重入锁等。

	我们这边重点看如下几种：`可重入锁`、`读写锁`、`可中断锁`、`公平锁`。
	
	|  | Synchronized | ReentranLock | ReentrantReadWirteLock |
	| :-: | :-: | :-: | :-: |
	| 可重入锁 | ✔️ | ✔️ | ➖ |
	| 读写锁 | ➖ | ➖ | ✔️ |
	| 可中断锁 | ❌ | ✔️ | ➖ |
	| 公平锁 | ❌ | ❌ or ✔️ | ❌ or ✔️ |
	
	> ✔️ 表示支持；❌ 表示不支持；➖ 表示无关

- `可重入锁`：可重入性表明了锁的分配机制，即基于线程的分配，而不是基于方法调用的分配。若锁具备可重入性则称其为可重入锁。

	`synchronized` 和 `ReentrantLock` 都是可重入锁。

	> 例如：一个线程执行到 method1 的 synchronized 方法时，而在 method1中会调用另外一个 synchronized 方法 method2，此时该线程不必重新去申请锁，而是可以直接执行方法 method2。

- `读写锁`：读写锁将对一个资源的访问分成了 2 个锁。比如文件，一个读锁 (共享锁) 和一个写锁 (排它锁)。正因为有了读写锁，才使得多个线程之间的读操作不会发生冲突。

	`ReadWriteLock` 接口就是读写锁，`ReentrantReadWriteLock` 实现了这个接口。可以通过 readLock() 获取读锁，通过 writeLock() 获取写锁。

- `可中断锁`：即可以中断的锁。`synchronized` 就不是 `可中断锁`，而 `Lock` 是 `可中断锁`。Lock 接口中的 `lockInterruptibly()` 方法就体现了 Lock 的可中断性。

	> 例如：某一线程 A 正在执行锁中的代码，另一线程 B 正在等待获取该锁，可能由于等待时间过长，线程 B 不想等待了，想先处理其他事情，我们可以让它中断自己或者在别的线程中中断它，这种就是可中断锁。

- `公平锁`：尽量以 `请求锁的顺序` 来获取锁。同时若有多个线程在等待一个锁，当这个锁被释放时，等待时间最久的线程或 `最先请求的线程` 会获得该锁，这种就是公平锁。

	非公平锁即无法保证锁的获取是按照请求锁的顺序进行的，这样就可能导致某个或者一些线程永远获取不到锁。
	
	- synchronized 就不是 `公平锁`，它无法保证等待的线程获取锁的顺序。
	- Lock 也不是 `公平锁`，它无法保证等待的线程获取锁的顺序。对于 `ReentrantLock` 和 `ReentrantReadWriteLock`，默认是非公平锁，可设置为公平锁。

#### Lock 和 synchronized
##### synchronized
- synchronized 是 Java 的关键字，当它用来修饰一个方法或者一个代码块的时候，能够保证在同一时刻最多只有一个线程执行该段代码。简单总结如下四种用法：
	- `代码块`：对某一代码块使用，synchronized 后跟括号，括号里是变量。

		```java
		public int func(int m){
			synchronized(m) {
				//...
			}
		}
		```
		
	- `方法声明`：它和同步代码块一样，作用于同一个对象。即一次只能一个线程进入该方法，其他线程想在此时调用该方法只能排队等候。

		```java
		// 放在范围操作符之后，返回类型声明之前
		public synchronized void func() {
			// ...
		}
		```

	- `同步一个对象`：synchronized 后面括号里是对象，此时线程获得的是对象锁。由于调用的是同一个对象的同步代码块，因此这两个线程会进行同步。当一个线程进入同步语句块时，另一个线程就必须等待。

		> 同一对象需同步，不同对象不同步。

		```java
		public class SynchronizedExample {
			public void func1() {
				synchronized (this) {
					for (int i = 0; i < 10; i++) {
						System.out.print(i + " ");
					}
				}
			}
		}
		
		// Output: 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9
		public static void main(String[] args) {
			SynchronizedExample e1 =  new SynchronizedExample();
			ExecutorService executorService =
				Executors.newCachedThreadPool();
			executorService.execute(() -> e1.func1());
			executorService.execute(() -> e1.func1());
		}
		
		// Output: 0 0 1 1 2 2 3 3 4 4 5 5 6 6 7 7 8 8 9 9
		public static void main(String[] args) {
			SynchronizedExample e1 = new SynchronizedExample();
			SynchronizedExample e2 = new SynchronizedExample();
			ExecutorService executorService = 
				Executors.newCachedThreadPool();
			executorService.execute(() -> e1.func1());
			executorService.execute(() -> e2.func1());
		}
		```

	- `同步一个类`：作用于整个类，也就是说两个线程调用同一个类的不同对象上的这种同步语句，也会进行同步。

		> 同一个类，不同一对象需同步。

		```java
		public class SynchronizedExample {
			public void func2() {
				synchronized (SynchronizedExample.class) {
					for (int i = 0; i < 10; i++) {
						System.out.print(i + " ");
					}
				}
			}
		}
		
		// Output: 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9
		public static void main(String[] args) {
		SynchronizedExample e1 = new SynchronizedExample();
		SynchronizedExample e2 = new SynchronizedExample();
		ExecutorService executorService =
			Executors.newCachedThreadPool();
		executorService.execute(() -> e1.func2());
		executorService.execute(() -> e2.func2());
		}
		```

	-  同步一个 `静态方法`：

		```java
		public synchronized static void fun() {
			// ...
		}
		```
	
##### Lock
> Lock 是锁接口，其实现类为 ReetrantLock。  
> ReadWriteLock 是读写锁接口，其实现类为 ReetrantReadWriteLock。  

- Lock：

	```java
	public interface Lock {
		/**
		 * 获取锁，如果锁被其他线程获取，处于等待状态
		 * 必须主动去释放锁，并且在发生异常时不会自动释放锁。
		 * 因此一般来说，使用 Lock 必须在 try{...}catch{...} 块中进行，
		 * 并且将释放锁的操作放在 `finally` 块中进行，
		 * 以保证锁一定被被释放，防止死锁的发生。
		 */
		void lock();
		
		// 尝试获取锁
		// 若失败，等待的过程中可响应中断 threadWait.interrupt()
		void lockInterruptibly() throws InterruptedException;  
		
		// 尝试获取锁，若获取成功，就马上返回 true
		// 否则马上返回 false (锁已经被其他线程获取)
		boolean tryLock();  
		
		// 尝试获取锁，若获取失败，会等待 unit 时间
		// 等待期间还拿不到锁就马上返回 false
		boolean tryLock(long time, TimeUnit unit) 
			throws InterruptedException;  
			
		// 释放锁，一定要在 finally 块中释放
		void unlock();  
		Condition newCondition();
	}
	```

- ReetrantLock：可重入锁，Lock 接口的实现类，且内部定义了公平锁与非公平锁 (默认为非公平锁)：

	```java
	// 默认情况
	public ReentrantLock() {  
		sync = new NonfairSync();  
	}
	
	// 可以手动设置为公平锁：
	public ReentrantLock(boolean fair) {  
		sync = fair ? new FairSync() : new NonfairSync();  
	}  
	```

- ReadWriteLock：一个用来获取读锁 (共享锁)，一个用来获取写锁 (排它锁)。也就是说将文件的读写操作分开，分成 2 个锁来分配给线程，从而使得多个线程可以同时进行读操作。

	```java
	public interface ReadWriteLock {  
		Lock readLock();	// 获取读锁  
		Lock writeLock();	// 获取写锁  
	}
	```
	
- ReentrantReadWirteLock 实现了 ReadWirteLock 接口，并未实现 Lock 接口。
	
	```java
	private ReadWriteLock rwl = new ReentrantReadWriteLock();
	
	public static void main(String[] args) {
		final Main main = new Main();
	
		new Thread(
			() -> main.testRWL(Thread.currentThread()) ).start();
		new Thread(
			() -> main.testRWL(Thread.currentThread()) ).start();
	
	    // 输出的结果是两个 thread 交替输出“正在读”
	}
	
	public void testRWL(Thread thread) {
		rwl.readLock().lock();
		try {
			long finish = System.currentTimeMillis() + 1;
			while (System.currentTimeMillis() <= finish) {
				System.out.println(thread.getName() + "正在读");
			}
			System.out.println(thread.getName() + "读结束");
		} finally {
			rwl.readLock().unlock();
		}
	}
	```
	
	- 若一个线程已经占用了读锁，其他线程可以马上获得读锁，但需要等待才能获取写锁，则申请写锁的线程会一直等待释放读锁。

	- 若一个线程已经占用了写锁，其他线程要获取读锁或写锁都需要等待，则申请的线程会一直等待释放写锁。

####  锁的比较
- `锁的实现`：synchronized 是 Java 关键字，Lock 是接口。synchronized 是 JVM 实现的，而 Lock / ReentrantLock 是 JDK 实现的。
- `性能比较`：新版本 Java 对 synchronized 进行了很多优化，例如自旋锁等。synchronized 与 ReentrantLock 大致相同。
- `异常处理方式`：
	- synchronized：会自动释放线程占有的锁，因此不会导致死锁现象发生；
	- Lock：若没有主动通过 unLock() 去释放锁，则很可能造成死锁现象，因此使用 Lock 时需要在 finally 块中释放锁。
- `等待可中断`：当持有锁的线程长期不释放锁的时候，正在等待的线程可以选择放弃等待，改为处理其他事情。

	> Lock / ReentrantLock 可中断，而 synchronized 不行。

#### 使用选择
- 除非需要使用 Lock / ReentrantLock 的 `高级功能`，否则优先使用 `synchronized`。

	> Lock 是基于在语言层面实现的锁，Lock 锁可以被中断，支持定时锁等。

- 因为 synchronized 是 `JVM` 实现的一种锁机制，JVM 原生地支持它，而 ReentrantLock 不是所有的 JDK 版本都支持。
- 并且使用 synchronized 不用担心没有释放锁而导致 `死锁问题`，因为 JVM 会确保锁的释放。

### 线程之间协作
### J.U.C - AQS
### J.U.C - 其他组件
### 线程不安全示例
### Java 内存模型
### 线程安全
### 锁优化
### 多线程开发良好实践

## Java 虚拟机

## Java I/O
