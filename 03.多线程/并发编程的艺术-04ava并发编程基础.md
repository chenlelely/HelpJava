# 4 Java并发编程基础
## 4.1 线程基础简介
进程是代码在数据集合上的一次运行活动 ， 是系统进行**资源分配和调度的基本单位** ， 线程则是进程的一个执行路径；操作系统在分配资源时是把资源分配给进程 的， 但是 **CPU 资源 比较特殊 ，它是被分配到线程的** ， 因为真正要占用 CPU 运行的是线程 ， 所以也说线程是 **CPU 分配的基本单位**。

**线程优先级**的范围从1~10，在线程构建的时候可以通过setPriority(int)方法来修改优先级，默认优先级是5；（线程优先级不能作为程序正确性的依赖，因为操作系统可以完全不用理会Java线程对于优先级的设定）

Java线程在运行的生命周期中可能处于6种不同的状态：
![img](Untitled.assets/04sgdfdsgadgype_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzMwMjgxNTU5,size_16,color_FFFFFF,t_70)

![img](Untitled.assets/04M6Ly9ibG9nLmNzZG4ubmV0L3FxXzMwMjgxNTU5,size_16,color_FFFFFF,t_70)

>注意：当线程进入到synchronized方法或者synchronized代码块时，线程切换到的是BLOCKED状态，而使用java.util.concurrent.locks下lock进行加锁的时候线程切换的是WAITING或者TIMED_WAITING状态，因为lock会调用LockSupport的方法。

1. 初始态：NEW
   - 创建一个Thread对象，但还未调用start()启动线程时，线程处于初始态。

2. 运行态：RUNNABLE
   在Java中，运行态包括**就绪态** 和 **运行态**。
   - 就绪态：`start()/yield()`
     - 该状态下的线程**已经获得执行所需的所有资源**，只要CPU分配执行权就能运行。
       所有就绪态的线程存放在**就绪队列**中。
   - 运行态
     - **获得CPU执行权，正在执行的线程**。
     - 由于一个CPU同一时刻只能执行一条线程，因此每个CPU每个时刻只有一条运行态的线程。
3. 阻塞态
   - 一个线程因为**等待临界区的锁被阻塞**产生的状态，Lock 或者synchronize 关键字产生的状态
   - 当一条正在执行的线程**请求某一资源（锁）失败**时，就会进入阻塞态。
   - 而在Java中，阻塞态**专指请求锁失败时进入的状态**。
   - 由一个**阻塞队列**存放所有阻塞态的线程。
   - 处于阻塞态的线程会**不断请求资源**，一旦请求成功，就会进入**就绪队列**,进入RUNNABLE 状态，等待执行。
   - PS：锁、IO、Socket等都资源。

4. 等待态
   - 一个线程**进入了锁**，但是需要等待其他线程执行某些操作。时间不确定
   - 当前线程中调用wait、join、park函数时，当前线程就会进入等待态（前提是获得了锁）。
   - 也有一个**等待队列**存放所有等待态的线程。
   - 线程处于等待态表示它需要等待其他线程的指示才能继续运行。
   - 进入等待态的线程会**释放CPU执行权，并释放资源**（如：锁）
5. 超时等待态
   - 当运行中的线程调用sleep(time)、wait、join、parkNanos、parkUntil时，就会进入该状态；
   - 它和等待态一样，**并不是因为请求不到资源，而是主动进入，并且进入后需要其他线程唤醒；**
   - 进入该状态后释放CPU执行权 和 占有的资源。
   - 与等待态的区别：**到了超时时间后自动进入阻塞队列，开始竞争锁**。
6. 终止态
   - 线程执行结束后的状态。

**注意：**

- 只有runnable 状态的线程才能获得CPU时间片，并被选中执行。
- wait()方法会**释放CPU执行权 和 占有的锁**。
- sleep(long)方法**仅释放CPU使用权，锁仍然占用**；线程被放入**超时等待队列**，与yield相比，它会使线程较长时间得不到运行。
- yield()方法**仅释放CPU执行权，锁仍然占用**，线程会被放入**就绪队列**，会在短时间内再次执行。
- wait和notify必须配套使用，即必须使用同一把锁调用；
- wait和notify必须放在一个同步块中
- 调用wait和notify的对象必须是他们所处同步块的锁对象。

## 4.2 启动和终止线程
**线程start()**方法的含义是：当前线程（即parent线程）同步告知Java虚拟机，只要线程规划器空闲，应立即启动调用start()方法的线程。

### 4.2.1创建线程

1. 通过继承Thread类，重写run()方法，没有返回值void；

2. 通过实现runable接口重写run()方法，没有返回值void；

3. 通过实现callable接口,重写call()方法，call()方法有返回值类型。

