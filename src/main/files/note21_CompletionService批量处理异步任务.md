#### 利用CompletionService实现询价系统
不过在实际项目中，并不建议你这样做，因为Java SDK并发包里已经提供了设计精良的CompletionService。利用
CompletionService不但能帮你解决先获取到的报价先保存到数据库的问题，而且还能让代码更简练。
CompletionService的实现原理也是内部维护了一个阻塞队列，当任务执行结束就把任务的执行结果加入到阻塞队列中，
不同 的是CompletionService是把任务执行结果的Future对象加入到阻塞队列中，而上面的示例代码是把任务最终的执行结果放入 了阻塞队列中。


```  
// 创建线程池 ExecutorService executor =
Executors.newFixedThreadPool(3); // 创建CompletionService CompletionService<Integer> cs = new
ExecutorCompletionService<>(executor); // 异步向电商S1询价 cs.submit(()->getPriceByS1());
// 异步向电商S2询价 cs.submit(()->getPriceByS2());
// 异步向电商S3询价 cs.submit(()->getPriceByS3()); // 将询价结果异步保存到数据库
for (int i=0; i<3; i++) {
Integer r = cs.take().get();
executor.execute(()->save(r)); }
```



#### 利用CompletionService实现Dubbo中的Forking Cluster

Dubbo中有一种叫做Forking的集群模式，这种集群模式下，支持并行地调用多个查询服务，只要有一个成功返回结果，整个 服务就可以返回了  

```  
geocoder(addr) {
 //并行执行以下3个查询服务，
  r1=geocoderByS1(addr); 
  r2=geocoderByS2(addr); 
  r3=geocoderByS3(addr); 
  //只要r1,r2,r3有一个返回 //则返回
  return r1|r2|r3;
}
```

```
// 创建线程池 ExecutorService executor =
Executors.newFixedThreadPool(3); // 创建CompletionService CompletionService<Integer> cs =
new ExecutorCompletionService<>(executor); // 用于保存Future对象
List<Future<Integer>> futures =
new ArrayList<>(3); //提交异步任务，并保存future到futures futures.add(
cs.submit(()->geocoderByS1())); futures.add(
cs.submit(()->geocoderByS2())); futures.add(
cs.submit(()->geocoderByS3())); // 获取最快返回的任务执行结果
Integer r = 0;
try {
// 只要有一个成功返回，则break for (int i = 0; i < 3; ++i) {
r = cs.take().get(); //简单地通过判空来检查是否成功返回 if (r != null) {
break; }
}
} finally {
//取消所有任务
for(Future<Integer> f : futures)
    f.cancel(true);
}
// 返回结果 return r;
```


#### 总结
当需要批量提交异步任务的时候建议你使用CompletionService。CompletionService将线程池Executor和阻塞队列 BlockingQueue的功能融合在了一起，
能够让批量异步任务的管理更简单。除此之外，CompletionService能够让异步任务的 执行结果有序化，先执行完的先进入阻塞队列，利用这个特性，
你可以轻松实现后续处理的有序性，避免无谓的等待，同时还 可以快速实现诸如Forking Cluster这样的需求。
CompletionService的实现类ExecutorCompletionService，需要你自己创建线程池，虽看上去有些啰嗦，
但好处是你可以让多 个ExecutorCompletionService的线程池隔离，这种隔离性能避免几个特别耗时的任务拖垮整个应用的⻛险。