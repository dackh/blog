# **wait()、notify()、notifyAll()**
Object是所有类的基类，它有5个方法组成了等待、通知机制的核心：notify()、notifyAll()、wait()、wait(long) 和wait(long,int)。在java中，所有的类都是从Object继承而来，因此，所有的类都拥有这些共同的方法可供使用。
### **wait()**
```
public final void wait()  throws InterruptedException,IllegalMonitorStateException
```
该方法用来将当前线程置入休眠状态，直到接到通知或中断为止。**在调用wait()之前，线程必须要获得对象的对象级别的锁**，即只能在同步方法或同步代码块中调用wait()方法。进入wait()方法后，当前线程释放锁。在从wait()返回前，线程与其他线程竞争重新获得锁。如果调用wait()时，没有持有适当的锁，则抛出IllegalMonitorStateException，它是RuntimeException的一个子类，因此不需要try-catch结构。

### **notify()**

```
    public final native void notify() throws IllegalMonitorStateException
```
该方法也要在同步方法或同步代码块中调用，即在调用前，线程也必须要获得该对象的对象级别锁，如果调用notify()时没有持有适当的锁，也会抛出IllegalMonitorStateException。

该方法用来通知那些等待该对象的对象锁的其他线程。如果有多个线程等待，则线程规划器任意挑选其中一个wait()状态得线程来发出通知，并使它们等待获取该对象的对象锁。（**notify后，当前线程不会马上释放对象锁，wait所在的线程并不能马上获取该对象锁，要等到程序退出synchronized代码块后，当前线程才会释放锁，wait所在的线程才可以获得该对象锁**）。

但不惊动其他同样等待该对象notify的线程们，当第一个获得了该对象的wait线程运行完毕之后，它会释放该对象的锁，此时如果该对象没有再次使用notify语句，则即便对象已经空闲，其他wait状态等待的线程由于没有得到该对象的通知，会继续阻塞在wait状态，直到这个对象发出一个notify或notifyAll。

**wait状态等待的是被notify而不是锁。**

### **notifyAll()**

```
     public final native void notifyAll() throws IllegalMonitorStateException
```
该方法与notify()方法的工作方式相同，重要的一点差异：
notifyAll使所有的方法在该对象wait的线程统统退出wait状态，变成等待获取该对象上的锁，一旦该对象锁被释放（即notifyAll线程退出同步代码块时），**它们就会去竞争。如果其中一个线程获得了该对象的锁，它就会继续往下执行，在它退出synchronized代码块，释放锁后，其他的已经被唤醒的线程将会继续竞争该锁，一直进行下去，直到所有被唤醒的线程都执行完毕。** 

```
	
	public class WaitAndNotify {
		public static void main(String[] args) throws InterruptedException{
			WaitAndNotify wan = new WaitAndNotify();
	//		synchronized(wan){
	//			wan.wait();
	//			System.out.println("wait");
	//		}
			new Thread(new Runnable(){
				public void run(){
					synchronized(wan){
						try {
							wan.wait();
							System.out.println("wait");
						} catch (InterruptedException e) {
							e.printStackTrace();
						}
					}
				}
			}).start();
			new Thread(new Runnable(){
				public void run(){
					synchronized(wan){
						try {
							wan.wait();
							System.out.println("wait");
						} catch (InterruptedException e) {
							e.printStackTrace();
						}
					}
				}
			}).start();
			new Thread(new Runnable(){
				public void run(){
					synchronized(wan){
						wan.notifyAll();		//当notify方法时只执行一个wait、而notifyAll方法将执行两个wait
						System.out.println("notify");
					}
				}
			}).start();
		}
	}

```
# **线程的状态及其转换**

