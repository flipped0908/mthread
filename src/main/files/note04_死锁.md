## 死锁 怎么解决

如果同类锁实现转账，就把两个操作串行化了，这样如如果数据量特别大的话，会产生性能问题

eg.  
在现实世界中 转账是两个账本， worker去哪账本，2个都在 ，只有一个在 ，一个都不在 3中情况 

```$xslt
Class Account{
    private int balance;
    void transfer(Account target, int amt){
        synchronized(this){
            synchronized(target.class){
                if(this.blance>amt){
                this.balance -= amt;
                target.blance += amt;
                }
            }
        }
    }
}
```

使用细粒度锁可以提高并行化的问题，但是又带来了死锁的问题

### 如何预防死锁

并发程序一旦死锁，一般没有特别好的办法，只能重新启动应用，因此解决死锁的问题，最好还是规避死锁  

> Coffman 总结了发生死锁的4个条件  
    1 互斥， 共享资源 X和Y只能被一个线程占用  
    2 占有且等待， 线程T1已取得共享资源X，在等待共享资源Y的时候，不释放共享资源X；  
    3 不可抢占， 其他线程不能强行抢占线程T1占有的资源  
    4 循环等待  
    
    
#### 1 破坏占用且等待的条件   

增加账本管理员 一次申请多有资源

`
class Allocator {
  private List<Object> als =
    new ArrayList<>();
  // 一次性申请所有资源
  synchronized boolean apply(
    Object from, Object to){
    if(als.contains(from) ||
         als.contains(to)){
      return false;  
    } else {
      als.add(from);
      als.add(to);  
    }
    return true;
  }
  // 归还资源
  synchronized void free(
    Object from, Object to){
    als.remove(from);
    als.remove(to);
  }
}
 
class Account {
  // actr 应该为单例
  private Allocator actr;
  private int balance;
  // 转账
  void transfer(Account target, int amt){
    // 一次性申请转出账户和转入账户，直到成功
    while(!actr.apply(this, target))
      ；
    try{
      // 锁定转出账户
      synchronized(this){              
        // 锁定转入账户
        synchronized(target){           
          if (this.balance > amt){
            this.balance -= amt;
            target.balance += amt;
          }
        }
      }
    } finally {
      actr.free(this, target)
    }
  } 
}

`


#### 2破坏不可抢占的条件

核心是主动释放占有的资源
> synchronizded 申请资源的时候如果申请不到 线程会进入阻塞状态，什么也做不了，也不释放资源  
    java.util.concurrent 这个包下提供的Lock是可以轻松解决这个问题的  
    
 发现死锁设置超时时间
 

#### 3 破坏循环等待的条件

需要对资源进行排序，让后按序申请资源

1~6处的代码对账户this和target排序，然后按照序号从小到大的顺序锁定账户
`
class Account {
  private int id;
  private int balance;
  // 转账
  void transfer(Account target, int amt){
    Account left = this        ①
    Account right = target;    ②
    if (this.id > target.id) { ③
      left = target;           ④
      right = this;            ⑤
    }                          ⑥
    // 锁定序号小的账户
    synchronized(left){
      // 锁定序号大的账户
      synchronized(right){ 
        if (this.balance > amt){
          this.balance -= amt;
          target.balance += amt;
        }
      }
    }
  } 
}

`






























