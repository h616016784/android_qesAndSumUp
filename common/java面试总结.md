# 一、java面试题总览
参考地址：[Java面试题全集（上）](https://blog.csdn.net/jackfrued/article/details/44921941)

# 二、java自己查看的知识
## 1、String类的Intern()方法
参考地址[String类的Intern()方法](https://blog.csdn.net/as1072966956/article/details/82051111)
## 2、java的gc原理
参考地址[java中GC的工作原理](https://www.cnblogs.com/diaozhaojian/p/10510608.html)
## 3、java的hashcode和equals方法
参考地址[java重写hashcode方法那点事](https://blog.csdn.net/zhengchao1991/article/details/78916471)
## 4、java集合方面总计
### 1） hashmap
参考地址[hashmap与hashtable的区别](https://www.jianshu.com/p/939b8a672070)

参考地址：[ConcurrentHashMap原理探究](https://www.cnblogs.com/huangjuncong/p/9478505.html)
### 2） LinkedHashMap
参考地址:[LinkedHashMap详解](https://blog.csdn.net/qq_26857649/article/details/81713420)

### 3） 红黑树
参考地址:[红黑树原理详解](https://blog.csdn.net/liushengxi_root/article/details/86073971)

## 5、java多线程方面总结
参考地址[java多线程面试和答案](https://blog.csdn.net/ll666634/article/details/78615505)

同步方法synchronized和Lock的差异，参考地址[java中synchronized与lock的理解与应用](https://www.cnblogs.com/hirampeng/p/9208242.html)
   - 在线程持有读锁的情况下，该线程不能取得写锁(因为获取写锁的时候，如果发现当前的读锁被占用，就马上获取失败，不管读锁是不是被当前线程持有)。

   - 在线程持有写锁的情况下，该线程可以继续获取读锁（获取读锁时如果发现写锁被占用，只有写锁没有被当前线程占用的情况才会获取失败）。

   - 仔细想想，这个设计是合理的：因为当线程获取读锁的时候，可能有其他线程同时也在持有读锁，因此不能把获取读锁的线程“升级”为写锁；而对于获得写锁的线程，它一定独占了读写锁，因此可以继续让它获取读锁，当它同时获取了写锁和读锁后，还可以先释放写锁继续持有读锁，这样一个写锁就“降级”为了读锁。

 - 综上：一个线程要想同时持有写锁和读锁，必须先获取写锁再获取读锁；写锁可以“降级”为读锁；读锁不能“升级”为写锁
## 6、java反射方面总结
## 7、java的socket方面使用
## 8、
