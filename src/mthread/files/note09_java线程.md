## 线程的生命周期

初始状态 而在操作系统层面，真正的线程还没有创建。  
可运行状态 真正的操作系统线程已经被成功创建了，所以可以分配 CPU 执行  
运行状态 当有空闲的 CPU 时，操作系统会将其分配给一个处于可运行状态的线程  
休眠状态 休眠状态的线程永远没有机会获得 CPU 使用权    
终止状态  

NEW（初始化状态）  
RUNNABLE（可运行 / 运行状态）  
BLOCKED（阻塞状态）  
WAITING（无时限等待）  
TIMED_WAITING（有时限等待）  
TERMINATED（终止状态）  

#### 1. RUNNABLE 与 BLOCKED 的状态转换
只有一种场景会触发这种转换，就是线程等待 synchronized 的隐式锁

#### 2. RUNNABLE 与 WAITING 的状态转换
第一种场景，获得 synchronized 隐式锁的线程，调用无参数的 Object.wait() 方法  
第二种场景，调用无参数的 Thread.join() 方法。
第三种场景，调用 LockSupport.park() 方法

#### 3. RUNNABLE 与 TIMED_WAITING 的状态转换
调用带超时参数的 Thread.sleep(long millis) 方法；  
获得 synchronized 隐式锁的线程，调用带超时参数的 Object.wait(long timeout) 方法；  
调用带超时参数的 Thread.join(long millis) 方法；  
调用带超时参数的 LockSupport.parkNanos(Object blocker, long deadline) 方法；  
调用带超时参数的 LockSupport.parkUntil(long deadline) 方法。  

#### 4. 从 NEW 到 RUNNABLE 状态
1 run()  
2 start()  

#### 5. 从 RUNNABLE 到 TERMINATED 状态
例如 run() 方法访问一个很慢的网络，我们等不下去了，想终止怎么办呢？Java 的 Thread 类里面倒是有个 stop() 方法，
不过已经标记为 @Deprecated，所以不建议使用了。正确的姿势其实是调用 interrupt() 方法。  

如果其他线程调用线程 A 的 interrupt() 方法，那么线程 A 可以通过 isInterrupted() 方法，检测是不是自己被中断了。

###总结
理解 Java 线程的各种状态以及生命周期对于诊断多线程 Bug 非常有帮助，
多线程程序很难调试，出了 Bug 基本上都是靠日志，靠线程 dump 来跟踪问题，
分析线程 dump 的一个基本功就是分析线程状态，大部分的死锁、饥饿、活锁问题都需要跟踪分析线程的状态。  

可以通过 jstack 命令
或者Java VisualVM这个可视化工具将 JVM 所有的线程栈信息导出来，
完整的线程栈信息不仅包括线程的当前状态、调用栈，还包括了锁的信息


## java中创建多少线程才是合适的 
对于 CPU 密集型的计算场景，理论上“线程的数量 =CPU 核数”就是最合适的。不过在工程上，线程的数量一般会设置为“CPU 核数 +1”    
对于  密集型的计算场景，最佳线程数 =CPU 核数 * [ 1 +（I/O 耗时 / CPU 耗时）]  I/O
对于 I/O 密集型计算场景，I/O 耗时和 CPU 耗时的比值是一个关键参数，不幸的是这个参数是未知的，而且是动态变化的，
所以工程上，我们要估算这个参数，然后做各种不同场景下的压测来验证我们的估计


## 为什么局部变量是线程安全的

局部变量存哪里？    
局部变量就是放到了调用栈里  

每个线程都有自己独立的调用栈

-------
##
题外话：  
很多技术的实现原理我都是靠推断，然后看源码验证，而不是像以前一样纯粹靠看源码来总结了。

