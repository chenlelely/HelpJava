# 8 Java中的并发工具类
## 8.1 等待多线程完成的CountDownLatch
https://juejin.im/post/5aeec3ebf265da0ba76fa327

CountDownLatch**允许一个或多个线程等待其他线程完成操作。**

````java
public class CountDownLatchTest {
    staticCountDownLatch c = new CountDownLatch(2);
    public static void main(String[] args) throws InterruptedException {
        new Thread(new Runnable() {
            @Override
            public void run() {
                System.out.println(1);
                c.countDown();
                System.out.println(2);
                c.countDown();
            }
        }).start();
        c.await();
        System.out.println("3");
    }
}
private static volatile CountDownLatch countDownLatch ＝new CountDownLatch(2) ;
````

````java
...
／／ 启动子线程
threadOne.start() ;
threadTwo.start() ;
System.out.println ("wait all child thread over !");
／／等待子线程执行完毕，返回
countDownLatch.await() ;
````

创建了一个 CountDownLatch 实例，因为有两个子线程所以构造函数的传参为 2。
主线程调用 countDownLatch.await()方法后会被阻塞。
子线程执行完毕后调用 countDownLatch.countDown()方法让 countDownLatch 内部的计数器减 l ，
所有子线程执行完毕并调用 countDown()方法后计数器会变为 0，这时候主线程的 await()方法才会返回 。
CountDownLatch的构造函数接收一个int类型的参数作为计数器，如果你想等待N个点完成，这里就传入N。
当我们**调用CountDownLatch的countDown()方法时，N就会减1，CountDownLatch的await()方法会阻塞当前线程，直到N变成零**。由于countDown方法可以用在任何地方，所以这里说的N个点，可以是N个线程，也可以是1个线程里的N个执行步骤。用在多个线程时，只需要把这个CountDownLatch的引用传递到线程里即可。
如果有某个解析sheet的线程处理得比较慢，我们不可能让主线程一直等待，所以可以使用另外一个带指定时间的await方法—`await（long time，TimeUnit unit）`，这个方法等待特定时间后，就会不再阻塞当前线程。join也有类似的方法。

## 8.2 同步屏障CyclicBarrier
CyclicBarrier的字面意思是可循环使用（Cyclic）的屏障（Barrier）。它要做的事情是，让一组线程到达一个屏障（也可以叫同步点）时被阻塞，直到最后一个线程到达屏障时，屏障才会开门，所有被屏障拦截的线程才会继续运行。
CyclicBarrier默认的构造方法是`CyclicBarrier（int parties）`，其参数表示屏障拦截的线程数量，每个线程调用await方法告诉CyclicBarrier我已经到达了屏障，然后当前线程被阻塞。

CyclicBarrier可以用于多线程计算数据，最后合并计算结果的场景。

### 8.2.3 CyclicBarrier和CountDownLatch的区别
CountDownLatch 的计数器是一次性的，也就是等到计数器值变为0 后，再调用 CountDownLatch 的 await 和 countdown 方法都会立刻返回，这就起不到线程同步的效果了，而CyclicBarrier的计数器可以使用reset()方法重置。所以CyclicBarrier能处理更为复杂的业务场景。
CyclicBarrier 是回环屏障的 意思 ，它可以让一组线程全部达到一个状态后再全部同 时执行 。这里之所以叫作回环是因为当所有等待线程执行完毕，并重置 CyclicBarrier 的状态后它可以被重用。之所以 叫作屏障是因为线程调用 await 方法后就会被阻塞，这个阻塞点就称为屏障点，等所有线程都调用了 await 方法后，线程们就会冲破屏障，继续 向下运行。

## 8.3 控制并发线程数的Semaphore
https://juejin.im/post/5aeec49b518825673614d183

Semaphore（信号量）是**用来控制同时访问特定资源的线程数量，它通过协调各个线程，以保证合理的使用公共资源。**

内部计数器是**递增的**，计数器到达同步数的时候才会放行

````java
//创建一个Semaphore 实例,有一个初始值
private static Semaphore semaphore = new Semaphore(O) ;
    //A线程执行任务并semaphore.release( );
    //B线程执行任务并semaphore.release( );
//等待子线程执行完毕，返回；
//传参为2说明调用 acquire 方法的线程会一直阻塞,直到信号量的计数变为 2 才会返回
semaphore.acquire (2) ;//线程同时启动
````


把它比作是控制流量的红绿灯。比如××马路要限制流量，只允许同时有一百辆车在这条路上行使，其他的都必须在路口等待，所以前一百辆车会看到绿灯，可以开进这条马路，后面的车会看到红灯，不能驶入××马路，但是如果前一百辆中有5辆车已经离开了××马路，那么后面就允许有5辆车驶入马路，这个例子里说的车就是线程，驶入马路就表示线程在执行，离开马路就表示线程执行完成，看见红灯就表示线程被阻塞，不能执行。

## 8.4 线程间交换数据的Exchanger
Exchanger（交换者）是一个用于线程间协作的工具类。Exchanger用于进行线程间的数据交换。**它提供一个同步点，在这个同步点，两个线程可以交换彼此的数据。**这两个线程通过exchange方法交换数据，**如果第一个线程先执行exchange()方法，它会一直等待第二个线程也执行exchange方法，当两个线程都到达同步点时，这两个线程就可以交换数据，将本线程生产出来的数据传递给对方**
