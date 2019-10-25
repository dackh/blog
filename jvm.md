# **JDK、JRE、JVM三者的关系**
- JDK(Java Development Kit)是针对Java开发的产品、是整个Java的核心，包括Java运行环境JRE、Java工具包和Java基础类库。
- JRE(Java Runtime Environment)是运行Java程序所必须的环境的集合，包含JVM标准实现及Java核心类库。
- JVM(Java Virtual Machine)是整个Java跨平台的最核心的部分，能够运行以Java语言写作的软件程序。所有的Java程序都会首先被编译为.class文件，这种类文件可以在虚拟机上运行，class文件并不直接与机器的操作系统相对应，而是经过虚拟机间接与操作系统交互，由虚拟机将程序解释给本地系统执行。
# **Java运行时区域**

![clipboard.png](https://segmentfault.com/img/bVbdRvK)

### **程序计数器**
内存中较小的内存空间，通过计数器的值可以选取下一条执行的字节码指令，分支、循环、跳转、异常处理、线程恢复等基础功能都需要依赖这个计数器来完成。

线程私有，生命周期跟线程相同。

如果正在执行一个Native方法，那么这个计数器值将为空。

### **虚拟机栈**
线程私有，生命周期跟线程相同。

每个方法在执行同时都会创建一个栈帧，用于存储局部变量表、操作数栈、动态链接、方法出口等信息。

在Java虚拟机规范中，对这个区域规定了两种异常情况：如果线程请求的栈深度大于虚拟机所允许的深度，将抛出StackOverflowError异常；
如果虚拟机栈可以动态扩展，如果扩展时无法申请到足够的内存，就会抛出OutOfMemoryError异常。

### **本地方法栈**
跟虚拟机栈所发挥的作用相似，区别在于虚拟机栈为虚拟机执行Java(也就是字节码)服务，而本地方法栈则为虚拟机使用到的Native方法服务。

### **Java堆**
用于存放对象实例，是Java虚拟机所管理的内存中最大的一块，同时也是所有线程共享的一块内存区域。

因为Java堆是垃圾收集器管理的主要区域，因此很多时候也被称为“GC"堆。由于现在收集器基本都采用分代收集算法，所以Java堆还可以细分为
- 新生代
- 老年代
- 永久代（永久代是Hotspot虚拟机特有的概念，是方法区的一种实现，别的JVM都没有这个东西。在Java 8中，永久代被彻底移除，取而代之的是另一块与堆不相连的本地内存——元空间。）

当一个对象被创建时，它首先进入新生代，之后有可能被转移到老年代中。

新生代存放着大量的生命很短的对象，因此新生代在三个区域中垃圾回收的频率最高。为了更高效地进行垃圾回收，把新生代继续划分成以下三个空间：
- Eden
- From Survivor
- To Survivor

### **方法区**
与Java堆一样，各个线程共享的内存区域，存储已被虚拟机加载的类信息、常量、静态变量、即使编译器编译后的代码等数据。

### **运行时常量池**
方法区的一部分，用于存放编译器生成的各种字面量和符号引用。

运行时常量池相对于class文件常量池的另外一个重要特征是具备动态性，Java语言并不要求常量一定只有编译期才能产生，也就是并非预置入class文件中常量池的内容才能进入方法区运行时常量池，运行期间也可能将新的常量放入池中，这种特性被开发人员利用得比较多的便是String类的intern()方法。

### **直接内存**
在JDK1.4中新加入了NIO类，引入了一种基于通道与缓冲区的I/O方法，它可以使用Native函数库直接分配堆外内存，然后通过一个存储在Java堆中的DirectByteBuffer对象作为这块内存的引用进行操作。
[堆外内存之 DirectByteBuffer 详解][1]
# **HotSpot虚拟机对象**
### **对象的创建**
在语言层上，创建对象通常仅仅是一个new关键字而已，而当虚拟机遇到一条new执行时，将由一下步骤：
- 检查类是否加载、解析、初始化过，没有则先执行相应的类加载过程。
- 在堆中分配内存
    - 划分可用空间：
        - 指针碰撞：堆内存规整
        - 空闲列表：堆内存不规整
    - 并发问题
        - 同步：采用CAS配上失败重试的方式保证更新操作的原子性
        - 把内存分配动作按照线程划分在不同的空间之中进行
- 将分配到的内存空间都初始化零值
- 设置对象的类实例、元数据、哈希码、GC分代年龄等信息。
- 执行<init>方法

### **对象的内存布局**
对象在内存中储存的布局可以分为3块区域：
- 对象头
    - 对象运行时数据、哈希码、GC分代年龄、锁状态标记、线程持有的锁、偏向线程ID等
    - 类型执行：即对象执向它的类元数据的指针，指明对象数据哪个类的实例。
- 实例数据
    - 对象真正存储的有效信息
- 对齐填充
    - 占位符作用
    
### **对象的访问定位**
- 句柄定位
- 直接指针


# **内存溢出**
内存溢出out of memory，是指程序在申请空间时，没有足够的内存空间供其使用，出现了Out of memory error。
### **堆内存溢出**
当new一个对象或者数组时，如果超出了Jvm的head内存最大限制就会爆出异常。

伪代码：

```
while(ture){
    new Object();
}
```

### **栈内存溢出**
在Java虚拟机规范中，对这个栈规定了两种异常情况，如果线程请求的栈深度大于虚拟机所允许的深度，将抛出StackOutFlowError异常，如果虚拟机可以动态扩展（当前大部分Java虚拟机都可动态扩展，只不过Java虚拟机规范中也允许固定长度的虚拟机栈），当扩展时无法申请得到足够的内存时将会抛出OutOfMemory。

#### **StackOutFlowError**
线程中的stack是线程私有的，默认大小通常为1M，可以通过-Xss来设置，-Xss越大，则线程获取的内存越大。
常见问题在线程内过度的调用函数，函数调用会消耗栈空间。

伪代码：

```
public void SOFETest(){
    SOFETest();
}
```

#### **OutOfMemoryError**
Java的栈空间被所有线程分配成一块一块的，每个线程只占一块。而Jvm的栈空间的最小分配单位有-Xss来决定。-Xss有两个语义，即定义每个线程的栈大小，也定义了虚拟机的最小栈内存的分配单位。

如果申请的线程没有获得栈空间可以分配了就会抛出OutOfMemoryError。表示栈空间不足，溢出异常。

代码：该代码可能导致JVM无法申请得到太多的栈内存而导致操作系统因为栈空间不足假死。
```
public class Main {
	public static void main(String[] args) throws ClassNotFoundException {
		CountDownLatch countDownLatch = new CountDownLatch(1);
		for(int i =0;i<1020000000;i++){
			new Thread(new Runnable(){
				@Override
				public void run() {
					int a = 1000;
					try {
						countDownLatch.await();
					} catch (InterruptedException e) {
						// TODO Auto-generated catch block
						e.printStackTrace();
					}
				}
				
			}).start();
		}
		countDownLatch.countDown();
	}
}
```

### **内存泄漏**
内存泄漏memory leak，指程序在申请内存之后，无法释放已申请的内存空间，一次内存泄漏危害可以忽略，多次memory leak将导致oom。

内存泄漏是指你向系统申请分配内存进行使用(new)，可是使用完了以后却不归还(delete)，结果你申请到的那块内存你自己也不能再访问（也许你把它的地址给弄丢了），而系统也不能再次将它分配给需要的程序。
 
# **jvm性能调优监控工具使用详解**

**该部分内容转自：[JVM性能调优监控工具jps、jstack、jmap、jhat、jstat、hprof使用详解][2]**

## **jps(Java Virture Machine Process Status Tool)**

jps主要用来输出JVM中运行的进程状态信息。语法格式如下：

    jps [options] [hostid]

如果不指定hostid就默认为当前主机或服务器。

命令行参数选项说明如下：

    -q 不输出类名、Jar名和传入main方法的参数
    -m 输出传入main方法的参数
    -l 输出main类或Jar的全限名
    -v 输出传入JVM的参数

比如下面：

    root@ubuntu:/# jps -m -l
    2458 org.artifactory.standalone.main.Main /usr/local/artifactory-2.2.5/etc/jetty.xml
    29920 com.sun.tools.hat.Main -port 9998 /tmp/dump.dat
    3149 org.apache.catalina.startup.Bootstrap start
    30972 sun.tools.jps.Jps -m -l
    8247 org.apache.catalina.startup.Bootstrap start
    25687 com.sun.tools.hat.Main -port 9999 dump.dat
    21711 mrf-center.jar

### **jstack**

jstack主要用来查看某个Java进程内的线程堆栈信息。语法格式如下：

    jstack [option] pid
    jstack [option] executable core
    jstack [option] [server-id@]remote-hostname-or-ip
 命令行参数选项说明如下：

    -l long listings，会打印出额外的锁信息，在发生死锁时可以用jstack -l pid来观察锁持有情况
    -m mixed mode，不仅会输出Java堆栈信息，还会输出C/C++堆栈信息（比如Native方法）

jstack可以定位到线程堆栈，根据堆栈信息我们可以定位到具体代码，所以它在JVM性能调优中使用得非常多。下面我们来一个实例找出某个Java进程中最耗费CPU的Java线程并定位堆栈信息，用到的命令有ps、top、printf、jstack、grep。

第一步先找出Java进程ID，我部署在服务器上的Java应用名称为mrf-center：

    root@ubuntu:/# ps -ef | grep mrf-center | grep -v grep
    root     21711     1  1 14:47 pts/3    00:02:10 java -jar mrf-center.jar
得到进程ID为21711，第二步找出该进程内最耗费CPU的线程，可以使用ps -Lfp pid或者ps -mp pid -o THREAD, tid, time或者top -Hp pid，我这里用第三个，输出如下：

![clipboard.png](https://segmentfault.com/img/bVbi3Lx)


TIME列就是各个Java线程耗费的CPU时间，CPU时间最长的是线程ID为21742的线程，用

    printf "%x\n" 21742

得到21742的十六进制值为54ee，下面会用到。    

OK，下一步终于轮到jstack上场了，它用来输出进程21711的堆栈信息，然后根据线程ID的十六进制值grep，如下：

    root@ubuntu:/# jstack 21711 | grep 54ee
    "PollIntervalRetrySchedulerThread" prio=10 tid=0x00007f950043e000 nid=0x54ee in Object.wait() [0x00007f94c6eda000]
可以看到CPU消耗在PollIntervalRetrySchedulerThread这个类的Object.wait()，我找了下我的代码，定位到下面的代码：

    // Idle wait
    getLog().info("Thread [" + getName() + "] is idle waiting...");
    schedulerThreadState = PollTaskSchedulerThreadState.IdleWaiting;
    long now = System.currentTimeMillis();
    long waitTime = now + getIdleWaitTime();
    long timeUntilContinue = waitTime - now;
    synchronized(sigLock) {
    	try {
        	if(!halted.get()) {
        		sigLock.wait(timeUntilContinue);
        	}
        } 
    	catch (InterruptedException ignore) {
        }
    }
它是轮询任务的空闲等待代码，上面的sigLock.wait(timeUntilContinue)就对应了前面的Object.wait()。


### **jmap（Memory Map）和jhat（Java Heap Analysis Tool）**

jmap用来查看堆内存使用状况，一般结合jhat使用。

jmap语法格式如下：

    jmap [option] pid
    jmap [option] executable core
    jmap [option] [server-id@]remote-hostname-or-ip
如果运行在64位JVM上，可能需要指定-J-d64命令选项参数。

    jmap -permstat pid
打印进程的类加载器和类加载器加载的持久代对象信息，输出：类加载器名称、对象是否存活（不可靠）、对象地址、父类加载器、已加载的类大小等信息，如下图：

![clipboard.png](https://segmentfault.com/img/bVbi3LB)

使用jmap -heap pid查看进程堆内存使用情况，包括使用的GC算法、堆配置参数和各代中堆内存使用情况。比如下面的例子：

    root@ubuntu:/# jmap -heap 21711
    Attaching to process ID 21711, please wait...
    Debugger attached successfully.
    Server compiler detected.
    JVM version is 20.10-b01
    
    using thread-local object allocation.
    Parallel GC with 4 thread(s)
    
    Heap Configuration:
       MinHeapFreeRatio = 40
       MaxHeapFreeRatio = 70
       MaxHeapSize      = 2067791872 (1972.0MB)
       NewSize          = 1310720 (1.25MB)
       MaxNewSize       = 17592186044415 MB
       OldSize          = 5439488 (5.1875MB)
       NewRatio         = 2
       SurvivorRatio    = 8
       PermSize         = 21757952 (20.75MB)
       MaxPermSize      = 85983232 (82.0MB)
    
    Heap Usage:
    PS Young Generation
    Eden Space:
       capacity = 6422528 (6.125MB)
       used     = 5445552 (5.1932830810546875MB)
       free     = 976976 (0.9317169189453125MB)
       84.78829520089286% used
    From Space:
       capacity = 131072 (0.125MB)
       used     = 98304 (0.09375MB)
       free     = 32768 (0.03125MB)
       75.0% used
    To Space:
       capacity = 131072 (0.125MB)
       used     = 0 (0.0MB)
       free     = 131072 (0.125MB)
       0.0% used
    PS Old Generation
       capacity = 35258368 (33.625MB)
       used     = 4119544 (3.9287033081054688MB)
       free     = 31138824 (29.69629669189453MB)
       11.683876009235595% used
    PS Perm Generation
       capacity = 52428800 (50.0MB)
       used     = 26075168 (24.867218017578125MB)
       free     = 26353632 (25.132781982421875MB)
       49.73443603515625% used
       ....
使用jmap -histo[:live] pid查看堆内存中的对象数目、大小统计直方图，如果带上live则只统计活对象，如下：

    root@ubuntu:/# jmap -histo:live 21711 | more
    
     num     #instances         #bytes  class name
    ----------------------------------------------
       1:         38445        5597736  <constMethodKlass>
       2:         38445        5237288  <methodKlass>
       3:          3500        3749504  <constantPoolKlass>
       4:         60858        3242600  <symbolKlass>
       5:          3500        2715264  <instanceKlassKlass>
       6:          2796        2131424  <constantPoolCacheKlass>
       7:          5543        1317400  [I
       8:         13714        1010768  [C
       9:          4752        1003344  [B
      10:          1225         639656  <methodDataKlass>
      11:         14194         454208  java.lang.String
      12:          3809         396136  java.lang.Class
      13:          4979         311952  [S
      14:          5598         287064  [[I
      15:          3028         266464  java.lang.reflect.Method
      16:           280         163520  <objArrayKlassKlass>
      17:          4355         139360  java.util.HashMap$Entry
      18:          1869         138568  [Ljava.util.HashMap$Entry;
      19:          2443          97720  java.util.LinkedHashMap$Entry
      20:          2072          82880  java.lang.ref.SoftReference
      21:          1807          71528  [Ljava.lang.Object;
      22:          2206          70592  java.lang.ref.WeakReference
      23:           934          52304  java.util.LinkedHashMap
      24:           871          48776  java.beans.MethodDescriptor
      25:          1442          46144  java.util.concurrent.ConcurrentHashMap$HashEntry
      26:           804          38592  java.util.HashMap
      27:           948          37920  java.util.concurrent.ConcurrentHashMap$Segment
      28:          1621          35696  [Ljava.lang.Class;
      29:          1313          34880  [Ljava.lang.String;
      30:          1396          33504  java.util.LinkedList$Entry
      31:           462          33264  java.lang.reflect.Field
      32:          1024          32768  java.util.Hashtable$Entry
      33:           948          31440  [Ljava.util.concurrent.ConcurrentHashMap$HashEntry;
class name是对象类型，说明如下：
    
    B  byte
    C  char
    D  double
    F  float
    I  int
    J  long
    Z  boolean
    [  数组，如[I表示int[]
    [L+类名 其他对象
还有一个很常用的情况是：用jmap把进程内存使用情况dump到文件中，再用jhat分析查看。jmap进行dump命令格式如下：

    jmap -dump:format=b,file=dumpFileName pid

我一样地对上面进程ID为21711进行Dump：

    root@ubuntu:/# jmap -dump:format=b,file=/tmp/dump.dat 21711     
    Dumping heap to /tmp/dump.dat ...
    Heap dump file created

dump出来的文件可以用MAT、VisualVM等工具查看，这里用jhat查看：

    root@ubuntu:/# jhat -port 9998 /tmp/dump.dat
    Reading from /tmp/dump.dat...
    Dump file created Tue Jan 28 17:46:14 CST 2014
    Snapshot read, resolving...
    Resolving 132207 objects...
    Chasing references, expect 26 dots..........................
    Eliminating duplicate references..........................
    Snapshot resolved.
    Started HTTP server on port 9998
    Server is ready.
注意如果Dump文件太大，可能需要加上-J-Xmx512m这种参数指定最大堆内存，即jhat -J-Xmx512m -port 9998 /tmp/dump.dat。然后就可以在浏览器中输入主机地址:9998查看了：

![clipboard.png](https://segmentfault.com/img/bVbi3LN)


上面红线框出来的部分大家可以自己去摸索下，最后一项支持OQL（对象查询语言）。


### **jstat（JVM统计监测工具）**

语法格式如下：

    jstat [ generalOption | outputOptions vmid [interval[s|ms] [count]] ]
vmid是Java虚拟机ID，在Linux/Unix系统上一般就是进程ID。interval是采样时间间隔。count是采样数目。比如下面输出的是GC信息，采样时间间隔为250ms，采样数为4：

    root@ubuntu:/# jstat -gc 21711 250 4
     S0C    S1C    S0U    S1U      EC       EU        OC         OU       PC     PU    YGC     YGCT    FGC    FGCT     GCT   
    192.0  192.0   64.0   0.0    6144.0   1854.9   32000.0     4111.6   55296.0 25472.7    702    0.431   3      0.218    0.649
    192.0  192.0   64.0   0.0    6144.0   1972.2   32000.0     4111.6   55296.0 25472.7    702    0.431   3      0.218    0.649
    192.0  192.0   64.0   0.0    6144.0   1972.2   32000.0     4111.6   55296.0 25472.7    702    0.431   3      0.218    0.649
    192.0  192.0   64.0   0.0    6144.0   2109.7   32000.0     4111.6   55296.0 25472.7    702    0.431   3      0.218    0.649

要明白上面各列的意义，先看JVM堆内存布局：

![clipboard.png](https://segmentfault.com/img/bVbi3LX)

可以看出：

    堆内存 = 年轻代 + 年老代 + 永久代
    年轻代 = Eden区 + 两个Survivor区（From和To）
现在来解释各列含义：

    S0C、S1C、S0U、S1U：Survivor 0/1区容量（Capacity）和使用量（Used）
    EC、EU：Eden区容量和使用量
    OC、OU：年老代容量和使用量
    PC、PU：永久代容量和使用量
    YGC、YGT：年轻代GC次数和GC耗时
    FGC、FGCT：Full GC次数和Full GC耗时
    GCT：GC总耗时

### **hprof（Heap/CPU Profiling Tool）**

hprof能够展现CPU使用率，统计堆内存使用情况。

语法格式如下：

    java -agentlib:hprof[=options] ToBeProfiledClass
    java -Xrunprof[:options] ToBeProfiledClass
    javac -J-agentlib:hprof[=options] ToBeProfiledClass
完整的命令选项如下：

    Option Name and Value  Description                    Default
    ---------------------  -----------                    -------
    heap=dump|sites|all    heap profiling                 all
    cpu=samples|times|old  CPU usage                      off
    monitor=y|n            monitor contention             n
    format=a|b             text(txt) or binary output     a
    file=<file>            write data to file             java.hprof[.txt]
    net=<host>:<port>      send data over a socket        off
    depth=<size>           stack trace depth              4
    interval=<ms>          sample interval in ms          10
    cutoff=<value>         output cutoff point            0.0001
    lineno=y|n             line number in traces?         y
    thread=y|n             thread in traces?              n
    doe=y|n                dump on exit?                  y
    msa=y|n                Solaris micro state accounting n
    force=y|n              force output to <file>         y
    verbose=y|n            print messages about dumps     y
来几个官方指南上的实例。

CPU Usage Sampling Profiling(cpu=samples)的例子：

    java -agentlib:hprof=cpu=samples,interval=20,depth=3 Hello

上面每隔20毫秒采样CPU消耗信息，堆栈深度为3，生成的profile文件名称是java.hprof.txt，在当前目录。 

CPU Usage Times Profiling(cpu=times)的例子，它相对于CPU Usage Sampling Profile能够获得更加细粒度的CPU消耗信息，能够细到每个方法调用的开始和结束，它的实现使用了字节码注入技术（BCI）：

    javac -J-agentlib:hprof=cpu=times Hello.java

Heap Allocation Profiling(heap=sites)的例子：

    javac -J-agentlib:hprof=heap=sites Hello.java

Heap Dump(heap=dump)的例子，它比上面的Heap Allocation Profiling能生成更详细的Heap Dump信息：

    javac -J-agentlib:hprof=heap=dump Hello.java
虽然在JVM启动参数中加入-Xrunprof:heap=sites参数可以生成CPU/Heap Profile文件，但对JVM性能影响非常大，不建议在线上服务器环境使用。

# **垃圾收集器**
程序计数器、虚拟机栈和本地方法栈这三个区域属于线程私有的，只存在于线程的生命周期内，线程结束之后也会消失，因此不需要对这三个区域进行垃圾回收。垃圾回收主要是针对 Java 堆和方法区进行。

### **判断对象是否死亡**
#### **引用计数算法**
给对象添加一个引用计数器，每当有一个地方引用它，计数器值就加1；引用时效时，计算器值就减1；当计数器值为0的对象就是不可能再被使用的。

当两个对象相互引用时，此时引用计数器的值永远不为0，导致无法对它们进行垃圾回收。
   

     public class ReferenceCountingGC {
            public Object instance = null;
        
            public static void testGC() {
                ReferenceCountingGC objA = new ReferenceCountingGC();
                ReferenceCountingGC objB = new ReferenceCountingGC();
                objA .instance = objB ;
                objB .instance = objA ;
                objA = null;
                objB = null;
                
                System.gc();
            }
        }

#### **可达性分析算法**
以GC Roots为起始点，从这些节点开始向下搜索，能够搜索到的对象都是存活的，不可达的对象则为不可用。 

![clipboard.png](https://segmentfault.com/img/bVbdRv2)

在Java语言中，可作为GC Roots的对象包括下面几种：
- 虚拟机栈中引用的对象
- 方法区中静态属性引用的对象
- 方法区中常量引用的对象
- 本地方法栈中Native方法引用的对象



### **引用类型**
无论是引用计数算法还是可达性分析算法判断对象是否存活都与引用有关。在JDK1.2之后，Java对引用的概念进行了扩充，划分为强度不同的四个的引用类型。 
#### **强引用**
通过new来创建对象的引用类型，被强引用的对象永远不会被垃圾收集器回收。

```
Object obj = new Object();
```
#### **软引用**
通过SortReference类来实现，只有在内存不足的时候才会被回收。
```
    Object obj = new Object();
    SoftReference<Object> sr = new SoftReference<Object>(obj);
    obj = null;
```

#### **弱引用**
通过WeakReference类来实现，只能存活到下一次垃圾收集发生之前。
```
    Object obj = new Object();
    WeakReference<Object> wr = new WeakReference<Object>(obj);
    obj = null;
```

WeakHashMap 的 Entry 继承自 WeakReference，主要用来实现缓存。

private static class Entry<K,V> extends WeakReference<Object> implements Map.Entry<K,V>
Tomcat 中的 ConcurrentCache 就使用了 WeakHashMap 来实现缓存功能。ConcurrentCache 采取的是分代缓存，经常使用的对象放入 eden 中，而不常用的对象放入 longterm。eden 使用 ConcurrentHashMap 实现，longterm 使用 WeakHashMap，保证了不常使用的对象容易被回收。

    public final class ConcurrentCache<K, V> {
    
        private final int size;
    
        private final Map<K, V> eden;
    
        private final Map<K, V> longterm;
    
        public ConcurrentCache(int size) {
            this.size = size;
            this.eden = new ConcurrentHashMap<>(size);
            this.longterm = new WeakHashMap<>(size);
        }
    
        public V get(K k) {
            V v = this.eden.get(k);
            if (v == null) {
                v = this.longterm.get(k);
                if (v != null)
                    this.eden.put(k, v);
            }
            return v;
        }
    
        public void put(K k, V v) {
            if (this.eden.size() >= size) {
                this.longterm.putAll(this.eden);
                this.eden.clear();
            }
            this.eden.put(k, v);
        }
    }

#### **虚引用**
也称为幽灵引用或者幻影引用，是最弱的一种引用关系。

通过PhantomReference类来实现，为一个对象设置虚引用关联的唯一目的就是能在这个对象被收集器回收时收到一个系统通知。

```
    Object obj = new Object();
	PhantomReference<Object> wr = new PhantomReference<Object>(obj, null);
	obj = null;
```

# **垃圾收集算法**
### **标记清除**
算法分为“标记”跟“清除”两个阶段：首先标记出所有需要回收的对象，在标记完成之后统一回收所有被标记的对象。

不足：
- 效率问题，标记跟清除两个过程的效率都不高
- 空间问题，标记清除之后会产生大量不连续的内存碎片。

![clipboard.png](https://segmentfault.com/img/bVbdR7y)

### **复制算法**
将内存分为大小相等的两块，每次只使用其中的一块，当这块内存用完了，就将还存活的对象负责到另一块上面，然后再把一是要难过过得内存空间一次清理掉。

不足：
- 代价太高，只使用一半内存。

![clipboard.png](https://segmentfault.com/img/bVbdR8C)

### **标记整理算法**
首先标记出所有需要回收的对象，然后将所有存活的对象都向一端移动，最后清理掉端边界以外的内存。

![clipboard.png](https://segmentfault.com/img/bVbdR86)

### **分代收集算法**
根据对象的存活周期将内存划分为几块。一般将Java堆分为新生代跟老年代，这样就可以根据各个年代的特点采用最适当的收集算法。
- 新生代：复制算法
- 老年代：标记整理或标记清除算法。

# **垃圾收集器**
如果说手机算法是内存回收的方法论，那么垃圾收集器就是内存回收的具体实现。

![clipboard.png](https://segmentfault.com/img/bVbdSaH)

上图展示了7种不同分代的收集器，如果两个收集器之间存在连线，就说明它们可以搭配使用。

知道目前为止还没有最好的收集器出现，更加没有万能的收集器，所以我们只能选择对具体应用最合适的收集器。

### **Serial收集器**

![clipboard.png](https://segmentfault.com/img/bVbdT4O)

最基本、最悠久的收集器，单线程收集器，复制算法 

在它进行垃圾收集时，必须暂停其他所有的工作线程，直到它收集结束。

相比较与其他收集器，它具有：简单而高效的特点，对于限定CPU环境来说，Serial收集器没有线程交互的开销。

依然是虚拟机运行在Client模式下的默认新生代收集器。

### **ParNew收集器**

![clipboard.png](https://segmentfault.com/img/bVbdT4J)

Serial的多线程版本、并行，复制算法。

是许多运行在Server模式下的虚拟机中首选的新生代收集器，因为目前除了Serial收集器外，只有它能与CMS收集器配合使用。

默认开启的收集线程数与CPU的数量相同，在CPU非常多的环境下，可以使用-XX:ParallelGCThreads参数来限制垃圾收集的线程数。

### **Parallel Scavenge收集器**
新生代、并行的多线程收集器，复制算法。

Parallel Scavenge收集器的目标是达到一个可控制的吞吐量：CPU用户运行用户代码的时间与CPU的执行时间，即吞吐量 = 运行用户代码时间 / (运行用户代码时间 + 垃圾收集时间)。

停顿时间越短越适合需要与用户交互的程序，良好的响应速度能提升用户体验，而高吞吐量可以高效率地利用CPU时间，尽快完成程序的运算任务，主要适合在后台运算而不需要太多交互的任务。

Parallel Scavenge收集器提供了两个参数用于精确控制吞吐量，分别是控制最大垃圾收集停顿时间-XX:MaxGCPauseMillis参数以及直接设置吞吐量大小的-XX:GCTimeRatio参数。

### **Serial Old收集器**

![clipboard.png](https://segmentfault.com/img/bVbdT7E)

Serial的老年代版本，单线程，标记-整理算法。

这个收集器的主要意义在于给Client模式下的虚拟机使用，如果在Server默认下，它还有两大用途：
- 在JDK1.5以及之前版本中与Parallel Scavenge收集器搭配使用。
- 作为CMS收集器的后备方案。

### **Parallel Old收集器**

![clipboard.png](https://segmentfault.com/img/bVbdT8d)

parallel Scavenge的老年代版本，多线程，标记-整理算法，JDK1.6之后提供。

在注重吐吞量以及CPU资源敏感的场合，都可以优先考虑Parallel Scavenge加Parallel Old收集器。

### **CMS收集器**

![clipboard.png](https://segmentfault.com/img/bVbdT9g)

CMS(Concurrent Mark Sweep)，从名字来就可以看出，基于标记-清除算法。

并发收集、低停顿。

运算过程分为4个步骤：
- 初始标记：标记GC Roots能直接关联得到的对象，速度很快
- 并发标记：进行GC Roots Tracing过程，时间较长，不停顿
- 重新标记：修正并发标记期间因用户程序继续运作而导致标记产生变动的那一部分对象的标记记录，停顿时间一般比初始标记稍长一些，但远比并发标记短。
- 并发清理：耗时较长，不停顿。
整个过程耗时最长的并发标记和并发清除过程收集器线程都可以与用户安城一起工作。

CMS还远达不到完美的程度，还有以下3个缺点：
- 对CPU资源敏感，并发阶段占用CPU资源而导致用户线程变慢；低停顿时间是以牺牲吞吐量为代价的，导致CPU利用率不高。 
- 无法处理浮动垃圾，由于CMS并发清理阶段用户线程还在运行着，伴随着程序运行自然还会有新的垃圾产生，这部分垃圾CMS无法在档次收集中处理掉它们，只有留待下一次GC时再清理。也是由于垃圾收集阶段用户还需要运行，故还需要预留足够的内存空间给用户线程使用，因此CMS收集器不能像其他收集器那样等到老年代几乎完全被填满了在进行收集，需要预留一部分空间提供并发收集时的程序运作使用。
- 标记-清除算法会导致大量的空间碎片，空间碎片过多时，将会给大对象分配带来很大麻烦，往往会出现老年代还有很大空间剩余却无法找到足够大的连续空间来分配当前对象，不得不提前触发一次Full GC。
 
### **G1收集器**
一款面向服务端应用的垃圾收集器。HotSpot开发团队赋予它的使命是未来可以替代掉JDK1.5中发布的CMS收集器。

与其他收集器相比，G1具有以下特点：
- 并发与并行：使用多个CPU来缩短Stop-The-World停顿的时间。
- 分代收集：与其他收集器一样，分代概念在G1中依然得以保存，虽然G1可以不需要其他收集器配合就能独立管理GC堆，但它能够采用不同的方式去处理新创建的对象和一存活了一段时间、熬过多次GC的旧对象。
- 空间整合：从整体来看是基于标记-整理算法实现的收集器，从局部（两个Region之间)来看是基于复制算法来实现的，意味着G1运作期间不会产生内存空间碎片。
- 可预测停顿：能够让使用者明确指定一个长度为M毫秒的时间片段内，消耗在垃圾收集上的时间不得超过N毫秒。

G1将整个Java堆划分为多个大小相等的独立区域（Region)，虽然还保留新生代和老年代的概念，但是新生代和老年代不再是物理隔离级别，它们都是一部分Region的集合。

![clipboard.png](https://segmentfault.com/img/bVbdUmi)


G1收集器之所以能够建立可预测的停顿时间模型，是因为它可以有计划低避免在整个Java堆中进行全区域的垃圾回收。G1跟踪各个Region里面的垃圾堆积的价值大小（回收所获得的空间大小以及回收所需时间的经验值），在后台维护一个优先列表，每次根据允许的收集时间，优先回收价值最大的Region。

虚拟机使用Remembered Set来避免全栈扫描，G1中每个Region都有一个与之对应的Remembered Set，用来记录该Region对象的引用对象所在的Region。

![clipboard.png](https://segmentfault.com/img/bVbdUmd)

如果不计算Remembered Set的操作，G1收集器的运作大致可分为：
- 初始标记
- 并发标记
- 最终标记：为了修正在并发标记期间因用户程序继续运作而导致标记产生变化的那一部分标记记录，虚拟机将这段时间对象变化记录在线程Remembered Set Logs里面，最终标记阶段需要把Remembered Set Logs的数据合并到Remembered Set中，这阶段需要停顿线程，但是可以并行执行。
- 筛选回收：首先对各个Region的回收价值和成本进行排序，根据用户所期望的GC停顿时间来制定回收计划。此阶段其实可以做到与用户程序一起并发执行，但是因为只回收一部分Region，时间是用户可控制的，而且停顿用户线程将大幅度提高收集效率。

# **回收策略**
- **新生代GC(Minor GC)**：发生在新生代的垃圾收集动作，因为Java对象大多都具备朝生息灭的特性，所以Minor GC非常频繁，一般回收速度也比较快。
- **老年代GC(major GC/full GC)**：发生在老年代的GC，出现了Major GC经常会伴随一次的Minor GC（但非绝对，在paraller Scavenge收集器的手机策略里就有直接进行Major GC的策略选择过程）。Major GC的速度一般会比Minor GC慢10倍以上。
- **Full GC的触发条件**
   - **调用System.gc()**：只是建议虚拟机执行Full GC，但是虚拟机不一定真正地执行，不建议使用这种方式，而是让虚拟机管理内存。
   - **老年代空间不足**：老年代空间不足的最常见场景为前文所讲的大对象直接进入老年代，长期存活的对象进入老年代等。为了避免以上原因引起的 Full GC，应当尽量不要创建过大的对象以及数组。除此之外，可以通过 -Xmn 虚拟机参数调大新生代的大小，让对象尽量在新生代被回收掉，不进入老年代。还可以通过 -XX:MaxTenuringThreshold 调大对象进入老年代的年龄，让对象在新生代多存活一段时间。
   - **空间分配担保失败**：使用复制算法的 Minor GC 需要老年代的内存空间作担保，如果担保失败会执行一次 Full GC。具体内容请参考上面的第五小节。
   - **JDK 1.7 及以前的永久代空间不足**：在 JDK 1.7 及以前，HotSpot 虚拟机中的方法区是用永久代实现的，永久代中存放的为一些 Class 的信息、常量、静态变量等数据，当系统中要加载的类、反射的类和调用的方法较多时，永久代可能会被占满，在未配置为采用 CMS GC 的情况下也会执行 Full GC。如果经过 Full GC 仍然回收不了，那么虚拟机会抛出 java.lang.OutOfMemoryError。为避免以上原因引起的 Full GC，可采用的方法为增大永久代空间或转为使用 CMS GC。
   - **Concurrent Mode Failure**：执行 CMS GC 的过程中同时有对象要放入老年代，而此时老年代空间不足（有时候“空间不足”是指 CMS GC 当前的浮动垃圾过多导致暂时性的空间不足），便会报 Concurrent Mode Failure 错误，并触发 Full GC。

# **内存分配策略**
对象的内存分配规则并不是百分百固定的，其细节取决于当前使用的是哪一种垃圾收集器组合，还有虚拟机中与内存有关的参数设置。
- **对象优先在eden分配**
- **大对象直接进入老年代**
- **长期存活对象进入老年代**：Survivor区对象每经历一次Minor GC，年龄就增加1，当年龄达到MaxTenuringThreshold设置的值（默认15），就将会被晋升到老年代。
- **动态对象年龄判定**：如果Survivor空间中相同年龄所有对象大小的总和大于Survivor空间的一半，年龄大于或等于该年龄的对象就可以直接进入老年代。
- **空间分配担保**：在发生Minor GC之前，虚拟机会先检查老年代最大可用的连续空间是否大于新生代所有对象总空间，如果这个条件成立，那么Minor GC就是安全的。如果不成立，则虚拟机会先查看HandlePromotionFailure设置值是否允许担保失败。如果允许，那么会继续检查老年代最大可用的连续空间是否大于历次晋升到老年代对象的平均大小，如果大于，将会尝试进行一次Minor GC，尽管这次Minor GC是有风险的；如果小于，或者HandlePromotionFailure设置不允许冒险，那这时也要改为进行一次Full GC。

# **类加载机制**
### **类加载的时机**
虚拟机规范严格规定了有且只有下面5种情况必须立即对类进行初始化：
- 遇到new、getstatic、putstatic或invokestatic这4条字节码指令时，如果累没有进行过初始化，则需要先触发其初始化。生成这4条指令的最常见的java代码场景：使用new关键字实例化对象，读取或设置一个类的静态字段（被final修饰、已在编译期把结果放入常量池的静态字段除外）、调用一个类的静态方法。

- 使用java.lang.reflect包的方法对类进行反射调用，如果类没有初始化则需要先触发其初始化。
- 当虚拟机加载一个类的时候，父类还没有进行初始化，则需先触发父类初始化。
- 但虚拟机启东市，用户需要指定一个要执行的主类（包含main()方法的那个类），虚拟机会先执行这个主类。
- 当使用JDK1.7的动态语言规则时，如果一个java.lang.invoke.MethodHandle实例最后的解析结果REF_getStatic、REF_putStatic、REF_invokeStatic的方法句柄，并且这个方法所对应的类还没有进行初始化，则需要先触发其初始化。

以下情况不会初始化：
- **通过子类引用父类静态变量**，只有直接定义这个字段的类才会被初始化，因此通过其子类来引用父类中定义的静态字段只会初始化父类。
- **通过数组定义来引用类**。`SuperClass[] sca = new SuperClass[];`。
- **引用常量**，常量在编辑阶段会存入调用类的常用池中，本质上并没有直接引用到定义常量的类。
### **类加载的过程**

![clipboard.png](https://segmentfault.com/img/bVbeIIC)

#### **加载**
在加载阶段，虚拟机需要完成下面三件事：
- 通过一个类的全限定名来获取定义此类的二进制字节流。
- 将这个字节流所代表的静态存储结构转化为方法区的运行时数据结构。
- 在内存中生成一个代表这个类的java.lang.Class对象，作为方法去对这个类的各种数据的访问入口。

#### **验证**
验证是连接阶段的第一步，这一阶段的目的是为了确保Class文件的字节流中包含的信息符合当前的虚拟机要求，并且不会危害虚拟机自身的安全。

从整体看，验证阶段大致上会完成4个阶段的检验动作：
- 文件格式验证：验证字节流是否符合Class文件格式的规范，并且能被当前版本的虚拟机处理。（魔数、主、次版本号等）
- 元数据验证：对字节码描述的信息进行语义分析，验证点：是否有父类、是否继承了不被允许继承的类、如果不是抽象类，是否实现了其父类或接口之中要求实现的所有方法。
- 字节码验证：通过数据流和控制分析确定程序的语义是否合法、符合逻辑。
- 符号引用验证：发生在虚拟机将符号引用转化为直接饮用的时候，这个转化动作在解析阶段发生。符号引用可以看做对类自身以外（常量池中各种符号引用）的信息进行匹配性检验：能够通过字符串描述的全限定名找到对应的类、方法、字段以及访问性能否被当前类访问。

#### **准备**
正式为类变量分配内存并设置初始值的阶段，这些变量所使用的内存都将在方法区中进行分配。同时设置变量的初始值（零值）。
 
#### **解析**
将常量池中的符号引用替换成直接饮用的过程。

- 符号引用：以一组符号来描述所引用的目标，符号可以是任何形式的字面量，只要使用时能无歧义地定位到目标即可。
- 直接引用：直接指向目标的执行、相对偏移量或能间接定位到目标的句柄。

#### **初始化**
开始执行类中定义的java程序代码，根据程序员通过程序制定的主观计划去初始化类变量和其他资源，或者说执行类构造器<clinit>()方法的过程。
- <clinit>()方法和构造函数不同，虚拟机保证子类的<clinit>()方法执行之前，父类的<clinit>()方法已经执行完毕。
- <clinit>()方法并不是必需的，如果累没有静态语句块，也没有对变量的赋值操作，那么编译器可以不为这个类生成<clinit>()方法。
- 接口中不能使用静态语句块，但仍然有变量初始化的赋值操作，因此接口与类一样都会生成<clinit>()方法，但是与类不同的是，执行借口的<clinit>()方法先执行父接口的，只有当父接口定义的变量被使用时，父接口才会初始化。
- 虚拟机保证一个类的<clinit>()方法在多线程环境中被正确的加锁、同步。

#### 源码
- [【JVM源码探秘】细说Class的装载、链接和初始化](https://hunterzhao.io/post/2018/05/17/hotspot-explore-class-loading-linking-and-initializing/)
- [你知道Java类是如何被加载的吗？](https://zhuanlan.zhihu.com/p/60328095)

### **类加载器**
虚拟机设计团队把类加载阶段中“通过一个类的全限定名来获取描述此类的二进制字节流”这个动作放到java虚拟机外部去实现，以便让应用程序自己决定如何获取所需要的类。实现这个动作的代码模块称为“类加载器”。

对于任意一个类，都需要由加载它的加载器和这个类本身确立其在Java虚拟机中的唯一性，每一个类加载器都拥有一个独立的类名称空间。两个类“相等”包括代表类的Class对象的equals()方法、isAssignableFrom()方法、isInstance()方法的返回结果，也包括使用instanceof关键字作对象所属关系判定等情况。

从Java虚拟机的角度来讲，只存在两种不同的类加载器：
- 启动类加载器（Bootstrap ClassLoader）：这个类加载器使用C++语言实现，是虚拟机自身的一部分。
- 所有其他的类加载器：由Java语言实现独立于虚拟机外部，并且全部继承自抽象类java.lang.ClassLoader。

从Java开发人员的角度来看，类加载器划分为更细致一些：
- 启动类加载器（Bootstrp ClassLoader）：负责将存在<JAVA_HOME>\lib目录中的，或者被-Xbootclasspath参数所指定的路径中的，并且是虚拟机识别的（仅按照文件名识别）类库加载到虚拟机内存中。启动类加载器无法被Java程序直接饮用，用户在编写自定义类加载器时，如果需要把加载请求委派给引导类加载器，那直接使用null替代即可。
- 扩展类加载器（Extemsion ClassLoader）：由sun.misc.Launcher$ExtClassLoader实现，负责加载<JAVA_HOME>\lib\ext目录中的，或者被java.ext.dirs系统变量所指定的路径中的所有类库，开发者可以直接使用扩展类加载器。
- 应用程序类加载器（Application ClassLoader）：这个类加载器由sun.misc.Launcher$App-ClassLoader实现。由于这个类加载器是Classloader中的getSystemClassLoader()的返回值，所以一般也称为系统类加载器。负责加载用户路径(ClassPath)上所指定的类库，开发者可以直接使用这个类加载器。如果应用程序没有自定义过自己的类加载器，一般情况下这个就是程序中默认的类加载器。

#### **双亲委派模型**

应用程序都是由三种类加载器互相配合进行加载的，如果有必要还可以加入自己定义的类加载器，这些类加载器之间的关系一般如下：

![clipboard.png](https://segmentfault.com/img/bVbe7vS)

图中展示的类加载器之间的这种层次关系，称为类加载器的双亲委派模型，双亲委派模型除了顶层的启动类加载器外，其余的类加载器都应当有自己的父类加载器，这里加载器之间的父子关系一般不会以继承（Inheritance）的关系来实现，而是都是用组合（Composition）关系来服用父加载器的代码。

##### **工作过程**
如果一个类加载器收到了类加载的请求，它首先会把这个请求委派给父类加载器去完成，每一个层次的类加载器都是如此，因此所有的加载请求最终都应该传送到顶层的启动类加载器中，只有父加载器反馈自己无法尝试完成这个加载请求时，子加载器才会尝试自己去加载。

##### **好处**
Java类随着它的类加载器一起具备了一种带有优先级的层次关系。

例如类java.lang.Object存放在rt.jar中，无论哪一个类加载器要加载这个类，最终都是委派给处于模型最顶端的启动类加载器来进行加载，因此Object类在程序的各种加载器环境中都是一个类。相反如果没有双亲委派模型，如果用户自己编写了一个称为java.lang.Object的类，并放在程序的ClassPath中，那么系统中将会出现多个不同的Object类。

##### **实现**

```
    protected Class<?> loadClass(String name, boolean resolve) throws ClassNotFoundException {
        synchronized (getClassLoadingLock(name)) {
            // First, check if the class has already been loaded
            Class<?> c = findLoadedClass(name);
            if (c == null) {
                try {
                    if (parent != null) {
                        c = parent.loadClass(name, false);
                    } else {
                        c = findBootstrapClassOrNull(name);
                    }
                } catch (ClassNotFoundException e) {
                    // ClassNotFoundException thrown if class not found
                    // from the non-null parent class loader
                }

                if (c == null) {
                    // If still not found, then invoke findClass in order
                    // to find the class.
                    c = findClass(name);
                }
            }
            if (resolve) {
                resolveClass(c);
            }
            return c;
        }
    }
```


  [1]: http://www.importnew.com/26334.html
  [2]: https://my.oschina.net/feichexia/blog/196575
