# ReentrantLock 源码解析

**可重入锁:** 也叫做递归锁，指的是同一线程 外层函数获得锁之后 ，内层递归函数仍然有获取该锁的代码，但不受影响。

在JAVA中ReentrantLock 和synchronized 都是可重入锁；

```java
*  <pre> {@code
* class X {
*   private final ReentrantLock lock = new ReentrantLock();
*   // ...
*
*   public void m() {
*     lock.lock();  // block until condition holds
*     try {
*       // ... method body
*     } finally {
*       lock.unlock()
*     }
*   }
* }}</pre>
*
```