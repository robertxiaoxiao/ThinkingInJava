## Threadlocal源码解析

​	当使用ThreadLocal维护变量时，ThreadLocal为每个使用该变量的线程提供独立的变量副本，所以每一个线程都可以独立地改变自己的副本，而不会影响其它线程所对应的副本。

　　从线程的角度看，目标变量就象是线程的本地变量，这也是类名中“Local”所要表达的意思。

### 1 实例

​	线程对象封闭.

```java
  static class Bank{

      ThreadLocal<Integer>  t=new ThreadLocal<Integer>(){

          protected Integer initialValue(){
              return 100;
          }
      };

      public  int get(){
          return t.get();
      }

      public  void set(){
          t.set(t.get()+10);
      }

  }

  static class  Transfer implements Runnable{
      Bank bank;
      public Transfer(Bank bank){
          this.bank=bank;
      }

      @Override
      public void run() {
          for(int i=0;i<10;i++)
              bank.set();

          System.out.println(Thread.currentThread()+"   "+bank.get());
      }
  }

public static void main(String[] args){

      Bank bank=new Bank();
      Transfer  t= new Transfer(bank);
    //每个线程维持一个自己的threadlocalmap，不需要同步，使用弱引用实现，在对应的thread不调用之后，自动清除
      Thread t1=new Thread(t);
      Thread t2=new Thread(t);
      t1.start();
      t2.start();

      System.out.println(bank.get());
    /*
    *100
	Thread[Thread-0,5,main]   200
	Thread[Thread-1,5,main]   200
    /
}
```

```java
static class ThreadLocalMap {

    /**
     * The entries in this hash map extend WeakReference, using
     * its main ref field as the key (which is always a
     * ThreadLocal object).  Note that null keys (i.e. entry.get()
     * == null) mean that the key is no longer referenced, so the
     * entry can be expunged from table.  Such entries are referred to
     * as "stale entries" in the code that follows.
     */
    static class Entry extends WeakReference<ThreadLocal<?>> {
        /** The value associated with this ThreadLocal. */
        Object value;

        Entry(ThreadLocal<?> k, Object v) {
            super(k);
            value = v;
        }
    }
         /**
         * Construct a new map initially containing (firstKey, firstValue).
         * ThreadLocalMaps are constructed lazily, so we only create
         * one when we have at least one entry to put in it.
         */
        ThreadLocalMap(ThreadLocal<?> firstKey, Object firstValue) {
            table = new Entry[INITIAL_CAPACITY];
            int i = firstKey.threadLocalHashCode & (INITIAL_CAPACITY - 1);
            table[i] = new Entry(firstKey, firstValue);
            size = 1;
            setThreshold(INITIAL_CAPACITY);
        }

```

### 2  Weakreference

​	WeakReference如字面意思，弱引用， 当一个对象仅仅被weak reference（弱引用）指向, 而没有任何其他strong reference（强引用）指向的时候, 如果这时GC运行, 那么这个对象就会被回收，不论当前的内存空间是否足够，这个对象都会被回收。

​	WeakReference(T referent)：referent就是被弱引用的对象（注意区分弱引用对象和被弱引用的对应，弱引用对象是指WeakReference的实例或者其子类的实例），比如有一个Apple实例apple，可以如下使用，并且通过get()方法来获取apple引用。也可以再创建一个继承WeakReference的类来对Apple进行弱引用，下面就会使用这种方式。

​	**Entry继承WeakReference,使用弱引用，可以将ThreadLocal对象的生命周期和线程生命周期解绑，持有对ThreadLocal的弱引用，可以使得ThreadLocal在没有其他强引用的时候被回收掉，这样可以避免因为线程得不到销毁导致ThreadLocal对象无法被回收。**

```
WeakReference<Apple> appleWeakReference = new WeakReference<>(apple);
Apple apple2 = appleWeakReference.get();
```

###    3  ThreadLocal主要方法

​	threadloacl对Threadlocalmap进行封装，对外提供set(),get()方法。

```java
/**
 * Sets the current thread's copy of this thread-local variable
 * to the specified value.  Most subclasses will have no need to
 * override this method, relying solely on the {@link #initialValue}
 * method to set the values of thread-locals.
 *
 * @param value the value to be stored in the current thread's copy of
 *        this thread-local.
 */
public void set(T value) {
    Thread t = Thread.currentThread();
    ThreadLocalMap map = getMap(t);
    if (map != null)
        map.set(this, value);
    else
        createMap(t, value);
}

//ThreadLocal在1.6版本中不是用一个全局的Map来存各个线程的变量副本，而是在Thread类中有一个ThreadLocalMap的变量，然后用Thread.currentThread().threadLocals.get(this)来引用的各线程变量副本，这样避免了去同步全局的Map
ThreadLocal.ThreadLocalMap getMap(Thread t) {
        return t.threadLocals;
    }

void createMap(Thread t, T firstValue) {
        t.threadLocals = new ThreadLocalMap(this, firstValue);
    }

    /**
     * Returns the value in the current thread's copy of this
     * thread-local variable.  If the variable has no value for the
     * current thread, it is first initialized to the value returned
     * by an invocation of the {@link #initialValue} method.
     *
     * @return the current thread's value of this thread-local
     */
    public T get() {
        Thread t = Thread.currentThread();
        //每个线程中都存在一个threalocalmap的引用
        ThreadLocalMap map = getMap(t);
        if (map != null) {
            ThreadLocalMap.Entry e = map.getEntry(this);
            if (e != null) {
                @SuppressWarnings("unchecked")
                T result = (T)e.value;
                return result;
            }
        }
        return setInitialValue();
    }

```