````java
 public class CreateThreadDemo {
 
     public static void main(String[] args) {
         //1.继承Thread
         Thread thread = new Thread() {
             @Override
             public void run() {
                 System.out.println("继承Thread");
                 super.run();
             }
         };
         thread.start();
         //2.实现runable接口
         Thread thread1 = new Thread(new Runnable() {
             @Override
             public void run() {
                 System.out.println("实现runable接口");
             }
         });
         thread1.start();
         //3.实现callable接口
         ExecutorService service = Executors.newSingleThreadExecutor();
         Future<String> future = service.submit(new Callable() {
             @Override
             public String call() throws Exception {//有返回值类型
                 return "通过实现Callable接口";
             }
         });
         try {
             String result = future.get();
             System.out.println(result);
         } catch (InterruptedException e) {
             e.printStackTrace();
         } catch (ExecutionException e) {
             e.printStackTrace();
         }
     }
 
 }
````

由于java不能多继承可以实现多个接口，因此，在创建线程的时候尽量多考虑采用实现接口的形式；

实现callable接口，提交给ExecutorService返回的是异步执行的结果，另外，通常也可以利用FutureTask(Callable callable)将callable进行包装然后FeatureTask提交给ExecutorsService。如图：
![FutureTaskæ¥å£å®ç°å³ç³»](Untitled.assets/04fasdfasdfasdfasformat,png)

另外由于FeatureTask也实现了Runable接口也可以利用上面第二种方式（实现Runable接口）来新建线程；

可以通过Executors将Runable转换成Callable，具体方法是：`Callable callable(Runnable task, T result)， Callable callable(Runnable task)`。
### 4.2.2线程中断Interrupt()
中断可以理解为线程的一个**标识位属性**，它**表示一个运行中的线程是否被其他线程进行了中断操作**。
中断好比其他线程对该线程打了个招呼，其他线程通过**调用该线程的interrupt()方法对其进行中断操作**。
**线程通过检查自身是否被中断来进行响应**，线程通过方法`isInterrupted()`来进行判断是否被中断，也可以调用静态方法`Thread.interrupted()`对当前线程的中断标识位进行复位。如果该线程已经处于终结状态，即使该线程被中断过，在调用该线程对象的`isInterrupted()`时依旧会返回false。需要注意的是，当抛出`InterruptedException`时候，会**清除中断标志位**，也就是说在调用isInterrupted会返回false。

> wait()、sleep()、join()方法中调用Interrupt()会抛出IntrrruptedException

**安全地终止线程**

- 中断状态是线程的一个标识位，而中断操作是一种简便的线程间交互方式，而这种交互方式最适合用来取消或停止任务。
- 除了中断以外，还可以利用一个boolean变量来控制是否需要停止任务并终止该线程

````java
public class Shutdown {
    public static void main(String[] args) throws Exception {
        Runner one = new Runner();
        Thread countThread = new Thread(one, "CountThread");
        countThread.start();
        // 睡眠1秒，main线程对CountThread进行中断，使CountThread能够感知中断而结束
        TimeUnit.SECONDS.sleep(1);
        countThread.interrupt();
        
        Runner two = new Runner();
        countThread = new Thread(two, "CountThread");
        countThread.start();
        // 睡眠1秒，main线程对Runner two进行取消，使CountThread能够感知on为false而结束
        TimeUnit.SECONDS.sleep(1);
        two.cancel();
    }
    private static class Runner implements Runnable {
        private long i;
        private volatile boolean on = true;
        @Override
        public void run() {
            while (on && !Thread.currentThread().isInterrupted()){
                i++;
            }
            System.out.println("Count i = " + i);
        }
        public void cancel() {
            on = false;
        }
    }
}
//输出
Count i = 543487324
Count i = 540898082
````

>main线程通过**中断操作**和**cancel()方法**均可使CountThread得以终止。这种通过标识位或者中断操作的方式能够使线程在终止时有机会去清理资源，而不是武断地将线程停止，因此这种终止线程的做法显得更加安全和优雅。

## 4.3 线程间通信
线程开始运行，拥有自己的栈空间，就如同一个脚本一样，按照既定的代码一步一步地执行，直到终止。

> Java支持多个线程同时访问一个对象或者对象的成员变量，由于每个线程可以拥有这个变量的拷贝（虽然对象以及成员变量分配的内存是在共享内存中的，但是每个执行的线程还是可以拥有一份拷贝，这样做的目的是加速程序的执行，这是现代多核处理器的一个显著特性），所以程序在执行过程中，一个线程看到的变量并不一定是最新的。

### 4.3.1. volatile和synchronized关键字
**关键字volatile**可以用来修饰字段（成员变量），就是告知程序任何对该变量的访问**均需要从共享内存中获取**，而对它的改变必须同步刷新回共享内存，它能保证所有线程对变量访问的**可见性**

**关键字synchronized**可以修饰方法或者以同步块的形式来进行使用，它主要确保多个线程在同一个时刻，只能有一个线程处于方法或者同步块中，它保证了线程对变量访问的**可见性和排他性**。

