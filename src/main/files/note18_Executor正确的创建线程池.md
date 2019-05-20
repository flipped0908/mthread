### 线程池是一种生产者-消费者模式

``` 
池，仅用来说明工作原理
class MyThreadPool{
  //利用阻塞队列实现生产者-消费者模式
  BlockingQueue<Runnable> workQueue;
  //保存内部工作线程
  List<WorkerThread> threads 
    = new ArrayList<>();
  // 构造方法
  MyThreadPool(int poolSize, 
    BlockingQueue<Runnable> workQueue){
    this.workQueue = workQueue;
    // 创建工作线程
    for(int idx=0; idx<poolSize; idx++){
      WorkerThread work = new WorkerThread();
      work.start();
      threads.add(work);
    }
  }
  // 提交任务
  void execute(Runnable command){
    workQueue.put(command);
  }
  // 工作线程负责消费任务，并执行任务
  class WorkerThread extends Thread{
    public void run() {
      //循环取任务并执行
      while(true){ ①
        Runnable task = workQueue.take();
        task.run();
      } 
    }
  }  
}

/** 下面是使用示例 **/
// 创建有界阻塞队列
BlockingQueue<Runnable> workQueue = 
  new LinkedBlockingQueue<>(2);
// 创建线程池  
MyThreadPool pool = new MyThreadPool(
  10, workQueue);
// 提交任务  
pool.execute(()->{
    System.out.println("hello");
});
```

#### ThreadPoolExecutor 
```
ThreadPoolExecutor(
                    int corePoolSize, // 线程池中保有的最小线程数
                    int maximumPoolSize，// 线程池创建的最大线程数，
                    long keepAliveTime， 
                    TimeUnit unit,
                    BlockingQueue<Runnable> workQueue,
                    ThreadFactory threadFactory, // 通过这个参数你可以自定义如何创建线程，例如你可以给线程指定一个有意义的名字。
                    RejectedExecutionHandler handler  // 通过这个参数你可以自定义任务的拒绝策略
                                                        CallerRunsPolicy：提交任务的线程自己去执行该任务。
                                                        AbortPolicy：默认的拒绝策略，会throws RejectedExecutionException。
                                                        DiscardPolicy：直接丢弃任务，没有任何异常抛出。
                                                        DiscardOldestPolicy：丢弃最老的任务，其实就是把最早进入工作队列的任务丢弃，然后把新任务加入到工作队列。

                                                        



                    
                    );
```


####  使用线程池要注意些什么

不建议使用Executors的最重要的原因是：Executors提供的很多方法默认使用的都是无界的LinkedBlockingQueue，
高负载情境下，无界队列很容易导致OOM，而OOM会导致所有请求都无法处理，这是致命问题。所以强烈建议使用有界队列。  


使用有界队列，当任务过多时，线程池会触发执行拒绝策略，线程池默认的拒绝策略会throw RejectedExecutionException 这是个运行时异常，
对于运行时异常编译器并不强制catch它，所以开发人员很容易忽略。因此默认拒绝策略要慎重使用。
如果线程池处理的任务非常重要，建议自定义自己的拒绝策略；

> 并且在实际工作中，自定义的拒绝策略往往和降级策略配合使用。


最致命的是任务虽然异常了，但是你却获取不到任何通知，这会让你误以为任务都执行得很正常   
```
try {
  //业务逻辑
} catch (RuntimeException x) {
  //按需处理
} catch (Throwable x) {
  //按需处理
} 

```










































