## 互斥锁 原子性问题

> 同一时刻只有一个线程执行 叫做互斥

![](./img/03-01.png)
  
我们通常能想到 加锁 解锁  的模型  
> 我们忽略的是我们 锁住的是什么， 我们保护的又是什么

改进后的锁模型
![](./img/03-02.png)  
  

java中常用的synchronize用法
```$xslt
classX{
    synchronized void foo(){
        // 临界区
    }
    
    synchronized static void bar(){
        // 临界区
    }
    
    Object obj = new Object();
    void baz(){
    
        synchronized(obj){
            // 临界区
        }
        
    }
}
```

> 管程中的锁规则 解锁happen-before后续对这个锁的加锁  
---

> 受保护资源和锁的关系是N:1的关系  一把锁可以保护多个资源， 多把锁不能保护同一个资源


锁保护资源的确定

对比：  
```$xslt
class SafeCalc {
  long value = 0L;
  synchronized long get() {
    return value;
  }
  synchronized void addOne() {
    value += 1;
  }
}
```

![](./img/03-03.png)

都是 this 锁 可以保护同一块资源 解决原子性问题

```$xslt
class SafeCalc {
  static long value = 0L;
  synchronized long get() {
    return value;
  }
  synchronized static void addOne() {
    value += 1;
  }
}
```
 ![](./img/03-04.png)  
 
 一个是对象锁，一个是类锁 保护的资源不一致


#### 总结
synchronized 是 java里提供锁的语义

锁一定要有有个锁住额对象，锁定的对象要锁住的资源和在哪里加解锁就是设计层面的事情了




