### 4.3.2. 等待/通知机制
**等待/通知**的相关方法是任意Java对象都具备的，因为这些方法被定义在所有对象的超类`java.lang.Object`上

> 等待/通知机制，是指一个线程A调用了对象O的wait()方法进入等待状态，而另一个线程B调用了对象O的notify()或者notifyAll()方法，线程A收到通知后从对象O的wait()方法返回，进而执行后续操作。上述**两个线程通过对象O来完成交互，而对象上的wait()和notify/notifyAll()的关系就如同开关信号一样，用来完成等待方和通知方之间的交互工作**。

等待方遵循如下原则。
1）获取对象的锁。
2）如果条件不满足，那么调用对象的wait()方法，被通知后仍要检查条件。
3）条件满足则执行对应的逻辑。

````java
synchronized(对象) {
    while(条件不满足) {
        对象.wait();
    }
    对应的处理逻辑
}
````

通知方遵循如下原则。
1）获得对象的锁。
2）改变条件。
3）通知所有等待在对象上的线程。

````java
synchronized(对象) {
    改变条件
    对象.notifyAll();
}
````

**注意：**

1）使用wait()、notify()和notifyAll()时需要**先对调用对象加锁**。
2）调用wait()方法后，线程状态由RUNNING变为WAITING，并将当前线程放置到对象的**等待队列**。
3）notify()或notifyAll()方法调用后，等待线程依旧不会从wait()返回，需要调用notify()或notifAll()的线程**释放锁之后，等待线程才有机会从wait()返回**。
4）notify()方法将等待队列中的一个等待线程**从等待队列中移到同步队列中**，而notifyAll()方法则是将等待队列中所有的线程全部移到同步队列，被移动的线程状态**由WAITING变为BLOCKED**。
5）**从wait()方法返回的前提是获得了调用对象的锁**。
![img](Untitled.assets/04,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4U5,size_16,color_FFFFFF,t_70)

WaitThread首先获取了对象的锁，然后调用对象的wait()方法，从而放弃了锁并进入了对象的等待队列WaitQueue中，进入等待状态。由于WaitThread释放了对象的锁，NotifyThread随后获取了对象的锁，并调用对象的notify()方法，将WaitThread从WaitQueue移到SynchronizedQueue中，此时WaitThread的状态变为阻塞状态。NotifyThread释放了锁之后，WaitThread再次获取到锁并从wait()方法返回继续执行。

### 4.3.3 Thread.join()的使用
如果一个线程A执行了threadB.join()语句，其含义是：**当前线程A等待threadB线程终止之后才从threadB.join()返回**。当线程终止时，会调用线程自身的notifyAll()方法，会通知所有等待在该线程对象上的线程。可以看到join()方法的逻辑结构与等待/通知经典范式一致，即加锁、循环和处理逻辑3个步骤。

````java
//thread.join()源码
//加锁当前线程对象
public final synchronized void join() throws InterruptedException {
    // 条件不满足，继续等待
    while (isAlive()) {
        wait(0);
    }
    // 条件符合，方法返回
}
````

### 4.3.4 ThreadLocal的使用
ThreadLocal，即线程本地变量，是一个**以ThreadLocal对象为键、任意对象为值**的存储结构。这个结构被附带在线程上，也就是说**一个线程可以根据一个ThreadLocal对象查询到绑定在这个线程上的一个值**。可以通过`set(T)`方法来设置一个值，在当前线程下再通过`get()`方法获取到原先设置的值。如果你**创 建了 一 个ThreadLocal 变量 ，那么访问这个变量 的每个线程都会有这个变量的一个本地副本** 。
![img](Untitled.assets/040cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzMwMjgxNTU5,size_16,color_FFFFFF,t_70)

Thread 类中有一个` threadLocals `和一个` inheritableThreadLocals `， 它们 都是` ThreadLocalMap `类型 的变量 ， 而 `ThreadLocalMap` 是一个定制化的 `Hashmap`。**每个线程可以关联多个** `ThreadLocal `变量。
每个线程的本地变量不是存放在 ThreadLocal 实例里面，而是**存放在调用线程的 threadLocals 变量里面** 。 也就是说 ThreadLocal 类型的本地变量**存放在具体的线程内存空间中** 。threadLocals 是一个 HashMap 结构 ， 其中 **key 为我们定义的 ThreadLocal 变量的 this 引用（当前线程）， value 是通过 set 方法传递的值** 。
ThreadLocal 就是一个工具壳，它通过 set 方法把 value 值放入调用线程的 threadLocals 里面并存放起来 ， 当调用 线程调用它的 get 方法时，再从当 前线程的 threadLocals 变量里面将其拿出来使用 。

Threadlocal **不支持继承性**，同一个 ThreadLocal 变量在父线程中被设置值后 ， 在子线程中 是获取不到的。
InheritableThreadLocal继承自 ThreadLocal ， 其提供了一个特性，就是**让子线程可 以访问在父线程中设置的本地变量** 。

