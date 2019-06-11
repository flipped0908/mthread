

将一个类所有的属性都设置成final的，并且只允许存在只读方法，那么这个类基 本上就具备不可变性了。  
类和属性都是final的，所有方法均是只读的。  

#### 利用享元模式避免创建重复对象
如果你熟悉面向对象相关的设计模式，相信你一定能想到享元模式(Flyweight Pattern)。利用享元模式可以减少创建对象
的数量，从而减少内存占用。Java语言里面Long、Integer、Short、Byte等这些基本数据类型的包装类都用到了享元模式。  

享元模式本质上其实就是一个对象池，利用享元模式创建对象的逻辑也很简单:创建之前，首先去对象池里看看是不是存在; 
如果已经存在，就利用对象池里的对象;如果不存在，就会新创建一个对象，并且把这个新创建出来的对象放进对象池里。

#### 总结
利用Immutability模式解决并发问题，也许你觉得有点陌生，其实你天天都在享受它的战果。
Java语言里面的String和Long、 Integer、Double等基础类型的包装类都具备不可变性，这些对象的线程安全性都是靠不可变性来保证的。
Immutability模式是 最简单的解决并发问题的方法，建议当你试图解决一个并发问题时，可以首先尝试一下Immutability模式，看是否能够快速解 决。
具备不变性的对象，只有一种状态，这个状态由对象内部所有的不变属性共同决定。其实还有一种更简单的不变性对象，那就 是无状态。  
无状态对象内部没有属性，只有方法。除了无状态的对象，你可能还听说过无状态的服务、无状态的协议等等。无 状态有很多好处，
最核心的一点就是性能。在多线程领域，无状态对象没有线程安全问题，无需同步处理，自然性能很好;在 分布式领域，
无状态意味着可以无限地水平扩展，所以分布式领域里面性能的瓶颈一定不是出在无状态的服务节点上。


#### 类似RPC的Router的实现代码如下所示
```
public final class Router{

    private final String ip; 
    private final Integer port; 
    private final String iface; //构造函数
    public Router(String ip,
    Integer port, String iface){
        this.ip = ip;
        this.port = port;
        this.iface = iface;
    }
    //重写equals方法
    public boolean equals(Object obj){
        if (obj instanceof Router) { 
            Router r = (Router)obj;
             return iface.equals(r.iface) &&ip.equals(r.ip) && port.equals(r.port);
        }
        return false;
    }
    
    public int hashCode() { //省略hashCode相关代码
    }
}
    //路由表信息
public class RouterTable {
    //Key:接口名
    //Value:路由集合
    ConcurrentHashMap<String, CopyOnWriteArraySet<Router>>
    rt = new ConcurrentHashMap<>(); //根据接口名获取路由表

    public Set<Router> get(String iface){
        return rt.get(iface); 
    }
    //删除路由
    public void remove(Router router) {
        Set<Router> set=rt.get(router.iface); 
        if (set != null) {
            set.remove(router); 
        }
    }
    //增加路由
    public void add(Router router) {
        Set<Router> set = rt.computeIfAbsent( route.iface, r -> new CopyOnWriteArraySet<>()); 
        set.add(router);
    } 
}


```




####


目前Copy-on-Write在Java并发编程领域知名度不是很高，很多人都在无意中把它忽视了，但其实Copy-on-Write才是最简单的 并发解决方案。
它是如此简单，以至于Java中的基本数据类型String、Integer、Long等都是基于Copy-on-Write方案实现的。
Copy-on-Write是一项非常通用的技术方案，在很多领域都有着广泛的应用。不过，它也有缺点的，那就是消耗内存，每次修 改都需要复制一个新的对象出来，
好在随着自动垃圾回收(GC)算法的成熟以及硬件的发展，这种内存消耗已经渐渐可以接 受了。所以在实际工作中，如果写操作非常少，
那你就可以尝试用一下Copy-on-Write，效果还是不错的。








