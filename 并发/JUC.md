# juc  笔记

> juc： java.util.concurrent.*


# juc atomic  
利用CAS算法实现线程安全的

AtomicInteger


# juc包下的集合类
ReentrantLock 
ReentrantReadWriteLock 
CountDownLatch
CyclicBarrier
Semaphore
Lock、condition


aqs 队列中的第一个node中thread总是为空。

waitStatus 在解锁过程中起着很大的作用，在加入队列时都是后一个线程将前一个线程的
stats改为-1.

一个线程时不会初始化aqs队列，线程交替执行时也不会初始化aqs队列。



参考[https://blog.csdn.net/java_lyvee/article/details/98966684]
