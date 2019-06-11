####  ThreadLocal的使用方法 
下面这个静态类ThreadId会为每个线程分配一个唯一的线程Id，如果一个线程前后两次调用ThreadId的get()方法，
两次get()方 法的返回值是相同的。但如果是两个线程分别调用ThreadId的get()方法，那么两个线程看到的get()方法的返回值是不同的。  

####    ThreadLocal与内存泄露

在线程池中使用ThreadLocal为什么可能导致内存泄露呢?原因就出在线程池中线程的存活时间太⻓，往往都是和程序同生共 死的，
这就意味着Thread持有的ThreadLocalMap一直都不会被回收，再加上ThreadLocalMap中的Entry对ThreadLocal是弱引 用(WeakReference)，
所以只要ThreadLocal结束了自己的生命周期是可以被回收掉的。但是Entry中的Value却是被Entry强 引用的，所以即便Value的生命周期结束了，
Value也是无法被回收的，从而导致内存泄露。
那在线程池中，我们该如何正确使用ThreadLocal呢?其实很简单，既然JVM不能做到自动释放对Value的强引用，那我们手动 释放就可以了。
如何能做到手动释放呢?估计你⻢上想到try{}finally{}方案了，这个简直就是手动释放资源的利器。示例的代 码如下，你可以参考学习。

```  
ExecutorService es; ThreadLocal tl; es.execute(()->{
//ThreadLocal增加变量 tl.set(obj);
try {
// 省略业务逻辑代码 }finally {
//手动清理ThreadLocal
    tl.remove();
  }
});
```
#### 总结
线程本地存储模式本质上是一种避免共享的方案，由于没有共享，所以自然也就没有并发问题。如果你需要在并发场景中使用 一个线程不安全的工具类，
最简单的方案就是避免共享。避免共享有两种方案，一种方案是将这个工具类作为局部变量使用， 另外一种方案就是线程本地存储模式。
这两种方案，局部变量方案的缺点是在高并发场景下会频繁创建对象，而线程本地存储 方案，每个线程只需要创建一个工具类的实例，所以不存在频繁创建对象的问题。
线程本地存储模式是解决并发问题的常用方案，所以Java SDK也提供了相应的实现:ThreadLocal。通过上面我们的分析，你 应该能体会到Java 
SDK的实现已经是深思熟虑了，不过即便如此，仍不能尽善尽美，例如在线程池中使用ThreadLocal仍可能
导致内存泄漏，所以使用ThreadLocal还是需要你打起精神，足够谨慎。