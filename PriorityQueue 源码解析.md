## 一  PriorityQueue 源码解析

- ##### 1  构造方法

  ​	PriorityQueue提供四种构造方法，包含缺省和显式实现的比较器，前三种构造方法都是调用第四种方法。

  ```java
  public PriorityQueue() {
      this(DEFAULT_INITIAL_CAPACITY, null);
  }
  public PriorityQueue(int initialCapacity) {
      this(initialCapacity, null);
  }
  public PriorityQueue(Comparator<? super E> comparator) {
      this(DEFAULT_INITIAL_CAPACITY, comparator);
  }
  
  public PriorityQueue(int initialCapacity,Comparator<? super E> comparator) {
      if (initialCapacity < 1)
          throw new IllegalArgumentException();
      this.queue = new Object[initialCapacity];
    	this.comparator = comparator;
  
  }
  ```

- ##### 2 主要方法

   1 grow():

  ```java
  private void grow(int minCapacity) {
      int oldCapacity = queue.length;
      // Double size if small; else grow by 50%
      int newCapacity = oldCapacity + ((oldCapacity < 64) ?
                                       (oldCapacity + 2) :
                                       (oldCapacity >> 1));
      // overflow-conscious code
      if (newCapacity - MAX_ARRAY_SIZE > 0)
          newCapacity = hugeCapacity(minCapacity);
      queue = Arrays.copyOf(queue, newCapacity);
  }
  ```

  2  add():

  ```
  public boolean add(E e) {
      return offer(e);
  }
  ```

  3  offer() : Inserts the specified element into this priority queue.

```
public boolean offer(E e) {
    if (e == null)
        throw new NullPointerException();
    modCount++;
    int i = size;
    if (i >= queue.length)
        grow(i + 1);
    size = i + 1;
    if (i == 0)
        queue[0] = e;
    else
        siftUp(i, e);
    return true;
}
```

 	4 peek(): 取队列开头的元素，不剪出

```
public E peek() {
    return (size == 0) ? null : (E) queue[0];
}
```

​	5 poll(): 取出队列开头的元素并剪出

```
public E poll() {
    if (size == 0)
        return null;
    int s = --size;
    modCount++;
    E result = (E) queue[0];
    E x = (E) queue[s];
    queue[s] = null;
    if (s != 0)
        siftDown(0, x);
    return result;
}
```

- 3 优先级队列实现

  1 siftup(): 采用上

  ```
  private void siftUp(int k, E x) {
      if (comparator != null)
          siftUpUsingComparator(k, x);
      else
          siftUpComparable(k, x);
  }
  
  @SuppressWarnings("unchecked")
  private void siftUpComparable(int k, E x) {
      Comparable<? super E> key = (Comparable<? super E>) x;
      while (k > 0) {
          int parent = (k - 1) >>> 1;
          Object e = queue[parent];
          if (key.compareTo((E) e) >= 0)
              break;
          queue[k] = e;
          k = parent;
      }
      queue[k] = key;
  }
  
  @SuppressWarnings("unchecked")
  private void siftUpUsingComparator(int k, E x) {
      while (k > 0) {
          int parent = (k - 1) >>> 1;
          Object e = queue[parent];
          if (comparator.compare(x, (E) e) >= 0)
              break;
          queue[k] = e;
          k = parent;
      }
      queue[k] = x;
  }
  ```

  2  siftdown(): Inserts item x at position k, maintaining heap invariant 

  ```
  private void siftDownComparable(int k, E x) {
      Comparable<? super E> key = (Comparable<? super E>)x;
      
     //零扩展，右移位符，直接给高位加0，相当于/2;
     //移位符只针对int型，其他类型会被强制转化，只移右五位
      int half = size >>> 1;        // loop while a non-leaf
      while (k < half) {
          int child = (k << 1) + 1; // assume left child is least
          Object c = queue[child];
          int right = child + 1;
          if (right < size &&
              ((Comparable<? super E>) c).compareTo((E) queue[right]) > 0)
              c = queue[child = right];
          if (key.compareTo((E) c) <= 0)
              break;
          queue[k] = c;
          k = child;
      }
      queue[k] = key;
  }
  ```
