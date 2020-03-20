# 7 Java中的线程池

引入原因

1. 任务处理过程从主线程中分离出来， 使得主循环能够更快地重新等待下一个到来的连接，使得任务在完成前面的请求之前可以接受新的请求， 从而提高响应性。
2. 任务可以并行处理， 从而能同时服务多个请求。 如果有多个处理器， 或者任务由于某种原因被阻塞， 程序的吞吐量将得到提高。
3. 任务处理代码必须是线程安全的， 因为当有多个任务时会并发地调用这段代码。

无限制创建线程的不足：(解决方式： 线程池 Executor 框架)

1. 线程生命周期的开销非常高
2. 资源消耗
3. 稳定性

使用线程池的好处：

1. 降低资源消耗
2. 提高响应速度
3. 提高线程的可管理性  

线程池提供了一个线程队列，队列中保存着所有等待状态的线程，避免了创建于销毁的额外开销，提高了响应速度。

## 7.1 Executor框架

Java的线程既是工作单元，也是执行机制。从JDK 5开始，把**工作单元与执行机制**分离开来。**工作单元包括Runnable和Callable，而执行机制由Executor框架提供。**

### Executor框架的两级调度模型

在HotSpot VM的线程模型中，**Java线程（java.lang.Thread）被一对一映射为本地操作系统线程**。Java线程启动时会创建一个本地操作系统线程；当该Java线程终止时，这个操作系统线程也会被回收。操作系统会调度所有线程并将它们分配给可用的CPU。

**在上层，Java多线程程序通常把应用分解为若干个任务，然后使用用户级的调度器（Executor框架）将这些任务映射为固定数量的线程；在底层，操作系统内核将这些线程映射到硬件处理器上**。这就是两级调度模型。从图中可以看出，**应用程序通过Executor框架控制上层的调度；而下层的调度由操作系统内核控制，下层的调度不受应用程序的控制。**
<img src="07Java线程池相关.assets/20190808092243572.png" alt="img" style="zoom:67%;" />

### 1.Executor框架的结构

Executors 是 一 个 工 厂 类 ， 可 以 创 建 3 种 类 型 的 ThreadPoolExecutor 和 2 种 类 型 的ScheduledThreadPool。  

- java.util.concurrent.Executor:负责线程的使用与调度的根接口
  - ExecutorService 子接口：线程池的主要接口
    - ThreadPoolExecutor 线程池的实现类
    - ScheduledExecutorService 子接口：负责线程的调度
      - ScheduledThreadPoolExecutor:继承ThreadPoolExecutor,实现ScheduledExecutorService

1. 任务;包括被执行任务需要实现的接口：**Runnable接口或Callable接口**。
2. 任务的执行;包括任务执行机制的核心接口Executor，以及继承自Executor的ExecutorService接口。Executor框架有两个关键类实现了ExecutorService接口（ThreadPoolExecutor和ScheduledThreadPoolExecutor）。
3. 异步计算的结果;包括接口Future和实现Future接口的FutureTask类。
   <img src="07Java线程池相关.assets/20190808092949956.png" alt="img" style="zoom:50%;" />

### 2.Executor框架的成员

Executor框架的主要成员：**ThreadPoolExecutor、ScheduledThreadPoolExecutor、Future接口、Runnable接口、Callable接口和Executors。**

- **ThreadPoolExecutor**
  - FixedThreadPool。创建使用固定线程数的线程池
  - SingleThreadExecutor。创建使用单个线程的线程池，多任务下任务排队
  - CachedThreadPool。创建一个大小无界的线程池，通常会创建与所需数量相同的线程，然后在他回收旧线程时候停止创建新线程

- **ScheduledThreadPoolExecutor**
  - ScheduledThreadPoolExecutor适用于需要多个后台线程执行周期任务，同时为了满足资源管理的需求而需要限制后台线程的数量的应用场景。
  - SingleThreadScheduledExecutor适用于需要单个后台线程执行周期任务，同时需要保证顺序地执行各个任务的应用场景

- **Future接口**
  - Future接口和实现Future接口的FutureTask类用来表示异步计算的结果，返回一个FutureTask对象

- **Runnable接口和Callable接口**
  - Runnable接口和Callable接口的实现类，都可以被ThreadPoolExecutor或ScheduledThreadPoolExecutor执行。它们之间的区别Runnable不会返回结果，而Callable可以返回结果Future<> res = executor.submin(Callable),使用res.get()方法获取结果。

