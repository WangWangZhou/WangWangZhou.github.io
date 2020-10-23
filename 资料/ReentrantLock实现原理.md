# AQS原理(AbstractQueuedSynchronizer)与ReentrantLock实现原理


# AQS原理

## 概述

AQS全称是 AbstractQueuedSynchronizer，是阻塞式锁和相关的同步器工具的框架。

![](https://xinqianpingtaib2btest.oss-cn-shenzhen.aliyuncs.com/xinqianpingtaib2btest/blogimg/2020/39cf0e93d453420a6fadf4b4a1ee4331_721070-20170504110246211-10684485.png)

**特点：**

- 用 state 属性来表示资源的状态（分独占模式和共享模式），子类需要定义如何维护这个状态，控制如何获取
  锁和释放锁
  - getState - 获取 state 状态
  - setState - 设置 state 状态
  - compareAndSetState - cas 机制设置 state 状态
  - 独占模式是只有一个线程能够访问资源，而共享模式可以允许多个线程访问资源
- 提供了基于 FIFO 的等待队列，类似于 Monitor 的 EntryList
- 条件变量来实现等待、唤醒机制，支持多个条件变量，类似于 Monitor 的 WaitSet



子类主要实现这样一些方法（默认抛出 UnsupportedOperationException）


- <font color=crimson >tryAcquire(int arg)：独占式获取同步状态，获取同步状态成功后，其他线程需要等待该线程释放同步状态才能获取同步状态</font>
- <font color=crimson >tryRelease(int arg)：独占式释放同步状态</font>
- <font color=crimson >tryAcquireShared(int arg)：共享式获取同步状态，返回值大于等于0则表示获取成功，否则获取失败</font>
- <font color=crimson >tryReleaseShared(int arg)：共享式释放同步状态</font>
- <font color=crimson >isHeldExclusively()：当前同步器是否在独占式模式下被线程占用，一般该方法表示是否被当前线程所独占</font>



获取锁的姿势 

```
// 如果获取锁失败
if (!tryAcquire(arg)) {
	// 入队, 可以选择阻塞当前线程 park unpark
}
```

释放锁的姿势

```
// 如果释放锁成功
if (tryRelease(arg)) {
	// 让阻塞线程恢复运行
}
```



## 利用AQS手写(实现自定义)不可重入锁

写一个MyLock类，实现一个Lock接口，实现里面的抽象方法（包括lock加锁，lockInterruptibly加锁,可打断，tryLock尝试加锁，tryLock尝试加锁,带超时，unlock解锁，newCondition创建条件变量）。在MyLock类里面写一个内部类MySync，继承AQS队列同步器。

![](https://xinqianpingtaib2btest.oss-cn-shenzhen.aliyuncs.com/xinqianpingtaib2btest/blogimg/2020/20201021190437.png)



自定义锁，代码

```java
// 自定义锁（不可重入锁）
class MyLock implements Lock {

    // 独占锁  同步器类
    class MySync extends AbstractQueuedSynchronizer {
        @Override
        protected boolean tryAcquire(int arg) {
            if(compareAndSetState(0, 1)) {
                // 加上了锁，并设置 owner 为当前线程
                setExclusiveOwnerThread(Thread.currentThread());
                return true;
            }
            return false;
        }

        @Override
        protected boolean tryRelease(int arg) {
            setExclusiveOwnerThread(null);
            setState(0);
            return true;
        }

        @Override // 是否持有独占锁
        protected boolean isHeldExclusively() {
            return getState() == 1;
        }

        public Condition newCondition() {
            return new ConditionObject();
        }
    }

    private MySync sync = new MySync();

    @Override // 加锁（不成功会进入等待队列）
    public void lock() {
        sync.acquire(1);
    }

    @Override // 加锁，可打断
    public void lockInterruptibly() throws InterruptedException {
        sync.acquireInterruptibly(1);
    }

    @Override // 尝试加锁（一次）
    public boolean tryLock() {
        return sync.tryAcquire(1);
    }

    @Override // 尝试加锁，带超时
    public boolean tryLock(long time, TimeUnit unit) throws InterruptedException {
        return sync.tryAcquireNanos(1, unit.toNanos(time));
    }

    @Override // 解锁
    public void unlock() {
        sync.release(1);
    }

    @Override // 创建条件变量
    public Condition newCondition() {
        return sync.newCondition();
    }
}
```

思考，compareAndSetState为什么要把状态`state`设置为`1 ` ？

测试

```java
import java.util.concurrent.TimeUnit;
import java.util.concurrent.locks.AbstractQueuedSynchronizer;
import java.util.concurrent.locks.Condition;
import java.util.concurrent.locks.Lock;

@Slf4j(topic = "c.TestAqs")
public class TestAqs {
    public static void main(String[] args) {
        MyLock lock = new MyLock();
        new Thread(() -> {
            lock.lock();
            try {
                log.debug("locking...");
                sleep(1);
            } finally {
                log.debug("unlocking...");
                lock.unlock();
            }
        },"t1").start();

        new Thread(() -> {
            lock.lock();
            try {
                log.debug("locking...");
            } finally {
                log.debug("unlocking...");
                lock.unlock();
            }
        },"t2").start();
    }
}
```



# ReentrantLock原理

ReentrantLock主要利用CAS+AQS队列同步器来实现。它支持公平锁和非公平锁,两者的实现类似。

![](https://xinqianpingtaib2btest.oss-cn-shenzhen.aliyuncs.com/xinqianpingtaib2btest/blogimg/2020/20201021195957.png)

## 1.非公平锁实现原理

从空参构造器开始看，默认为非公平锁实现

```java
    public ReentrantLock() {
        sync = new NonfairSync();
    }
```

接下来，看加锁流程。找到`lock`方法

```java
    public void lock() {
        sync.lock();
    }
```

进去

```java
abstract void lock();
```

发现是一个抽象方法，这时，查看它的实现（如果是用`idea`，点左边向下的小箭头，或者双击选中`lock`方法，按下快捷键`Ctrl+Alt+B`，会弹出提示）这里我们要选择非公平锁的实现，也就是`NonfairSync`。

![image-20201021202051297](C:\Users\MagicBook\AppData\Roaming\Typora\typora-user-images\image-20201021202051297.png)



```java
    static final class NonfairSync extends Sync {

        final void lock() {
            //尝试加锁，把状态从0改成1
            if (compareAndSetState(0, 1))
                //加锁成功，把线程设置为当前线程
                setExclusiveOwnerThread(Thread.currentThread());
            else
                acquire(1);
        }
```

### 加锁流程



没有竞争时，

![](https://xinqianpingtaib2btest.oss-cn-shenzhen.aliyuncs.com/xinqianpingtaib2btest/blogimg/2020/20201021202808.png)

第一个竞争出现时

![](https://xinqianpingtaib2btest.oss-cn-shenzhen.aliyuncs.com/xinqianpingtaib2btest/blogimg/2020/20201021202912.png)

**线程Thread-0**已经改了状态，**线程Thread-1**用cas修改失败，执行`acquire`方法

```java
    public final void acquire(int arg) {
        if (!tryAcquire(arg) &&
            acquireQueued(addWaiter(Node.EXCLUSIVE), arg))
            selfInterrupt();
    }
```

tryAcquire方法，会尝试获取锁，

- 成功返回true，取反，得false，不会进下面的代码
- 失败返回false，取反，得true，进入下面的代码

因为这个时候，`state`是`1`，结果仍然失败。tryAcquire进入代码块。

> 接下来，大概步骤就是尝试创建节点对象，加入等待队列。

接下来进入 addWaiter 逻辑，构造 Node 队列
- 图中黄色三角表示该 Node 的 waitStatus 状态，其中 0 为默认正常状态
- Node 的创建是懒惰的
- 其中第一个 Node 称为 Dummy（哑元）或哨兵，用来占位，并不关联线程

![](https://xinqianpingtaib2btest.oss-cn-shenzhen.aliyuncs.com/xinqianpingtaib2btest/blogimg/2020/20201021204514.png)



```java
    private Node addWaiter(Node mode) {
        //根据给定的模式（独占或者共享）新建Node
        Node node = new Node(Thread.currentThread(), mode);
        //快速尝试添加尾节点
        Node pred = tail;
        if (pred != null) {
            node.prev = pred;
            //CAS设置尾节点
            if (compareAndSetTail(pred, node)) {
                pred.next = node;
                return node;
            }
        }
        //多次尝试
        enq(node);
        return node;
    }
```

创建好节点，进入`acquireQueued`方法

​```java
    final boolean acquireQueued(final Node node, int arg) {
        boolean failed = true;
        try {
            boolean interrupted = false;
            for (;;) {
                //获取当前节点的前驱节点
                final Node p = node.predecessor();
                //看一下前驱节点是不是头节点，如果是的话，继续tryAcquire
                if (p == head && tryAcquire(arg)) {
                    setHead(node);
                    p.next = null; // help GC
                    failed = false;
                    return interrupted;
                }
                if (shouldParkAfterFailedAcquire(p, node) &&
                    parkAndCheckInterrupt())
                    interrupted = true;
            }
        } finally {
            if (failed)
                cancelAcquire(node);
        }
    }
```



当前线程进入 acquireQueued 逻辑
1. acquireQueued 会在一个死循环中不断尝试获得锁，失败后进入 park 阻塞
2. 如果自己是紧邻着 head（排第二位），那么再次 tryAcquire 尝试获取锁，当然这时 state 仍为 1，失败
3. 进入 shouldParkAfterFailedAcquire 逻辑，将前驱 node，即 head 的 waitStatus 改为 -1，这次返回 false

![](https://xinqianpingtaib2btest.oss-cn-shenzhen.aliyuncs.com/xinqianpingtaib2btest/blogimg/2020/20201021210405.png)

4. shouldParkAfterFailedAcquire 执行完毕回到 acquireQueued ，再次 tryAcquire 尝试获取锁，当然这时
state 仍为 1，失败
5. 当再次进入 shouldParkAfterFailedAcquire 时，这时因为其前驱 node 的 waitStatus 已经是 -1，这次返回
true
6. 进入 parkAndCheckInterrupt， Thread-1 park（灰色表示）

![](https://xinqianpingtaib2btest.oss-cn-shenzhen.aliyuncs.com/xinqianpingtaib2btest/blogimg/2020/20201021211555.png)



再次有多个线程经历上述过程竞争失败，变成这个样子

![](https://xinqianpingtaib2btest.oss-cn-shenzhen.aliyuncs.com/xinqianpingtaib2btest/blogimg/2020/20201021211911.png)

### 解锁流程

**Thread-0** 释放锁，进入 tryRelease 流程，如果成功

- 设置 exclusiveOwnerThread 为 null

- state = 0



![](https://xinqianpingtaib2btest.oss-cn-shenzhen.aliyuncs.com/xinqianpingtaib2btest/blogimg/2020/20201021211955.png)

当前队列不为 null，并且 head 的 waitStatus = -1，进入 unparkSuccessor 流程

找到队列中离 head 最近的一个 Node（没取消的），unpark 恢复其运行，本例中即为 `Thread-1`

回到 `Thread-1` 的 acquireQueued 流程



![](https://xinqianpingtaib2btest.oss-cn-shenzhen.aliyuncs.com/xinqianpingtaib2btest/blogimg/2020/20201021213510.png)

如果加锁成功（没有竞争），会设置

- exclusiveOwnerThread 为 `Thread-1`，state = 1

- head 指向刚刚 `Thread-1` 所在的 Node，该 Node 清空 Thread

- 原本的 head 因为从链表断开，而可被垃圾回收

如果这时候有其它线程来竞争（非公平的体现），例如这时有 `Thread-4` 来了

![](https://xinqianpingtaib2btest.oss-cn-shenzhen.aliyuncs.com/xinqianpingtaib2btest/blogimg/2020/20201021214829.png)



如果不巧又被 Thread-4 占了先

- Thread-4 被设置为 exclusiveOwnerThread，state = 1

- Thread-1 再次进入 acquireQueued 流程，获取锁失败，重新进入 park 阻塞



ReentrantLock释放锁流程代码

```java
    public void unlock() {
        sync.release(1);
    }
```

然后

```java
    public final boolean release(int arg) {
        if (tryRelease(arg)) {
            Node h = head;
            if (h != null && h.waitStatus != 0)
                unparkSuccessor(h);
            return true;
        }
        return false;
    }
```

然后

```java
    protected boolean tryRelease(int arg) {
        throw new UnsupportedOperationException();
    }
```

然后

![](https://xinqianpingtaib2btest.oss-cn-shenzhen.aliyuncs.com/xinqianpingtaib2btest/blogimg/2020/20201021212706.png)

```java
        protected final boolean tryRelease(int releases) {
            int c = getState() - releases;
            if (Thread.currentThread() != getExclusiveOwnerThread())
                throw new IllegalMonitorStateException();
            boolean free = false;
            if (c == 0) {
                free = true;
                setExclusiveOwnerThread(null);
            }
            setState(c);
            return free;
        }
```



## 2.可重入锁原理

```java
    static final class NonfairSync extends Sync {
		//...

		// Sync 继承过来的方法，方便阅读，放在此处
        final boolean nonfairTryAcquire(int acquires) {
            final Thread current = Thread.currentThread();
            int c = getState();
            if (c == 0) {
                if (compareAndSetState(0, acquires)) {
                    setExclusiveOwnerThread(current);
                    return true;
                }
            }
            //如果已经获得了锁，线程还是当前线程，表示发生了锁重入
            else if (current == getExclusiveOwnerThread()) {
                int nextc = c + acquires;
                if (nextc < 0) // overflow
                    throw new Error("Maximum lock count exceeded");
                setState(nextc);
                return true;
            }
            return false;
        }
        
        // Sync 继承过来的方法，方便阅读，放在此处
        protected final boolean tryRelease(int releases) {
            // state减减
            int c = getState() - releases;
            if (Thread.currentThread() != getExclusiveOwnerThread())
                throw new IllegalMonitorStateException();
            boolean free = false;
            //支持锁重入，只有state减为0，才释放成功
            if (c == 0) {
                free = true;
                setExclusiveOwnerThread(null);
            }
            setState(c);
            return free;
        }        
        
    }
```



## 3.可打断原理

### 不可打断模式

在此模式下，即使它被打断，仍会驻留在 AQS 队列中，一直要等到获得锁后方能得知自己被打断了

```java
  //Sync 继承自AQS
  static final class NonfairSync extends Sync {
        //...
      
    	
        private final boolean parkAndCheckInterrupt() {
            //如果打断标记已经是true,则park会失效
            LockSupport.park(this);
            // interrupted会清除打断标记
            return Thread.interrupted();
        }
      
      
        final boolean acquireQueued(final Node node, int arg) {
            boolean failed = true;
            try {
                boolean interrupted = false;
                for (;;) {
                    final Node p = node.predecessor();
                    if (p == head && tryAcquire(arg)) {
                        setHead(node);
                        p.next = null; // help GC
                        failed = false;
                        //还是需要获得锁后，才能返回打断状态
                        return interrupted;
                    }
                    if (shouldParkAfterFailedAcquire(p, node) &&
                        parkAndCheckInterrupt())
                        //如果是因为 interruppt 被唤醒，返回打断状态为true
                        interrupted = true;
                }
            } finally {
                if (failed)
                    cancelAcquire(node);
            }
        } 

      
        public final void acquire(int arg) {
            if (!tryAcquire(arg) &&
                acquireQueued(addWaiter(Node.EXCLUSIVE), arg))
                //如果打断状态为 true
                selfInterrupt();
        }     

        static void selfInterrupt() {
            //重新产生一次中断
            Thread.currentThread().interrupt();
        }      
    }
```



### 可打断模式

``` java
    static final class NonfairSync extends Sync {
		//...
        
        
        public final void acquireInterruptibly(int arg)
                throws InterruptedException {
            if (Thread.interrupted())
                throw new InterruptedException();
            //如果没有获得到锁，进入(...)
            if (!tryAcquire(arg))
                doAcquireInterruptibly(arg);
        }

        // (...)可打断的获取锁流程
        private void doAcquireInterruptibly(int arg)
            throws InterruptedException {
            final Node node = addWaiter(Node.EXCLUSIVE);
            boolean failed = true;
            try {
                for (;;) {
                    final Node p = node.predecessor();
                    if (p == head && tryAcquire(arg)) {
                        setHead(node);
                        p.next = null; // help GC
                        failed = false;
                        return;
                    }
                    if (shouldParkAfterFailedAcquire(p, node) &&
                        parkAndCheckInterrupt())
                        // 在 park 过程中如果被 interrupt 会进入此
                        // 这时候抛出异常，而不会再次进入 for(;;)
                        throw new InterruptedException();
                }
            } finally {
                if (failed)
                    cancelAcquire(node);
            }
        }        
        
    }
```



## 4.公平锁实现原理





# 参考资料

[java中的锁原理、 锁优化、CAS、AQS详解](http://www.360doc.com/content/19/0410/10/13328254_827651184.shtml)
[Java并发之AQS原理浅析上](https://www.cnblogs.com/NathanYang/p/9944632.html)
[Thread中interrupted()方法和isInterrupted()方法区别总结](https://blog.csdn.net/zhuyong7/article/details/80852884)