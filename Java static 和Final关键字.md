#### Java 关键字(static,final,volatile)

#### 一：static

1）修饰成员变量：

static关键字可以修饰成员变量，它所修饰的成员变量不属于对象的数据结构，而是属于类的变量，通常通过类名来引用static成员。

*当创建对象后，成员变量是存储在堆中的，而static成员变量和类的信息一起存储在方法区, 而不是在堆中，一个类的static成员变量只有“一份”（存储在方法区），无论该类创建了多少对象*。结果输结果输出123，因为num存在方法区只有一份！

```java
public class StaticDemo {

 public static void main(String[] args) {

  People p1 = new People(18);

  People p2 = new People(19);

  People p3 = new People(20);

 }

}

class People{

 private int age;

 private static int num;

 public People(int age) {

  this.age = age;

  System.out.print(++num);

 }
}
```

2）static修饰方法：

static修饰的方法*不需要针对某些对象进行操作，其运行结果仅仅与参数有关*，调用的话直接用类名就可以调用了。因此static方法中不能对非static成员进行访问，

static方法的作用在于提供一些工具方法（如Arrays.sort)和工厂方法（有兴趣可以了解静态工厂模式）等



3）static块：

属于类的代码块，在类加载期间执行并且只执行一次，可以用来加载静态资源。（如图片、音频和视频等文件）

#### 二：final

1）final修饰变量：

final意为不可改变，修饰成员变量时可以在生命时初始化或者在构造函数中初始化，修饰局部变量时在使用之前初始化。要是企图改变final修饰等变量值则会编译错误！

 

2）final修饰方法：

final修饰等方法不能被重写。

 

3）final修饰类：

final修饰等类不能被继承，jdk中的一些基本类库被定义成final，例如String，Math，Integer等，这样可以防止对系统造成危害。

 

#### 三：static final

​	static final修饰的成员变量被成为常量，必须声明时初始化并且不可被改变，static final常量在编译期间会被取代为其初始值。



#### 四：valatile

​      修饰变量，主要是用于处理并发过程中所有线程的可见性，常见的两类应用场景：

- 运算结果不 依赖变量的当前值，或者只有单一线程的修改能够改变变量的值（单个线程写）

- 变量不需要跟其他状态共同参与不变约束

  ```java
  volatiel boolean shudownrequested;
  public void shutdown(){
      shudownrequested=true;
  }
  public void dowork(){
      while(!shutdownrequested)
          //do something
  }
  ```

    特点是：对volatile变量修改后其他进程马上能看到新值，而普通变量必须要 先回写至内存中，然后才对其他进程可见。

java内存模型中规定的两类：

- 主内存：所有变量都存储在主内存中
- 工作内存：每个线程所私有，保存了所使用变量中的主内存副本拷贝

  线程对变量的所有操作必须是在工作内存中完成的，不能直接读写主内存的变量，线程之间的变量值传递均需要主内存来完成。



#### 五 synchronized

​	synchronized是Java中的关键字，是一种同步锁。它修饰的对象有以下几种：

1. 修饰一个代码块，被修饰的代码块称为同步语句块，其作用的范围是大括号{}括起来的代码，作用的对象是调用这个代码块的对象；
2. 修饰一个方法，被修饰的方法称为同步方法，其作用的范围是整个方法，作用的对象是调用这个方法的对象；
3. 修改一个静态的方法，其作用的范围是整个静态方法，作用的对象是这个类的所有对象；
4. 修改一个类，其作用的范围是synchronized后面括号括起来的部分，作用主的对象是这个类的所有对象。

```
public class SynchronizedDemo implements Runnable {
    private static int count = 0;

    public static void main(String[] args) {
        for (int i = 0; i < 10; i++) {
            Thread thread = new Thread(new SynchronizedDemo());
            thread.start();
        }
        try {
            Thread.sleep(500);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        System.out.println("result: " + count);
    }

    @Override
    public void run() {
   	 //对类加锁，相对于对所有实例加锁
        synchronized (SynchronizedDemo.class) {
            for (int i = 0; i < 1000000; i++)
                count++;
        }
    }
}
```

