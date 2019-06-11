
通过Copy-on-Write这两个容器实现的读操作是无锁的，由于 无锁，所以将读操作的性能发挥到了极致。

类Unix的操作系统中创建进程的API是fork()，传统的fork()函数会创建 父进程的一个完整副本，例如父进程的地址空间现在用到了1G的内存，
那么fork()子进程的时候要复制父进程整个进程的地址 空间(占有1G内存)给子进程，这个过程是很耗时的。而Linux中的fork()函数就聪明得多了，
fork()子进程的时候，并不复制 整个进程的地址空间，而是让父子进程共享同一个地址空间;只用在父进程或者子进程需要写入的时候才会复制地址空间，
从 而使父子进程拥有各自的地址空间。
本质上来讲，父子进程的地址空间以及数据都是要隔离的，使用Copy-on-Write更多地体现的是一种延时策略，
只有在真正需 要复制的时候才复制，而不是提前复制好，


很多文件系统也同样用到了，例如Btrfs (B-Tree File System)、aufs(advanced multi-layered unification filesystem)等。
Docker容器镜像的设计是 Copy-on-Write，
甚至分布式源码管理系统Git背后的设计思想都有Copy-on-Write

Copy-on-Write最大的应用领域还是在函数式编程领域
Copy-on-Write也是可以按需复制的，如果你感兴趣可以参考Purely Functional Data Structures这本书，里面描述了各种 具备不变性的数据结构的实现。


> 对读的性能要求很高，读多写少，弱一致性


#####