## 7.2 ThreadPoolExecutor

<img src="07Java线程池相关.assets/20190808085857398.png" alt="img" style="zoom: 67%;" />

<img src="07Java线程池相关.assets/20190808090007202.png" alt="img" style="zoom: 50%;" />

### **创建线程池**

`new ThreadPoolExecutor(corePoolSize, maximumPoolSize, keepAliveTime,milliseconds,threadFactory,runnableTaskQueue, handler);`

1. corePoolSize（线程池的基本大小）
2. maximumPoolSize（线程池最大数量）
3. keepAliveTime（线程活动保持时间）
4. milliseconds（存活时间单位）
5. ThreadFactory：用于设置创建线程的工厂
6. runnableTaskQueue（任务队列）
   1. ArrayBlockingQueue： 基于数组的有界阻塞队列， FIFO
   2. LinkedBlockingQueue ： 基于链表的无界阻塞队列 ，FIFO ，吞吐量高于ArrayBlockingQueue， Executors.newFixedThreadPoll()使用了这个队列
   3. SynchronousQueue： 一个只存储一个元素的阻塞队列， 每个插入操作必须等到另一个线程调用移除操作， 否则插入一直处于阻塞状态， 吞吐量高于 LinkedBlockingQueue，Executors#newCachedThreadPoll()使用了这个队列
   4. PriorityBlockingQueue： 具有优先级的无界阻塞队列  
7. RejectedExecutionHandler（饱和策略）
   1. AbortPolicy：丢弃并抛出异常。
   2. CallerRunsPolicy：只用调用者所在线程来运行任务。
   3. DiscardOldestPolicy：丢弃队列里最近的一个任务，并执行当前任务。
   4. DiscardPolicy：丢弃不抛异常

注意：

1. 只有当任务都是同类型并且相互独立时， 线程池的性能才能达到最佳。 如果将运行时间较长的与运行时间较短的任务混合在一起， 那么除非线程池很大， 否则将可能造成拥塞。 如果提交的任务依赖于其他任务， 那么除非线程池无限大， 否则将可能造成死锁。 幸运的是，在基于网络的典型服务器应用程序中——web 服务器、 邮件服务器、 文件服务器等， 它们的请求通常都是同类型的并且相互独立的。
2. 设置线程池的大小：基于` Runtime.getRuntime().avialableprocessors() `进行动态计算
   对于**计算密集型的任务**， 在 N 个处理器的系统上， 当线程池为 N+1 时， 通过能实现最优的利用率（缺页故障等暂停时额外的线程也能确保 CPU 时钟周期不被浪费） 。
   对于**包含 IO 操作或者其他阻塞操作的任务**， 由于线程并不会一直执行， 因此线程池的规模应该更大， 比如 2*N。 
   要正确地设置线程池的大小， 你必须估算出任务的等待时间与计算时间的比值。 线程等待时间所占比例越高， 需要越多线程。 线程 CPU 时间所占比例越高， 需要越少线程。 这种估算不需要很精确， 而且可以通过一些分析或监控工具来获得。 你还可以通过另一种方法来调节线程池的大小： 在某个基准负载下， 分别设置不同大小的线程池来运行应用程序， 并观察 CPU 利用率。
   最佳线程数目 = （线程等待时间与线程计算时间之比 + 1） * CPU 数目  
3. 线程的创建与销毁
   基本大小也就是线程池的目标大小， 即在没有任务执行时线程池的大小， 并且只有在工作队列满了的情况下才会创建超出这个数量的线程。 线程池的最大大小表示可同时活动的线程数量的上限。 如果某个线程的空闲时间超过了存活时间， 那么将被标记为可回收的， 并且当线程池的当前大小超过了基本大小时， 这个线程将被终止。
4. 管理队列任务
   ThreadPoolExecutor 允许提供一个 BlockingQueue 来保存等待执行的任务。 基本的任务排队方法有 3 种： 无界队列、 有界队列和同步移交。
   一种稳妥的资源管理策略是使用有界队列， 有界队列有助于避免资源耗尽的情况发生， 但又带来了新的问题： 当队列填满后， 新的任务该怎么办？