![clipboard.png](https://segmentfault.com/img/bVbgFvt)

### **新建**
创建之后未启动

### **就绪状态**
调用了start方法，不过还未被OS调用，或者正在等待CPU时间片。

### **运行状态**
正在被OS执行。
### **阻塞状态**
阻塞状态分三种：
- 同步阻塞：线程正在等待一个排它锁。
- 限期阻塞：调用Thread.sleep()、join()方法等待sleep时间结束或join线程执行完毕。
- 无限期阻塞：通过wait()方法进入，等待notify()或notifyAll()方法唤醒。
### **终止状态**
线程执行完毕或出现异常退了run()。
# **volatile**
### **作用**
volatile可以保证线程的可见性并提供一定的有序性，但是无法保证原子性。

### **原理**
在JVM底层，volatile是采用**内存屏障**来实现的。

#### **内存屏障**
内存屏障是一个cpu指令，他的作用：
- 确保一些特定操作执行的顺序
- 影响一些数据的可见性（可能是某些指令执行后的结果）。编译器和cpu在保证输出结果一样的情况下对指令重排序，使性能得到优化。插入一个内存屏障，相当于告诉CPU和编译器先于这个命令的必须先执行，后于这个命令的必须后执行。

#### **内存屏障跟volatile的关系**
如果字段是volatile，java内存模型将在写操作后插入一个写屏障指令，在读操作前插入一个读屏障指令。这就意味着如果你对一个volatile字段进行写操作，那么，一旦你完成写入，任何访问这个字段的线程都将会得到最新的值；在你写入之前，会保证之前发生的事情已经发生，并且任何更新过的数据值都是可见的。

# 任务执行
串行执行任务：在单个线程中串行地执行各项任务。

显式地为任务创建线程：通过为每个线程创建一个新的线程来提供服务，从而实现更高的响应性。这种方式存在一定缺陷：
- 线程生命周期的开销非常高：线程的创建于销毁需要消耗计算资源。
- 资源消耗：活跃的线程会消耗系统资源，尤其是内存。如果在可用处理器的数量小于可运行的线程数量时，那么有些线程将会被闲置。
- 稳定性：在可创建线程的数量上存在一个限制，如果破坏了这些限制，那么可能会抛出OutOfMemoryError异常。
线程池

# Executor框架：
Executor虽然是一个简单的接口，但它却为灵活且强大的异步任务执行框架提供了基础。它提供了一种标准的方法将任务的提交过程与执行过程解耦开来，应用Runnable来表示任务。同时它还提供了对生命周期的支持。基于生产者-消费者模式，提交任务的操作相当于生产者，执行任务的线程则相当于消费者。

### Executor的生命周期
- Executor的实现通常会创建线程来执行任务，但如果无法正确关闭Executor，那么jvm也将无法关闭。
- 为了解决执行服务的生命周期问题，Executor扩展了ExecutorSerivce借口，添加了一些用户生命周期管理的方法。
- ExecutorService的生命周期有三种状态：运行、关闭和已终止。
    - ExecutorService在初始化创建时为运行状态。
    - Shutdown方法将执行平缓的关闭过程：不在接受新的任务，同时等待已经提交的任务执行完成——包括还未开始执行的任务。
    - ShutdownNow方法将执行粗暴的关闭过程：它将尝试取消所有运行的任务，并且不在启动队列中尚未开始执行的任务。
### Callable跟Future
- Runnable是一定有很大局限的抽象，它不能返回一个值或抛出一个受检查的异常。
- Callable是一种更好的抽象，它认为主入口点（cell）能够返回一个值，并可能抛出一个异常。
- Future表示一个任务的生命周期，并提供相应的方法来判断是否完成或取消。Executor执行的任务有4个生命周期：创建、提交、开始和完成。任务的生命周期只能前进不能后退。Future的get方法的行为取决于任务的状态，如果完成，那么get会立即返回或者抛出一个Exception，如果任务没有完成，那么get将阻塞并指导任务完成，如果任务抛出异常，那么get将该异常封装为ExecutionException并重新抛出。
##### Future的创建方式：
- ExecutorService的所有submit方法都返回一个Future，从而将一个Runnable或Callable提交给Executor，并得到一个Futrue用来获取任务的执行结果或者取消任务。
- 显式的为某个指定的Runnable或Callable实例化一个FutureTask。（FutureTask实现了Runnable，因此可以将它交给Executor来执行或者直接调用它的run方法）
# 线程中断：
Java并没有提供任何机制来安全的终止线程，但它提供了中断，这是一种协作机制，能够使一个线程终止另一个线程的当前工作。

Thread的中断方法：

    public class Thread{
    		public void interrupt(){….}
    		public Boolean isInterrupted(){….}
    		public static Boolean interrupted(){….}
    }

对中断的正确理解：他并不会真正的中断一个正在运行的线程，而只是发出中断请求，然后由线程在下一个合适的时刻中断自己。
- Interrupt()：中断目标线程， 
- IsInterrupt()：放回目标线程的中断状态
- Interrupted()：清除当前线程的中断状态，并返回它之前的值，这也是清除中断状态的唯一方法。
Thread.sleep()和Object.wait()都会检查线程何时中断，并且在发现中断时提前放回。它们在响应中断时执行的操作：清除中断状态，抛出InterruptedException，表示阻塞操作由于中断而提前结束。能够中断处于阻塞、限期等待、无限期等待等状态，但不能中断I/O阻塞跟synchronized锁阻塞。

在调用interrupted()时返回true，除非像屏蔽这个中断，不然就必须对它进行处理。如果一个线程的 run() 方法执行一个无限循环，并且没有执行 sleep() 等会抛出 InterruptedException 的操作，那么调用线程的 interrupt() 方法就无法使线程提前结束。
但是调用 interrupt() 方法会设置线程的中断标记，此时调用 interrupted() 方法会返回 true。因此可以在循环体中使用 interrupted() 方法来判断线程是否处于中断状态，从而提前结束线程。

##### 通过ExecutorService中断
- shutdownNow跟shutdown

##### 通过Future中断
- Future拥有一个cancel方法，该方法带有一个boolean类型参数mayInterruptIfRunning，表示取消操作是否成功，如果mayIterruptIfRunning为true并且任务当前正在某个线程中执行，那么这个线程能被中断，如果mayInterruptIfRunning为false，那么意味着，若任务还没有启动，就不要执行它。
# 线程池
 线程池：管理一组同构工作线程的资源池，通过重用现有的线程而不是创建新线程，可以在处理多个请求时分摊在线程创建和销毁过程中产生巨大开销。  

类库提供了一个灵活的线程池以及一些有用的默认配置。可以用过调用Executors中的静态工厂方法之一来创建一个线程池：
- newFixedThreadPool： 创建一个固定长度的线程池，每当提交一个任务时就创建一个线程，直到达到线程池的最大数量。newFixedThreadPool工厂方法将线程池的基本大小和最大大小设置为参数中的值，并且创建的线程池不会超时。
- newCachedThreadPool： 创建一个可缓存的线程池，如果线程池的当前规模超过了处理需求时，那么将回收空闲的线程，而当需求添加时，则可以添加新的线程，线程池的规模不存在任何限制。newCacheThreadPool工厂方法将线程池的最大大小设置为Integer.MAX_VALUE，而将基本大小设置为0，并将超时大小设置为1分钟。
- newSingleThreadPool： 一个单线程Executor，创建单个工作线程来执行任务，如果这个线程异常结束，会创建另一个线程来替代。newSingleThreadPool能够确保依照任务在队列中的顺序来串行执行（例如FIFO、LIFO、优先级）

### ThreadPoolExecutor
ThreadPoolExecutor为一些Executor提供了基本实现，这些Executor时由Executors中的newFixedThreadPool、newCachedThreadPool和newScheduledThreadExecutor等工厂方法返回的。 

ThreadPoolExecutor是一个灵活的、稳定的线程池，允许进行各种定制。如果默认的执行策略不能满足需求，那么可以通过ThreadPoolExecutor的构造参数来实例化一个对象，并根据自己的需求来定制。
   
       public ThreadPoolExecutor( int corePoolSize,
                                     int maximumPoolSize,
                                     long keepAliveTime,
                                     TimeUnit unit,
                                     BlockingQueue<Runnable> workQueue,
                                     ThreadFactory threadFactory,
                                      RejectedExecutionHandler handler){…}

线程池的基本大小，最大大小以及存活时间等因素公共负责线程的创建和销毁。
- 基本大小(corePoolSize)：线程池的目标大小，即在没有任务执行时线程池的大小，并且只有        在工作队列满了的情况下才会创建超过这个数量的线程。   
- 最大大小(maximumPoolSize)：表示可同时活动的线程数量的上限。 
- 存活时间(keepAliveTime)：当某个线程的空闲时间超过了存活时间，那么将被标记为可回收的，并且当线程的当前大小超过了基本大小时，这个线程将被终止。
 

### 管理队列任务   
在线程池中，如果新请求的到达速率超过线程池的处理速率，那么新到来的请求将积累起来。在线程池中，这些请求会在一个由Executor管理的Runnable队列中等待。  

ThreadPoolExecutor允许提供一个BlockingQueue来保存等待执行的任务。基本的任务排队方法有三种：有界队列、无界队列和同步移交。队列的选择与其他的配置参数有关，例如线程池的大小等。
- newFixedThreadPool和newSingleThreadPool在默认情况下使用一个无界的LinkedBlockingQueue。
- 一个更稳妥的资源管理策略是使用有界队列，例如ArrayBlockingQueue、有界的LinkedBlockingQueu、PriorityBlockingQueue。通过饱和策略来解决队列填满之后，新任务到来的情况。
- 对于非常大的或者无界的线程池，可以通过使用SynchronousQueue来避免任务排队，以及直接将任务从生产者持戒交给工作者线程。SynchronousQueue并不是一个真正的队列，而是一种在线程之间队形移交的机制。要将线程一个元素放入SynchronousQueue中，必须有另一个线程在等待接受。如果没有线程等待，并且线程池的当前大小小于最大值，那么ThreadPoolExecutor将创建一个新的线程。否则根据饱和策略，这个任务将被拒绝。使用直接移交更高效。只有线程池时无界或者可以拒绝任务时，SynchronousQueue才有实际价值。在newCacheThreadPool工厂方法中就使用了SynchronousQueue。

### 饱和策略
当有界队列被填满之后，饱和策略开始发挥作用。JDK提供了集中不同的RejectedExecutionHadler来实现。
-	终止(AbortPolicy)：默认的饱和策略，将策略将抛出未检查的RejectExecutionException，调用者可以捕获这个异常进行处理。
-	抛弃(DiscardPolicy)：将悄悄抛弃该任务。
-	抛弃最旧的(DiscardOldestPolicy)：抛弃下一个将被执行的任务，然后尝试重新提交新的任务。（如果工作队列是一个优先队列，那么将抛弃优先级最高的任务）
-	调用者执行(CallerRunsPolicy)：将任务会退给调用者，从而降低任务的流量。它不是在线程池中的线程执行新提交的任务，而是在一个调用了execute的线程执行该任务。在WebServer中使用有界队列且”调用者运行”饱和策略时，并且线程池所有的线程都被占用，队列已满的情况下，下一个任务将会由调用execute的主线程执行，此时主线程在执行任务期间不会accept请求，故新到来的请求被保存在TCP层的队列中而不是应用程序的队列中，如果持续过载，那么TCP层的请求队列最终将被填满，因为同样会开始抛出请求。
### 线程工厂：
每当线程池需要创建一个线程时，都是通过线程工厂方法来完成的，默认的线程工厂方法将创建一个新的、非守护的线程，并且不包含特殊的配置信息。通过制定一个特定的工厂方法，可以定制线程池的配置信息。在ThreadFactory中只定义了一个方法newThread，每当线程池需要创建一个新线程时都会调用这个方法。
    public interface ThreadFactory {
        /**
         * Constructs a new {@code Thread}.  Implementations may also initialize
         * priority, name, daemon status, {@code ThreadGroup}, etc.
         *
         * @param r a runnable to be executed by new thread instance
         * @return constructed thread, or {@code null} if the request to
         *         create a thread is rejected
         */
        Thread newThread(Runnable r);
    }

# 显式锁
Java5.0增加了一种新的机制：ReentranLock。与内置加锁机制不同的时，Lock提供了一种无条件的、可轮询的、定时的以及可中断的锁获取操作，所有加锁和解锁的方法都是显式的。

    Lock lock = new ReentrantLock();
    		//...
    		lock.lock();
    		try {
    			//更新对象状态
    			//捕获异常，并在必要时恢复不变形条件
    		} finally {
    			lock.unlock();  //不会自动清除锁
    		}

当程序的执行控制单元离开被保护的代码块时，并不会自动清除锁。

### 轮询锁与定时锁
可定时的与可轮询的锁获取模式是由tryLock方法实现的，与无条件的锁获取模式相比，它具有更完善的错误恢复机制。在内置锁中，死锁是一个很严重的问题，恢复程序的唯一方式是重新启动程序，而防止死锁的唯一方法就是在构造程序时避免出现不一致的锁顺序。可定时的与可轮询的锁提供了另一种选择：避免死锁的发生。

如果不能获得所有需要的锁，那么可以使用可定时的或可轮询的锁获取方式，从而使你重新获得控制权，它会释放已经获得的锁，然后重新尝试获取所有锁。（或其他操作）

在实现具有时间限制的操作时，定时锁同样费用有用：当在带有时间限制的操作中调用一个阻塞方法时，它能根据剩余时间来提供一个时限，如果不能在制定的时间内给出结果，那么就会使程序提前结束。
### 可中断锁
内置锁是不可中断的，故有时将使得实现可取消得任务变得复杂，而显示锁可以中断，lockInterruptibly方法能够获得锁的同时保持对中断的响应。LockInterruptibly优先考虑响应中断，而不是响应锁的普通获取或重入获取，既允许线程在还等待时就中断。（lock优先考虑获取锁，待获取锁成功之后才响应中断）
### 公平性
在ReentrantLock的构造函数中提供了两种公平性选择：创建一个非公平的锁（默认）或者一个公平的锁，在公平的锁上，线程讲按照它们发出的请求的顺序来获取锁，但在非公平锁上，则允许插队（当一个线程请求非公平锁时，同时该锁的状态变为可用，那么这个线程将跳过队列中所有的等待线程并获得这个锁）。  

在公平锁中，如果有一个线程持有这个锁或者有其他线程在队列中等待这个锁，那么新发的请求的线程将会放入队列中，而在非公平锁中，只有当锁被某个线程持有时，新发出的请求的线程才会放入队列中。

大多数情况下，非公平锁的性能要高于公平锁  
- 公平性锁在线程的挂起以及恢复中需要一定开销
- 假设线程A持有一个锁，并且线程B请求这个锁，那么在A释放时，讲唤醒B，这时C也请求这个锁，那么可能C很可能会在B被完全唤醒之前就执行完了。

# **java内存模型**
Java虚拟机规范试图定义一种java内存模型（Java Memory Model，JMM）来屏蔽掉各种硬件和操作系统的内存访问差异，以实现让Java程序在各种平台下都能达到一致的内存访问效果。

### **主内存和工作内存**
计算机系统中处理器上的寄存器的读写速度比内存要快几个数量级别，为了解决这种矛盾，在它们之间加入高速缓存。

加入高速缓存的存储交互很好的解决了处理器与内存的速度矛盾，但是也为计算机系统带来了更高的复杂度，因为它引入了一个新的问题：缓存一致性。

多处理器系统中，每个处理器都有自己的高速缓存，而它们又共享同一主内存，当多个处理器的运算任务涉及到同一块主内存区域时，将可能导致各自的缓存不一致。

为了解决一致性问题，需要各个处理器访问缓存时都遵循一些协议，在读写时根据协议来进行操作。

![clipboard.png](https://segmentfault.com/img/bVbgujF)

Java内存模型的主要目标是定义程序中各个变量的访问规则，即在虚拟机中将变量存储到内存以及从内存中取出变量的底层细节。

JMM将所有变量（实例字段、静态字段、数组对象的元素，线程共享的）都存储到主内存中，每个线程有自己的工作内存，工作内存中保存了被线程使用到的变量的主内存的副本拷贝，线程对变量的所有操作都必须在工作内存中进行。

![clipboard.png](https://segmentfault.com/img/bVbgukM)

### **内存间的交互操作**
Java内存模型定义了8个操作来完成主内存和工作内存间的交互操作。

![clipboard.png](https://segmentfault.com/img/bVbgul0)

- read：把一个变量从主内存传输到工作内存。
- load：在read之后执行，把read得到的值放入工作内存的变量副本中。
- use：把工作内存中的一个变量的值传递给执行引擎。
- assign：把一个从执行引擎接收到的值赋给工作内存的变量。
- store：把工作内存的一个变量传送到主内存中。
- write：在store之后执行，把store得到的值放入主内存的变量中。
- lock：作用与主内存变量，它把一个变量标识为一条线程独占的状态。
- unlock：把处于锁定状态得变量释放出来。

### **内存模型的三大特性**
#### **原子性**
Java内存模型保证了read、load、use、assign、store、write、lock和unlock操作具有原子性，例如对一个int类型的变量执行assign复制操作，这个操作就是原子性的。但是Java内存模型允许虚拟机将没有被volatile修饰的64位数据(long,double)的读写操作划分为两个32位的操作来进行，即load、store、read和write操作可以不具备原子性。

虽然Java内存模型保证了这些操作的原子性，但是int等原子性操作在多线程中还是会出现线程安全性问题。
可通过两个方式来解决：
- 使用AtomicInteger
- 通过synchronized互斥锁来保证操作的原子性。

#### **可见性**
可见性指一个线程修改共享变量的值，其他线程能够立即得知这个修改。Java内存模型通过在变量修改后将新值同步回主内存中，在变量读取前从主内存刷新变量值来实现可见性。

主要有三种方式实现可见性：
- volatile
- synchronized，对一个变量执行unlock操作之前，必须把变量值同步回主内存。
- final：被final修饰的字段在构造器中一旦初始化完成，并且没有发生this逃逸（其他线程有可能通过这个引用访问到初始化了一半的对象），那么其他线程就能够看见final字段的值。 

#### **有序性**
有序性是指：在本线程内观察，所有操作都是有序的。在一个线程观察另一个线程，所有操作都是无序的，无序是因为发生了指令重排序。在Java内存模型中，允许编译器和处理器对指令进行重排序，重排序过程中不会影响到单线程程序的执行，却会影响多线程并发执行的正确性。

volatile关键字通过添加内存屏障的方式禁止重排序，即重排序时不能把后面的指令放在内存屏障之前。

也可以通过synchronized来保证有序性，它保证每个时刻只有一个线程执行同步代码，相当于让线程顺序执行同步代码。


# **线程安全的方法**
### **互斥同步**
synchronized和ReentranLock。、
### **非阻塞同步**
互斥同步最主要的问题是线程阻塞和唤醒所带来的性能问题，因此这种这种同步烨称为阻塞同步。

互斥同步是一种悲观的并发策略，总是以为只要不去做正确的同步策略，那就肯定会出问题，无论共享数据是否真的会出现竞争，它都需要进行枷锁。

#### **CAS**
随着硬件指令集的发展，我们可以使用基于冲突检测的乐观并发策略：先进行操作，如果没有其他线程竞争共享资源，那么操作就成功了，否则采取补偿措施（不断尝试、知道成功为止）。这种乐观的并发策略的许多实现都不需要将线程阻塞，因此这种同步操作称为非阻塞同步。

乐观锁需要操作和冲突检测这两个步骤具备原子性，这里就不能再使用互斥同步来保证，只能靠硬件来完成。硬件支持的原子性操作最典型的是：比较和交换（Compare-And-swap，CAS）。CAS指令需要3个操作数，分别是内存地址V、旧的预期值A和新值B。当操作完成时，只有内存地址V等值旧的预期值时，才将V值更新为B。

### **无同步方案**
#### **栈封闭**
多个线程访问同一个方法的局部变量时，不会出现线程安全问题，因为局部变量存储在虚拟机栈中，属于线程私用。

#### **本地线程存储（Thead Local Storage）**
如果一段代码中所需要的数据必须与其他代码共享，那就看看这些共享数据的代码是否能保证在同一个线程中执行。如果能保证，我们就可以把共享数据的可见范围限制在同一个线程之内，这样，无须同步也能保证线程之间不出现数据争用的问题。

符合这种特点的应用并不少见，大部分使用消费队列的架构模式（如“生产者-消费者”模式）都会将产品的消费过程尽量在一个线程中消费完。其中最重要的一个应用实例就是经典 Web 交互模型中的“一个请求对应一个服务器线程”（Thread-per-Request）的处理方式，这种处理方式的广泛应用使得很多 Web 服务端应用都可以使用线程本地存储来解决线程安全问题。

可以使用 java.lang.ThreadLocal 类来实现线程本地存储功能。

# **synchronized的实现**
synchronized关键字在经过编译之后，会在同步代码块前后分别形成monitorenter和monitorexit两个字节码指令。

在执行monitorenter指令时，首先尝试获取对象的锁，如果这个对象没有被锁定或者当前线程已经拥有了这个锁，就把锁的计算器+1，相应的执行完monitorexit指令时将锁计算器减1，当计算器为0时，锁就被释放。

# **锁优化**
指JVM对synchronized的优化。
### **自旋锁**
互斥同步进入阻塞状态的开销都很大，应该尽量避免，在许多应用中，共享数据的锁定状态只会持续很短的一段时间。自旋锁的思想是让一个线程在共享数据的锁执行忙循环（自旋）一段时间，如果在这段时间内能够获得锁，就可以避免进入阻塞状态。

自旋锁虽然能够避免进入阻塞状态从而减少开销，但是它需要进行忙循环操作占用cpu时间，它只适用于共享数据的锁定状态很短的场景。

在JDK1.6中引入了自适应的自旋锁，自适应意味着自旋次数不再是固定的，而是由前一次在同一个锁上的自旋次数及锁的拥塞者的状态来决定。

### **锁清除**
锁清除是指对被检测出不存在竞争的共享数据的锁进行清除。

锁清除主要通过逃逸分析来支持，如果堆上的共享数据不可能逃逸出去被其他线程访问到，那么就可以把他们当成私有数据，也就可以将他们的锁清除。

对于一些看起来没有加锁的代码，其实隐式的加了很多锁，例如一下的字符串拼接代码就隐式加了锁：

```
public static String concatString(String s1, String s2, String s3) {
    return s1 + s2 + s3;
}
```
String是一个不可变的类，编译器会对String的拼接进行自动优化。在JDK1.5之前会转化为StringBuffer对象连续append()操作。

```
public static String concatString(String s1, String s2, String s3) {
    StringBuffer sb = new StringBuffer();
    sb.append(s1);
    sb.append(s2);
    sb.append(s3);
    return sb.toString();
}
```
每一个append()方法中都有一个同步块，虚拟机观察变量sb，很快发现它的动态作用域被限制在concatString方法内部，也就是说，sb的所有引用永远不会逃逸到contatString()方法之外，其他线程无法访问到它，因此可以进行锁清除。

### **锁粗化**
如果一系列的操作都对同一对象反复加锁和解锁，频繁的加锁操作就会导致性能消耗。

上一节的示例代码中连续的append()方法就属于这种情况。如果虚拟机探测到由这样的一串零碎的操作都是对同一个对象加锁，将会把锁的范围扩展（粗化）到整个操作序列的外部。对于上一节的示例代码就是扩展到第一个 append() 操作之前直至最后一个 append() 操作之后，这样只需要加锁一次就可以了。

### **轻量级锁**

轻量级锁跟偏向锁是java1.6中引入的，并且规定锁只可以升级而不能降级，这就意味着偏向锁升级成轻量级锁后不能降低为偏向锁，这种策略是为了提高获得锁的效率。

Java对象头通常由两个部分组成，一个是Mark Word存储对象的hashCode或者锁信息，另一个是Class Metadata Address用于存储对象类型数据的指针，如果对象是数组，还会有一部分存储的是数据的长度。

![clipboard.png](https://segmentfault.com/img/bVbfDIb)

对象头中的Mark Word布局

![clipboard.png](https://segmentfault.com/img/bVbfDId)



轻量级锁是相对于传统的重量级锁而言，它使用CAS操作来避免重量级锁使用互斥量的开销。对于大部分的锁，在整个同步周期内都是不存在晶振的，因此也就不需要使用互斥量进行同步，可以先采用CAS操作进行同步，如果CAS失败了再改用互斥量进行同步。

![clipboard.png](https://segmentfault.com/img/bVbfDLr)

当尝试获取一个锁对象时，如果锁对象标记为0 01，说明锁对象的锁未锁定（unlock）状态，此时虚拟机在当前线程的虚拟机栈创建Lock Record，然后使用CAS操作将对象的Mark Word更新为Lock Record指针。如果CAS操作成功了，那么线程就获取了该对象上的锁，并且对象的Mark Word的锁标记变为00，表示该对象处于轻量级锁状态。

如果CAS操作失败了，虚拟机首先会检查对象的Mark Word是否指向当前线程的虚拟机栈，如果是的话说明当前线程已经拥有了这个锁对象，那就可以直接进入同步块执行，否则说明这个锁对象已经被其他线程抢占了。如果有两个以上的线程竞争同一个锁，那轻量级锁就不再有效，要膨胀为重量级锁。

轻量级锁的步骤如下：
1. 线程1在执行同步代码块之前，JVM会先在当前线程的栈帧中创建一个空间用来存储锁记录，然后再把对象头中的Mark Word复制到该锁记录中，官方称之为Displaced Mark Word。然后线程尝试使用CAS将对象头中的Mark Word 替换为指向锁记录的指针。如果成功，则获得锁，进入步骤(3)。如果失败执行步骤(2)

2. 线程自旋，自旋成功则获得锁，进入步骤(3)。自旋失败，则膨胀成为重量级锁，并把锁标志位变为10，线程阻塞进入步骤(3)

3. 锁的持有线程执行同步代码，执行完CAS替换Mark Word成功释放锁，如果CAS成功则流程结束，CAS失败执行步骤(4)

4. CAS执行失败说明期间有线程尝试获得锁并自旋失败，轻量级锁升级为了重量级锁，此时释放锁之后，还要唤醒等待的线程

### **偏向锁**
偏向锁的思想是偏向于让第一个获取锁对象的线程，在之后的获取该锁就不再需要进行同步操作，甚至连CAS操作也不需要。

当锁对象第一次被线程获得的时候，进入偏向状态，标记为1 01。同时使用CAS操作将线程ID记录到Mark Word中，如果CAS操作成功，这个线程以后每次进入这个锁相关的同步块就不需要再进行任何同步操作。

当有另一个线程去尝试获取这个锁对象，偏向状态就宣告结束，此时偏向取消后恢复到未锁定状态或者轻量级锁状态。

偏向锁获得锁的步骤分为：
1. 初始时对象的Mark Word位为1，表示对象处于可偏向的状态，并且ThreadId为0，这是该对象是biasable&unbiased状态，可以加上偏向锁进入(2)。如果一个线程试图锁住biasable&biased并且ThreadID不等于自己ID的时候，由于锁竞争应该直接进入(4)撤销偏向锁。

2. 线程尝试用CAS将自己的ThreadID放置到Mark Word中相应的位置，如果CAS操作成功进入到3），否则进入(4)

3. 进入到这一步代表当前没有锁竞争，Object继续保持biasable状态，但此时ThreadID已经不为0了，对象处于biasable&biased状态

4. 当线程执行CAS失败，表示另一个线程当前正在竞争该对象上的锁。当到达全局安全点时（cpu没有正在执行的字节）获得偏向锁的线程将被挂起，撤销偏向（偏向位置0），如果这个线程已经死了，则把对象恢复到未锁定状态（标志位改为01），如果线程还活着，则把偏向锁置0，变成轻量级锁（标志位改为00），释放被阻塞的线程，进入到轻量级锁的执行路径中，同时被撤销偏向锁的线程继续往下执行。

5. 运行同步代码块

# **参考**
- Java并发编程实战
- [浅谈Java里的三种锁：偏向锁、轻量级锁和重量级锁][1]
- [CS-Notes][2]
- [【Java并发编程】之十：使用wait/notify/notifyAll实现线程间通信的几点重要说明][3]


  [1]: https://blog.csdn.net/u012722531/article/details/78244786
  [2]: https://github.com/CyC2018/CS-Notes/blob/master/notes/Java%20%E5%B9%B6%E5%8F%91.md#%E4%B8%83juc---aqs
  [3]: https://blog.csdn.net/ns_code/article/details/17225469