### 4 ThreadLocalMap主要方法

 	本质是一个hashmap，维持着一个entry类型的数组。

```java
static class ThreadLocalMap {

    /**
     * The entries in this hash map extend WeakReference, using
     * its main ref field as the key (which is always a
     * ThreadLocal object).  Note that null keys (i.e. entry.get()
     * == null) mean that the key is no longer referenced, so the
     * entry can be expunged from table.  Such entries are referred to
     * as "stale entries" in the code that follows.
     */
    
    /*
    第二次垃圾回收时回收，可以通过如下代码实现
Object obj = new Object();
WeakReference<Object> wf = new WeakReference<Object>(obj);
obj = null;
wf.get();//有时候会返回null
wf.isEnQueued();//返回是否被垃圾回收器标记为即将回收的垃圾
弱引用是在第二次垃圾回收时回收，短时间内通过弱引用取对应的数据，可以取到，当执行过第二次垃圾回收时，将返回null。
弱引用主要用于监控对象是否已经被垃圾回收器标记为即将回收的垃圾，可以通过弱引用的isEnQueued方法返回对象是否被垃圾回收器标记。
    */
    static class Entry extends WeakReference<ThreadLocal<?>> {
        /** The value associated with this ThreadLocal. */
        Object value;
		//entry的构造方法  key:线程 ，value: v
        Entry(ThreadLocal<?> k, Object v) {
            super(k);
            value = v;
        }
    }

    /**
     * The initial capacity -- MUST be a power of two.
     */
    private static final int INITIAL_CAPACITY = 16;

    /**
     * The table, resized as necessary.
     * table.length MUST always be a power of two.
     */
    private Entry[] table;

      ThreadLocalMap(ThreadLocal<?> firstKey, Object firstValue) {
            table = new Entry[INITIAL_CAPACITY];
            //哈希探测   size=2^n  
            int i = firstKey.threadLocalHashCode & (INITIAL_CAPACITY - 1);
            table[i] = new Entry(firstKey, firstValue);
            size = 1;
            setThreshold(INITIAL_CAPACITY);
        }
    
    /* * Construct a new map including all Inheritable ThreadLocals
        * from given parent map. Called only by createInheritedMap.
    */   
          private ThreadLocalMap(ThreadLocalMap parentMap) {
            Entry[] parentTable = parentMap.table;
            int len = parentTable.length;
            setThreshold(len);
            table = new Entry[len];

            for (int j = 0; j < len; j++) {
                Entry e = parentTable[j];
                if (e != null) {
                    @SuppressWarnings("unchecked")
                    ThreadLocal<Object> key = (ThreadLocal<Object>) e.get();
                    if (key != null) {
                        Object value = key.childValue(e.value);
                        Entry c = new Entry(key, value);
                        
                       //线性探测法  环状探测  一直找到hash不冲突的位置
                        int h = key.threadLocalHashCode & (len - 1);
                        while (table[h] != null)
                            h = nextIndex(h, len);
                        table[h] = c;
                        size++;
                    }
                }
            }
        }
    
    //环形探测法 
    private static int nextIndex(int i, int len) {
        return ((i + 1 < len) ? i + 1 : 0);
    }
    
   		 //扩容    
        /**
         * Double the capacity of the table.
         */
        private void resize() {
            Entry[] oldTab = table;
            int oldLen = oldTab.length;
            int newLen = oldLen * 2;
            Entry[] newTab = new Entry[newLen];
            int count = 0;

            for (int j = 0; j < oldLen; ++j) {
                Entry e = oldTab[j];
                if (e != null) {
                    // 返回线程的引用，如果引用被GC回收
                    ThreadLocal<?> k = e.get();
                    if (k == null) {
                        e.value = null; // Help the GC
                    } else {
                        int h = k.threadLocalHashCode & (newLen - 1);
                        while (newTab[h] != null)
                            h = nextIndex(h, newLen);
                        newTab[h] = e;
                        count++;
                    }
                }
            }

            setThreshold(newLen);
            size = count;
            table = newTab;
        }

```