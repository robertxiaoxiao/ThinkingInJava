ConcurrentHashMap源码解析

- 底层结构

  ​	利用 ==CAS + synchronized== 来保证并发更新的安全 
  底层使用==数组+链表+红黑树==来实现 。

![img](https://img-blog.csdn.net/20180327170909484?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3Byb2dyYW1tZXJfYXQ=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

- 重要成员变量

  table：默认为null，初始化发生在第一次插入操作，默认大小为16的数组，用来存储Node节点数据，扩容时大小总是2的幂次方。
  nextTable：默认为null，扩容时新生成的数组，其大小为原数组的两倍。
  sizeCtl ：默认为0，用来控制table的初始化和扩容操作，具体应用在后续会体现出来。 
  	-1 代表table正在初始化 。
  	-N 表示有N-1个线程正在进行扩容操作 。
  其余情况： 
  1、如果table未初始化，表示table需要初始化的大小。 
  2、如果table初始化完成，表示table的容量，默认是table大小的0.75倍，居然用这个公式算0.75（n - (n >>> 2)）。
  Node：保存key，value及key的hash值的数据结构。 

  ###### 其中value和next都用volatile修饰，保证并发的可见性。

```java
class Node<K,V> implements Map.Entry<K,V> {
    final int hash;
    final K key;
    volatile V val;
    volatile Node<K,V> next;
    //... 省略部分代码

}
```

- ForwardingNode：一个特殊的Node节点，hash值为-1，其中存储nextTable的引用。 
  只有table发生扩容的时候，ForwardingNode才会发挥作用，作为一个占位符放在table中表示当前节点为null或则已经被移动。

```java
final class ForwardingNode<K,V> extends Node<K,V> {
    final Node<K,V>[] nextTable;
    ForwardingNode(Node<K,V>[] tab) {
        super(MOVED, null, null, null);
        this.nextTable = tab;
    }
    }
```

#### 2 实例初始化

​	实例化ConcurrentHashMap时倘若声明了table的容量，在初始化时会根据参数调整table大小，==确保table的大小总是2的幂次方==。默认的table大小为16.

​	table的初始化操作回延缓到第一put操作再进行，并且初始化只会执行一次。

```java
private final Node<K,V>[] initTable() {
    Node<K,V>[] tab; int sc;
    while ((tab = table) == null || tab.length == 0) {
//如果一个线程发现sizeCtl<0，意味着另外的线程执行CAS操作成功，当前线程只需要让出cpu时间片
        if ((sc = sizeCtl) < 0) 
            
            /**
            ***thread类的静态方法
            **1. Thread.currentThread()：返回当前正在执行线程对象的引用
            **线程类的构造方法、静态块是被main线程调用的，而线程类的run()方法才是应用线程自己调用的。
            **2.Thread.sleep(2000)：让当前正在执行的线程休眠，但是不会释放资源的锁
           **3.Thread.yield(); 暂停当前执行的线程对象，并执行其他线程，放弃CPU资源但是时间待定
           **4.Thread.interrupt(); 测试当前线程是否已经中断
           */
            Thread.yield(); // lost initialization race; just spin
        else if (U.compareAndSwapInt(this, SIZECTL, sc, -1)) {
            try {
                if ((tab = table) == null || tab.length == 0) {
                    int n = (sc > 0) ? sc : DEFAULT_CAPACITY;
                    @SuppressWarnings("unchecked")
                    Node<K,V>[] nt = (Node<K,V>[])new Node<?,?>[n];
                    table = tab = nt;
                    sc = n - (n >>> 2);  //0.75*capacity
                }
            } finally {
                sizeCtl = sc;
            }
            break;
        }
    }
    return tab;
}
```

#### 3 重要操作

- put操作

  假设table已经初始化完成，put操作采用==CAS+synchronized==实现并发插入或更新操作： 
  - 当前bucket为空时，使用CAS操作，将Node放入对应的bucket中。 
  - 出现hash冲突，则采用synchronized关键字。倘若当前hash对应的节点是链表的头节点，遍历链表，若找到对应的node节点，则修改node节点的val，否则在链表末尾添加node节点；倘若当前节点是红黑树的根节点，在树结构上遍历元素，更新或增加节点。 
  - 倘若当前map正在扩容f.hash == MOVED， 则跟其他线程一起进行扩容

```java
final V putVal(K key, V value, boolean onlyIfAbsent) {
        if (key == null || value == null) throw new NullPointerException();
        int hash = spread(key.hashCode());
        int binCount = 0;
        for (Node<K,V>[] tab = table;;) {
            Node<K,V> f; int n, i, fh;
            if (tab == null || (n = tab.length) == 0)
                tab = initTable();  // lazy Initialization
            
            //hash 索引为i=(n-1)&key.hash 
    //        static final <K,V> Node<K,V> tabAt(Node<K,V>[] tab, int i) {
      //  return (Node<K,V>)U.getObjectVolatile(tab, ((long)i << ASHIFT) + ABASE);}
    //采用Unsafe.getObjectVolatie()来获取，而不是直接用table[index]的原因跟ConcurrentHashMap的//弱一致性有关。在java内存模型中，我们已经知道每个线程都有一个工作内存，里面存储着table的副本，虽然table是//volatile修饰的，但不能保证线程每次都拿到table中的最新元素，Unsafe.getObjectVolatile可以直接获取指定//内存的数据，保证了每次拿到数据都是最新的。

            else if ((f = tabAt(tab, i = (n - 1) & hash)) == null) {  // 当前bucket为空
                if (casTabAt(tab, i, null,
                             new Node<K,V>(hash, key, value, null)))
                    break;                   // no lock when adding to empty bin
            }
            else if ((fh = f.hash) == MOVED)  // 当前Map在扩容，先协助扩容，在更新值。
                tab = helpTransfer(tab, f); 
            else {  // hash冲突
                V oldVal = null;
                //将当前更新节点加锁，确保并发的成功
                synchronized (f) {
                    if (tabAt(tab, i) == f) {  // 链表头节点
                        if (fh >= 0) {
                            binCount = 1;
                            for (Node<K,V> e = f;; ++binCount) {
                                K ek;
                                if (e.hash == hash &&   // 节点已经存在，修改链表节点的值
                                    ((ek = e.key) == key ||
                                     (ek != null && key.equals(ek)))) {
                                    oldVal = e.val;
                                    if (!onlyIfAbsent)
                                        e.val = value;
                                    break;
                                }
                                Node<K,V> pred = e;
                                if ((e = e.next) == null) { // 节点不存在，添加到链表末尾
                                    pred.next = new Node<K,V>(hash, key,
                                                              value, null);
                                    break;
                                }
                            }
                        }
                        else if (f instanceof TreeBin) { // 红黑树根节点
                            Node<K,V> p;
                            binCount = 2;
                            if ((p = ((TreeBin<K,V>)f).putTreeVal(hash, key,
                                                           value)) != null) {
                                oldVal = p.val;
                                if (!onlyIfAbsent)
                                    p.val = value;
                            }
                        }
                    }
                }
                if (binCount != 0) {
                    if (binCount >= TREEIFY_THRESHOLD)  //链表节点超过了8，链表转为红黑树
                        treeifyBin(tab, i);
                    if (oldVal != null)
                        return oldVal;
                    break;
                }
            }
        }
        addCount(1L, binCount);  // 统计节点个数，检查是否需要resize
        return null;

 }
```

红黑树相关：https://www.jianshu.com/p/23b84ba9a498

https://blog.csdn.net/u011240877/article/details/53329023

![1552227055156](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\1552227055156.png)

