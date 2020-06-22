# 5 Java中的原子类
因为变量的类型有很多种，所以在Atomic包里一共提供了13个类，属于4种类型的原子更新方式，分别是**原子更新基本类型、原子更新数组、原子更新引用和原子更新属性（字段）**。Atomic包里的类基本都是使用**Unsafe实现的包装类**。
JUC 包提供 了一系列的原子性操作类，这些类都是使用**非阻塞算法 CAS 实现的**；

## 5.1原子更新基本类型类
getAndIncrement是如何实现原子操作的呢

```java
public final int getAndIncrement() {
    for (;;) {
        int current = get();
        int next = current + 1;
        if (compareAndSet(current, next))
            return current;
    }
}
public final boolean compareAndSet(int expect, int update) {
    return unsafe.compareAndSwapInt(this, valueOffset, expect, update);
}
```

源码中for循环体的第一步先取得AtomicInteger里存储的数值，第二步对AtomicInteger的当前数值进行加1操作，关键的第三步调用compareAndSet方法来进行原子更新操作，该方法**先检查当前数值是否等于current，等于意味着AtomicInteger的值没有被其他线程修改过，则将AtomicInteger的当前数值更新成next的值，如果不等compareAndSet方法会返回false，程序会进入for循环重新进行compareAndSet操作。**

Atomic包提供了3种基本类型的原子更新，但是Java的基本类型里还有char、float和double等。那么问题来了，如何原子的更新其他的基本类型呢？ Atomic包里的类基本都是使用Unsafe实现的；

> Unsafe只提供了3种CAS方法：`compareAndSwapObject`、`compareAndSwapInt`和`compareAndSwapLong`，再看AtomicBoolean源码，发现它是**先把Boolean转换成整型，再使用compareAndSwapInt进行CAS**，所以原子更新char、float和double变量也可以用类似的思路来实现。

与synchronized锁相比，这种原子更新方式代表一种不同的思维方式。**synchronized是悲观的**，它假定更新很可能冲突，所以先获取锁，得到锁后才更新。**原子变量的更新逻辑是乐观的**，它假定冲突比较少，但使用CAS更新，也就是进行冲突检测，如果确实冲突了，那也没关系，继续尝试就好了。**synchronized代表一种阻塞式算法，得不到锁的时候，进入锁等待队列，等待其他线程唤醒，有上下文切换开销。原子变量的更新逻辑是非阻塞式的，更新冲突的时候，它就重试，不会阻塞，不会有上下文切换开销。**





## 5.2原子更新数组

````java
public class AtomicIntegerArrayTest {
    static int[] value = new int[] { 1， 2 };
    static AtomicIntegerArray ai = new AtomicIntegerArray(value);
    public static void main(String[] args) {
        ai.getAndSet(0， 3);
        System.out.println(ai.get(0));//3
        System.out.println(value[0]);//1
    }
}
````

数组value通过构造方法传递进去，然后AtomicIntegerArray会将当前数组复制一份，所以当AtomicIntegerArray对内部的数组元素进行修改时，不会影响传入的数组。

