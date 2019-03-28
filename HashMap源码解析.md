### HashMap    HashTable源码解析

- 1 类图结构

  ![img](https://img-blog.csdn.net/20170116200932306)

   数据结构 ：

  ![img](https://img-blog.csdn.net/20170116201004040)

  

- 2  HashMapEntry 申明及属性

  Map.Entry<K,V>为  map接口中的entry接口 

  ```
  static class Node<K,V> implements Map.Entry<K,V> {
      final int hash;
      final K key;
      V value;
      Node<K,V> next;
  ```

- 3  常用方法

  ```java
  	HashMap<String, String> hashMap=new HashMap<>();
  	//添加方法
  	hashMap.put("1", "chris");
             //遍历方法1_for
  	Set<String> keys=hashMap.keySet();
  	for(String key:keys){
  		System.out.println(key+"="+hashMap.get(key));
  	}
  	//遍历方法1_iterator(for和iterator实现原理相同)
              Iterator iter = map.keySet().iterator(); 
              while (iter.hasNext()) { 
              String key = iter.next(); 
              String value = map.get(key); 
              } 
  	//遍历方法2_for
              Set<Entry<String, String>> entrys= hashMap.entrySet();
  	for(Entry<String, String> entry:entrys){
  		String key=entry.getKey();
  		String value=entry.getValue();
  	}
  	//遍历方法2_iterator
  	Iterator<Entry<String, String>> iterator=hashMap.entrySet().iterator();
  	while(iterator.hasNext()){
  		Map.Entry<String, String> entry=iterator.next();
  		String key=entry.getKey();
  		String value=entry.getValue();
  	}
  	//查询方法
  	hashMap.get("1");
  	//删除方法
  	hashMap.remove("1");	
  ```

- 4 显式构造方法

  ```
  public HashMap(int initialCapacity, float loadFactor) {  
          //初始容量不能<0  
          if (initialCapacity < 0)  
              throw new IllegalArgumentException("Illegal initial capacity: "  
                      + initialCapacity);  
          //初始容量不能 > 最大容量值，HashMap的最大容量值为2^30  
          if (initialCapacity > MAXIMUM_CAPACITY)  
              initialCapacity = MAXIMUM_CAPACITY;  
          //负载因子不能 < 0  
          if (loadFactor <= 0 || Float.isNaN(loadFactor))  
              throw new IllegalArgumentException("Illegal load factor: "  
                      + loadFactor);  
  ```

  ```java
      // 计算出大于 initialCapacity 的最小的 2 的 n 次方值。  
      int capacity = 1;  
      while (capacity < initialCapacity)  
          capacity <<= 1;  
        
      this.loadFactor = loadFactor;  
      //设置HashMap的容量极限，当HashMap的容量达到该极限时就会进行扩容操作  
      threshold = (int) (capacity * loadFactor);  
      //初始化table数组  
      table = new Entry[capacity];  
      init();  
  }
  ```

- 5 put方法实现

  1. HashMap通过键的hashCode来快速的存取元素。当不同的对象hashCode发生碰撞时，HashMap通过单链表来解决。

     ![img](https://upload-images.jianshu.io/upload_images/5222801-b17f2e0dfb2f8ead.png?imageMogr2/auto-orient/)

2.源码实现

```java
public V put(K key, V value) {
    return putVal(hash(key), key, value, false, true);
}
```

#### 	TIPS:

- final可以修饰变量，被final修饰的变量被赋初始值之后，不能对它重新赋值。

- final可以修饰方法，被final修饰的方法不能被重写。

- final可以修饰类，被final修饰的类不能够被继承。

  (1)   算出键值对在数组中的位置.      i = (n - 1) & hash，将数组长度减1后与运算hash。这个算法就很巧妙了，记得上面说过数组的长度//定是2的幂，即使初始长度非2的幂也会强制转换为2的幂:  

  (2)处理冲突

```java
final V putVal(int hash, K key, V value, boolean onlyIfAbsent,
               boolean evict) {
    Node<K,V>[] tab; Node<K,V> p; int n, i;
    
    //如果tab为空
    if ((tab = table) == null || (n = tab.length) == 0)
        n = (tab = resize()).length;
       //i = (n - 1) & hash，将数组长度减1后与运算hash。这个算法就很巧妙了，记得上面说过数组的长度//定是2的幂，即使初始长度非2的幂也会强制转换为2的幂:  算出键值对在数组中的位置
    
    //如果不冲突，直接插入
    if ((p = tab[i = (n - 1) & hash]) == null)
        tab[i] = newNode(hash, key, value, null);
    //处理冲突的情况
    else {
        Node<K,V> e; K k;
        //key相同，hash值相同 首个位置的节点与插入节点相同
        if (p.hash == hash &&
            ((k = p.key) == key || (key != null && key.equals(k))))
            e = p;
        
        //如果是红黑树的根
        else if (p instanceof TreeNode)
            e = ((TreeNode<K,V>)p).putTreeVal(this, tab, hash, key, value);
        else {
            //如果是链表结点，遍历到链表
            for (int binCount = 0; ; ++binCount) {
                
                //插入末尾
                if ((e = p.next) == null) {
                    p.next = newNode(hash, key, value, null);
                    //如果超过8，将链表转换为红黑树
                    if (binCount >= TREEIFY_THRESHOLD - 1) // -1 for 1st
                        treeifyBin(tab, hash);
                    break;
                }
                
                //找到相同的节点（引用相同或者  哈希值相同）
                if (e.hash == hash &&
                    ((k = e.key) == key || (key != null && key.equals(k))))
                    break;
                
                //循环条件  检索到链尾
                p = e;
            }
        }
        //找到了相同节点e,替换原节点的Value
        if (e != null) { // existing mapping for key
            V oldValue = e.value;
            if (!onlyIfAbsent || oldValue == null)
                e.value = value;
            //在hashmap中，留给linkedhashmap中重写，主要是把结点移到最后
            afterNodeAccess(e);
            return oldValue;
        }
        
    }
    ++modCount;
    if (++size > threshold)
        resize();
    afterNodeInsertion(evict);
    return null;
}
```

3. resize():  扩容后(n-1)的值在高一位多了1，因此原来的链表和红黑树的节点的位置可能出现高一位多了1，所以不能简单的直接将头节点移动到新数组,需要重新计算位置。而哈希值不需要重新计算，所以这可能就是** (n-1)&hash **这个算法的好处吧。

   

   jdk1.8相较1.7增加的优势：

   在移除旧链到新链上时，采用的index方式为  index=(e.hash & oldCap) ,它有两种结果，一个是0，一个是oldCap.

   ​	比如oldCap=8,hash是3，11，19，27时，(e.hash & oldCap)的结果是0，8，0，8，这样3，19组成新的链表，index为3；而11，27组成新的链表，新分配的index为3+8；

   ​	JDK1.7中重写hash是(e.hash & newCap-1)，也就是3，11，19，27对16取余，也是3，11，3，11，和上面的结果一样，但是index为3的链表是19，3，index为3+8的链表是27，11，也就是说1.7中经过resize后数据的顺序变成了倒叙，而1.8没有改变顺序。

   ![img](https://img-blog.csdn.net/20180108191034317?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvc29nYTYxMw==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

   ```java
     if (oldTab != null) {
               for (int j = 0; j < oldCap; ++j) {
                   Node<K,V> e;
                   if ((e = oldTab[j]) != null) {
                       oldTab[j] = null;
                       //链表头节点
                       if (e.next == null)
                           newTab[e.hash & (newCap - 1)] = e; @1
                       //红黑树节点
                       else if (e instanceof TreeNode) @2
                           ((TreeNode<K,V>)e).split(this, newTab, j, oldCap);
                       //将原链表插入新数组
                       else { // preserve order @3
                           
                           Node<K,V> loHead = null, loTail = null;
                           Node<K,V> hiHead = null, hiTail = null;
                           Node<K,V> next;
                           do {
                               next = e.next;
                               //原索引位置
                               //e.hash & oldCap==0 or oldcap
                               if ((e.hash & oldCap) == 0) {
                                   if (loTail == null)
                                       loHead = e;
                                   else
                                       loTail.next = e;
                                   loTail = e;
                               }
                               
                               //原索引+oldcap
                               else {
                                   if (hiTail == null)
                                       hiHead = e;
                                   else
                                       hiTail.next = e;
                                   hiTail = e;
                               }
                           } while ((e = next) != null);
                           
                           //原索引放入bucket
                           if (loTail != null) {
                               loTail.next = null;
                               newTab[j] = loHead;
                           }
                           //原索引+oldcap
                           if (hiTail != null) {
                               hiTail.next = null;
                               newTab[j + oldCap] = hiHead;
                           }
                       }
                   }
               }
           }
   ```

   #### Get算法

```java
   public V get(Object key) {
        Node<K,V> e;
        return (e = getNode(hash(key), key)) == null ? null : e.value;
    }
```

​	取出算法其实就是插入算法的逆向过程，可能直接从table[i]中取走键值对，也可能是红黑树或者链表中的一个节点。

​	**要注意的是只满足equlas相等并不能有效取出元素，还必须满足哈希值相等，所以要考虑重写key的hashCode()方法。**

```java
final Node<K,V> getNode(int hash, Object key) {
        Node<K,V>[] tab; Node<K,V> first, e; int n; K k;
        //如果hash索引到
        if ((tab = table) != null && (n = tab.length) > 0 &&
            (first = tab[(n - 1) & hash]) != null) {
            
            //找到并且是第一个节点
            if (first.hash == hash && // always check first node
                ((k = first.key) == key || (key != null && key.equals(k))))
                return first;
               
             
            if ((e = first.next) != null) {
            //进入红黑树查找
                if (first instanceof TreeNode)
                    return ((TreeNode<K,V>)first).getTreeNode(hash, key);
                   //进入链表中查找
                do {
                    if (e.hash == hash &&
                        ((k = e.key) == key || (key != null && key.equals(k))))
                        return e;
                } while ((e = e.next) != null);
            }
        }
        return null;
    }
```

### HashMap和Hashtable的区别

​	 HashMap和Hashtable都实现了Map接口，但决定用哪一个之前先要弄清楚它们之间的分别。主要的区别有：线程安全性，同步(synchronization)，以及速度。

1. HashMap几乎可以等价于Hashtable，除了HashMap是非synchronized的，并可以接受null(HashMap可以接受为null的键值(key)和值(value)，而Hashtable则不行)。
2. HashMap是非synchronized，而Hashtable是synchronized，这意味着Hashtable是线程安全的，多个线程可以共享一个Hashtable；而如果没有正确的同步的话，多个线程是不能共享HashMap的。Java 5提供了ConcurrentHashMap，它是HashTable的替代，比HashTable的扩展性更好。
3. 另一个区别是HashMap的迭代器(Iterator)是fail-fast迭代器，而Hashtable的enumerator迭代器不是fail-fast的。所以当有其它线程改变了HashMap的结构（增加或者移除元素），将会抛出ConcurrentModificationException，但迭代器本身的remove()方法移除元素则不会抛出ConcurrentModificationException异常。但这并不是一个一定发生的行为，要看JVM。这条同样也是Enumeration和Iterator的区别。
4. 由于Hashtable是线程安全的也是synchronized，所以在单线程环境下它比HashMap要慢。如果你不需要同步，只需要单一线程，那么使用HashMap性能要好过Hashtable。
5. HashMap不能保证随着时间的推移Map中的元素次序是不变的。

### 要注意的一些重要术语：

​	1) sychronized意味着在一次仅有一个线程能够更改Hashtable。就是说任何线程要更新Hashtable时要首先获得同步锁，其它线程要等到同步锁被释放之后才能再次获得同步锁更新Hashtable。

​	2) Fail-safe和iterator迭代器相关。如果某个集合对象创建了Iterator或者ListIterator，然后其它的线程试图“结构上”更改集合对象，将会抛出ConcurrentModificationException异常。但其它线程可以通过set()方法更改集合对象是允许的，因为这并没有从“结构上”更改集合。但是假如已经从结构上进行了更改，再调用set()方法，将会抛出IllegalArgumentException异常。

​	3) 结构上的更改指的是删除或者插入一个元素，这样会影响到map的结构。

```java
public T next() {
    if (modCount != expectedModCount)
        throw new ConcurrentModificationException();
    return nextElement();
}

public void remove() {
    if (!iterator)
        throw new UnsupportedOperationException();
    if (lastReturned == null)
        throw new IllegalStateException("Hashtable Enumerator");
    if (modCount != expectedModCount)
        throw new ConcurrentModificationException();

    synchronized(Hashtable.this) {
        Entry<?,?>[] tab = Hashtable.this.table;
        int index = (lastReturned.hash & 0x7FFFFFFF) % tab.length;

        @SuppressWarnings("unchecked")
        Entry<K,V> e = (Entry<K,V>)tab[index];
        for(Entry<K,V> prev = null; e != null; prev = e, e = e.next) {
            if (e == lastReturned) {
                // This field is used to make iterators on Collection-views of
     // the Hashtable fail-fast.
                modCount++;
                expectedModCount++;
                if (prev == null)
                    tab[index] = e.next;
                else
                    prev.next = e.next;
                count--;
                lastReturned = null;
                return;
            }
        }
        throw new ConcurrentModificationException();
    }
}
```