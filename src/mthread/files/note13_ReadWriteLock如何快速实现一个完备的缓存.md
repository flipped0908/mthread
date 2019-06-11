### ReadAndWriteLock -> cache

```$xslt
class Cache<K,V>{
    final Map<K,V> m = new HaseMap<>();
    final ReadWriteLock rwl = new ReentrantReadWriteLock();
    
    final Lock r = rwl.readLock();
    final Lock w = rwl.writeLock();
    
    v get(K k){
        r.lcok();
        try{
           return map.get(k);
        }finaly{
            r.unlock;
        }
    }
    
    void put(K k,V v){
        w.lock;
        try{
            map.put(k,v);
        }
        finaly{
            w.unlock;
        }
    }


}

```

按需加载缓存

```$xslt
  V get(K key) {
    V v = null;
    // 读缓存
    r.lock();         ①
    try {
      v = m.get(key); ②
    } finally{
      r.unlock();     ③
    }
    // 缓存中存在，返回
    if(v != null) {   ④
      return v;
    }  
    // 缓存中不存在，查询数据库
    w.lock();         ⑤
    try {
      // 再次验证
      // 其他线程可能已经查询过数据库
      v = m.get(key); ⑥
      if(v == null){  ⑦
        // 查询数据库
        v= 省略代码无数
        m.put(key, v);
      }
    } finally{
      w.unlock();
    }
    return v; 
  }


```

升级
```$xslt
// 读缓存
r.lock();         ①
try {
  v = m.get(key); ②
  if (v == null) {
    w.lock();
    try {
      // 再次验证并更新缓存
      // 省略详细代码
    } finally{
      w.unlock();
    }
  }
} finally{
  r.unlock();     ③
}
```

降级
```$xslt
 volatile boolean cacheValid;

 void processCachedData() {
    // 获取读锁
    r.lock();
    if (!cacheValid) {
      // 释放读锁，因为不允许读锁的升级
      r.unlock();
      // 获取写锁
      w.lock();
      try {
        // 再次检查状态  
        if (!cacheValid) {
          data = ...
          cacheValid = true;
        }
        // 释放写锁前，降级为读锁
        // 降级是可以的
        r.lock(); ①
      } finally {
        // 释放写锁
        w.unlock(); 
      }
    }
    // 此处仍然持有读锁
    try {use(data);} 
    finally {r.unlock();}
  }
```

### 总结
 缓存和数据源同步的问题   
 ReadWriteLock 实现了一个简单的缓存，这个缓存虽然解决了缓存的初始化问题，但是没有解决缓存数据与源头数据的同步问题，
 这里的数据同步指的是保证缓存数据和源头数据的一致性。解决数据同步问题的一个最简单的方案就是超时机制。
 所谓超时机制指的是加载进缓存的数据不是长久有效的，而是有时效的，当缓存的数据超过时效，
 也就是超时之后，这条数据在缓存中就失效了。而访问缓存中失效的数据，会触发缓存重新从源头把数据加载进缓存。  
 
 当然也可以在源头数据发生变化时，快速反馈给缓存，但这个就要依赖具体的场景了。
 例如 MySQL 作为数据源头，可以通过近实时地解析 binlog 来识别数据是否发生了变化，
 如果发生了变化就将最新的数据推送给缓存。另外，还有一些方案采取的是数据库和缓存的双写方案。











