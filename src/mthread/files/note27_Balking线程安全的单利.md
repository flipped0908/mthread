

需要快速放弃的一个最常⻅的例子是各种编辑器提供的自动保存功能。自动保存功能的实现逻辑一般都是隔一定时间自动执行 存盘操作，
存盘操作的前提是文件做过修改，如果文件没有执行过修改操作，就需要快速放弃存盘操作。  

``` 

//文件是否被修改过
boolean changed=false; //定时任务线程池 
ScheduledExecutorService ses = Executors.newSingleThreadScheduledExecutor(); //定时执行自动保存
void startAutoSave(){
    ses.scheduleWithFixedDelay(()->{ autoSave();
}, 5, 5, TimeUnit.SECONDS); }
//自动存盘操作 
void autoSave(){
    if (!changed) {
      return;
    }
    changed = false; //执行存盘操作
     //省略且实现 
     this.execSave();
}
//编辑操作 
void edit(){
    //省略编辑逻辑 ......
    changed = true;
}

```

因为对共享变量changed的读写没有使用同步，那如何 保证AutoSaveEditor的线程安全性呢?  


###  Balking模式的经典实现 

Balking模式本质上是一种规范化地解决“多线程版本的if”的方案  

``` 
boolean changed=false; //自动存盘操作
void autoSave(){
    synchronized(this){ 
        if (!changed) {
             return; 
         }
        changed = false;
    }
    //执行存盘操作 //省略且实现 
    this.execSave();
}
//编辑操作 
void edit(){
    //省略编辑逻辑 ...... 
    change();
}
//改变状态
void change(){
    synchronized(this){ 
        changed = true;
    } 
    }

```

####  用volatile实现Balking模式

前面我们用synchronized实现了Balking模式，这种实现方式最为稳妥，建议你实际工作中也使用这个方案。不过在某些特定
场景下，也可以使用volatile来实现，_**但使用volatile的前提是对原子性没有要求**_。  


有一个RPC框架路由表的案例，在RPC框架中，本地路由表是要和 注册中心进行信息同步的，应用启动的时候，
会将应用依赖服务的路由表从注册中心同步到本地路由表中，如果应用重启的时 候注册中心宕机，那么会导致该应用依赖的服务均不可用，
因为找不到依赖服务的路由表。为了防止这种极端情况出 现，RPC框架可以将本地路由表自动保存到本地文件中，
如果重启的时候注册中心宕机，那么就从本地文件中恢复重启前的 路由表。这其实也是一种降级的方案。


```   
//路由表信息
public class RouterTable {
    //Key:接口名
    //Value:路由集合
    ConcurrentHashMap<String, CopyOnWriteArraySet<Router>>
    rt = new ConcurrentHashMap<>();
    
    //路由表是否发生变化
    volatile boolean changed;
    
    //将路由表写入本地文件的线程池 ScheduledExecutorService ses=
    Executors.newSingleThreadScheduledExecutor();
    
    //启动定时任务
    //将变更后的路由表写入本地文件
    public void startLocalSaver(){
    ses.scheduleWithFixedDelay(()->{ autoSave();
      }, 1, 1, MINUTES);
    }
    
    //保存路由表到本地文件
    void autoSave() { 
        if (!changed) {
            return; 
        }
        changed = false; 
        //将路由表写入本地文件 
        //省略其方法实现 
        this.save2Local();
    }
    
    //删除路由
    public void remove(Router router) {
        Set<Router> set=rt.get(router.iface); 
        if (set != null) {
            set.remove(router); 
            //路由表已发生变化 
            changed = true;
        } 
    }
    //增加路由
    public void add(Router router) {
        Set<Router> set = rt.computeIfAbsent( route.iface, r ->
        new CopyOnWriteArraySet<>()); set.add(router);
        //路由表已发生变化
        changed = true;
    }

```



###   Balking模式有一个非常典型的应用场景就是单次初始化，
下面的示例代码是它的实现。这个实现方案中，
我们将init()声明为 一个同步方法，这样同一个时刻就只有一个线程能够执行init()方法;
init()方法在第一次执行完时会将inited设置为true，
这样后 续执行init()方法的线程就不会再执行doInit()了。

```  

class InitTest{
    boolean inited = false; 
    synchronized void init(){
        if(inited){
            return;
        } 
        //省略doInit的实现 
        doInit(); 
        inited=true;
    } 
}


class Singleton{
    private static
    Singleton singleton; //构造方法私有化
    private Singleton(){}; //获取实例(单例)
    public synchronized static Singleton getInstance(){
        if(singleton == null)
        { 
            singleton=new Singleton();
        }
    return singleton;
  }
}



class Singleton{
    private static volatile
    Singleton singleton; //构造方法私有化
    private Singleton() {} //获取实例(单例)
    public static Singleton getInstance() {
    //第一次检查 if(singleton==null){
        synchronize(Singleton.class){
            //获取锁后二次检查 
            if(singleton==null){
                singleton=new Singleton(); }
            }
         }
    return singleton;
  }
}

```

#####   总结
Balking模式和Guarded Suspension模式从实现上看似乎没有多大的关系，Balking模式只需要用互斥锁就能解决，
而Guarded Suspension模式则要用到管程这种高级的并发原语;
但是从应用的⻆度来看，它们解决的都是“线程安全的if”语义，不同之处 在于，
Guarded Suspension模式会等待if条件为真，而Balking模式不会等待。
Balking模式的经典实现是使用互斥锁，你可以使用Java语言内置synchronized，
也可以使用SDK提供Lock;如果你对互斥锁 的性能不满意，可以尝试采用volatile方案，不过使用volatile方案需要你更加谨慎。
当然你也可以尝试使用双重检查方案来优化性能，双重检查中的第一次检查，完全是出于对性能的考量:避免执行加锁操作，
因为加锁操作很耗时。而加锁之后的二次检查，则是出于对安全性负责。双重检查方案在优化加锁性能方面经常用到，
例 如《 ReadWriteLock:如何快速实现一个完备的缓存?》中实现缓存按需加载功能时，也用到了双重检查方案。





####  什么是多线程版本的if   

在自动保存文件的案例中， 全局的共享变量就是一个状态变量，
业务逻辑依赖于这个状态变量的状态，当满足某个状态的时候就执行某个业务逻辑，
放到多线程的场景里，这种 多线程的版本if 的应用场景还是很多的，有人把它总结为一种设计模式叫做balking模式   







