5. 饱和策略
   当有界队列被填满后， 饱和策略开始发挥作用。 ThreadPoolExecutor 的饱和策略可以通过setRejectedExecutionHandler 来修改。 JDK 提供了几种不同的 RejectedExecutionHandler 的实现， 每种实现都包含有不同的饱和策略： AbortPolicy、 CallerRunsPolicy、 DiscardPolicy、DiscardOldestPolicy。
   1. 中止策略是默认的饱和策略， 该策略将抛出未检查的 RejectedExecutionException。 调用者可以捕获这个异常， 然后根据需求编写自己的处理代码。
   2. 当新提交的任务无法保存到队列中执行时， 抛弃策略会悄悄抛弃该任务。
   3. 抛弃最旧的策略则会抛弃下一个将被执行的任务， 然后尝试重新提交下一个将被执行的任务（如果工作队列是一个优先级队列， 那么抛弃最旧的将抛弃优先级最高的任务）
   4. 调用者运行策略实现了一种调节机制， 该策略既不会抛弃任务， 也不会抛出异常， 而是将某些任务回退给调用者， 从而降低新任务的流量。 它不会在线程池的某个线程中执行新提交的任务， 而是在一个调用了 execute 的线程中执行该任务。 为什么好？ 因为当服务器过载时， 这种过载情况会逐渐向外蔓延开来——从线程池到工作队列到应用程序再到 TCP 层，最终达到客户端， 导致服务器在高负载下实现一种平缓的性能降低。
   5. 线程工厂
      在许多情况下都需要使用定制的线程工厂方法。 例如， 你希望为线程池中的线程指定一个UncaughtExceptionHandler， 或者实例化一个定制的 Thread 类用于执行调试信息的记录，你还可能希望修改线程的优先级（虽然不提倡这样做） ， 或者只是给线程取一个更有意义的名字， 用来解释线程的转储信息和错误日志。
   6. 在调用构造函数后再定制 ThreadPoolExecutor  

### **向线程池提交任务**

- execute()方法用于提交**不需要返回值**的任务，所以无法判断任务是否被线程池执行成功。
- submit()方法用于提交**需要返回值**的任务。线程池会返回一个**future类型的对象**，通过这个future对象可以判断任务是否执行成功，并且可以通过**future的get()方法来获取返回值**，get()方法会阻塞当前线程直到任务完成，而使用`get（long timeout，TimeUnit unit）`方法则会阻塞当前线程一段时间后立即返回，这时候有可能任务没有执行完。

### **关闭线程池**

可以通过调用ExecutorService 的`shutdown`或`shutdownNow`方法来关闭线程池。它们的<u>原理是遍历线程池中的工作线程，然后逐个调用线程的interrupt方法来中断线程，所以无法响应中断的任务可能永远无法终止</u>。但是它们存在一定的区别

- shutdownNow首先将线程池的状态设置成STOP，然后**尝试停止所有的正在执行或暂停任务的线程，并返回等待执行任务的列表**(不管任务是否执行完)
- shutdown只是将线程池的状态设置成SHUTDOWN状态，然后**中断所有没有正在执行任务的线程**。

但是我们无法通过常规方法来找出哪些任务已经开始但尚未结束， 这意味着我们无法在关闭过程中知道正在执行的任务的状态， 除非任务本身会执行某种检查。 要知道哪些任务还没有完成， 你不仅需要知道哪些任务还没有开始， 而且还需要知道当 Executor 关闭时哪些任务正在执行。

处理非正常的线程终止（只对 execute 提交的任务有效， submit 提交的话会在 future.get 时将受检异常直接抛出）
要为线程池中的所有线程设置一个 UncaughtExceptionHandler， 需要为 ThreadPoolExecutor的构造函数提供一个 ThreadFactory。 标准线程池允许当发生未捕获异常时结束线程， 但由于使用了一个 try-finally 块来接收通知， 因此当线程结束时， 将有新的线程来代替它。 如果没有提供捕获异常处理器或者其他的故障通知机制， 那么任务会悄悄失败， 从而导致很大的混乱。 如果你希望在任务由于发生异常而失败时获得通知， 并且执行一些特定于任务的恢复操 作 ， 那 么 可 以 将 任 务 封 装 在 能 捕 获 异 常 的 Runnable 或 Callable 中 ， 或 者 改 写ThreadPoolExecutor 的 afterExecute 方法。

只有通过 execute 提交的任务， 才能将它抛出的异常交给未捕获异常处理器。 如果一个由submit 提 交的 任 务 由于 抛 出 了异 常 而 结束 ， 那 么这 个 异 常将 被 Future.get 封 装 在ExecutionException 中重新抛出  

