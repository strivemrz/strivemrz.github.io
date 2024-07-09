---
title: java锁机制
categories: Java
tags:
  - Java
excerpt: ''
cover: http://image.qianye88.com/pic/d0888999ad7e0460ac1ab07384202225
---

#### 公平锁/非公平锁

公平锁是指多个线程按照申请锁的顺序来获取锁

非公平锁是指多个线程获取锁的顺序不是按照申请锁的顺序，有可能是后申请的线程比先申请的线程优先获取锁。又可能会造成优先级反转和饥饿现象。

#### 可重入锁

可重入锁又名递归锁，是指在同一个线程在外层方法获取锁的时候，在内层方法会自动获取锁

~~~java
// 此处用代码演示了可重入锁的代码层意思
synchronized void setA() throws Exception{   
    Thread.sleep(1000);
    setB();   // 因为获取了setA()的锁（即获取了方法外层的锁）、此时调用setB()将会自动获取setB()的锁，如果不自动获取的话方法B将不会执行           
}
 
synchronized void setB() throws Exception{
    Thread.sleep(1000);
}
~~~

#### 独享锁/共享锁

独享锁是指该锁一次只能被一个线程所持有

共享锁是指该锁可被多个线程所持有

#### 互斥锁/读写锁

独享锁/共享锁就是一种广义的说法，互斥锁/读写锁就是具体的实现

#### 乐观锁/悲观锁

乐观锁与[悲观锁](https://so.csdn.net/so/search?q=悲观锁&spm=1001.2101.3001.7020)不是指具体的什么类型的锁，而是指看待并发同步的角度。
悲观锁认为对于同一个数据的并发操作，一定是会发生修改的，哪怕没有修改，也会认为修改。因此对于同一个数据的并发操作，悲观锁采取加锁的形式。悲观的认为，不加锁的并发操作一定会出问题。
乐观锁则认为对于同一个数据的并发操作，是不会发生修改的。在更新数据的时候，会采用尝试更新，不断重新的方式更新数据。乐观的认为，不加锁的并发操作是没有事情的。

从上面的描述我们可以看出，悲观锁适合写操作非常多的场景，乐观锁适合读操作非常多的场景，不加锁会带来大量的性能提升。
悲观锁在Java中的使用，就是利用各种锁。
乐观锁在Java中的使用，是无锁编程，常常采用的是CAS算法，典型的例子就是原子类，通过CAS自旋实现原子操作的更新。

重量级锁是悲观锁的一种，自旋锁、轻量级锁与偏向锁属于乐观锁

#### 分段锁

分段锁其实是一种锁的设计，并不是具体的一种锁，对于ConcurrentHashMap而言，其并发的实现就是通过分段锁的形式来实现高效的并发操作。

ConcurrentHashMap中的分段锁称为Segment，它及类似于hashMap的结构。即内部拥有一个Entry数组，数组中的每个元素又是一个链表；同时又是一个ReentrantLock(Segment继承了ReentrantLock)。

当需要put元素的时候，并不是对整个hashmap加锁，而是先通过hashcode来知道他要放在那一个分段中，然后对这个分段进行加锁。所以当多线程put的时候，只要不是放在一个分段中，就实现了真正的并行的插入。

但是在统计size的时候，就是获取整个hashmap全局信息的时候，就需要就需要获取所有的分段锁才能统计。

分段锁的设计目的是细化锁的粒度，当操作不需要更新整个数组的时候，就仅仅针对数组中的一项进行加锁操作

#### 偏向锁/轻量级锁/重量级锁

这三种锁是指锁的状态，并且是针对**`Synchronized`**。在Java 5通过引入锁升级的机制来实现高效**`Synchronized`**。这三种锁的状态是通过对象监视器在对象头中的字段来表明的。
偏向锁是指一段同步代码一直被一个线程所访问，那么该线程会自动获取锁。降低获取锁的代价。

偏向锁的适用场景

始终只有一个线程在执行同步块，在它没有执行完释放锁之前，没有其它线程去执行同步块，在锁无竞争的情况下使用，一旦有了竞争就升级为轻量级锁，升级为轻量级锁的时候需要撤销偏向锁，撤销偏向锁的时候会导致stop the word操作； 
在有锁的竞争时，偏向锁会多做很多额外操作，尤其是撤销偏向所的时候会导致进入安全点，安全点会导致stw，导致性能下降，这种情况下应当禁用；

轻量级锁是指当锁是偏向锁的时候，被另一个线程所访问，偏向锁就会升级为轻量级锁，其他线程会通过自旋的形式尝试获取锁，不会阻塞，提高性能。
重量级锁是指当锁为轻量级锁的时候，另一个线程虽然是自旋，但自旋不会一直持续下去，当自旋一定次数的时候，还没有获取到锁，就会进入阻塞，该锁膨胀为重量级锁。重量级锁会让其他申请的线程进入阻塞，性能降低。

#### 自旋锁

在Java中，自旋锁是指尝试获取锁的线程不会立即阻塞，而是采用循环的方式去尝试获取锁，这样的好处是减少线程上下文切换的消耗，缺点是循环会消耗CPU。

自旋锁原理非常简单，如果持有锁的线程能在很短时间内释放锁资源，那么那些等待竞争锁的线程就不需要做内核态和用户态之间的切换进入阻塞挂起状态，它们只需要等一等（自旋），等持有锁的线程释放锁后即可立即获取锁，这样就**避免用户线程和内核的切换的消耗**。

自旋锁尽可能的减少线程的阻塞，适用于锁的竞争不激烈，且占用锁时间非常短的代码块来说性能能大幅度的提升，因为自旋的消耗会小于线程阻塞挂起再唤醒的操作的消耗

但是如果锁的竞争激烈，或者持有锁的线程需要长时间占用锁执行同步块，这时候就不适合使用自旋锁了，因为自旋锁在获取锁前一直都是占用cpu做无用功，同时有大量线程在竞争一个锁，会导致获取锁的时间很长，线程自旋的消耗大于线程阻塞挂起操作的消耗，其它需要cpu的线程又不能获取到cpu，造成cpu的浪费。

#### 熟悉几个概念以后我们开始详细说一下java中的锁：

我们所熟知的Java[锁机制](https://so.csdn.net/so/search?q=锁机制&spm=1001.2101.3001.7020)无非就是Sychornized 锁 和 Lock锁

Synchronized是基于JVM来保证数据同步的，而Lock则是在硬件层面，依赖特殊的CPU指令实现数据同步的

- Synchronized，它就是一个：非公平，悲观，独享，互斥，可重入的重量级锁
- ReentrantLock，它是一个：默认非公平但可实现公平的，悲观，独享，互斥，可重入，重量级锁。
- ReentrantReadWriteLocK，它是一个，默认非公平但可实现公平的，悲观，写独享，读共享，读写，可重入，重量级锁。

#### Synchronized的作用：

在JDK1.5之前都是使用synchronized关键字保证同步的

它可以把任意一个非NULL的对象当作锁。

1. 作用于方法时，锁住的是对象的实例(this)；
2. 当作用于静态方法时，锁住的是Class实例，又因为Class的相关数据存储在永久带PermGen（jdk1.8则是metaspace），永久带是全局共享的，因此静态方法锁相当于类的一个全局锁，会锁所有调用该方法的线程；
3. synchronized作用于一个对象实例时，锁住的是所有以该对象为锁的代码块。

#### Synchronized的实现：

实现如下图所示；

![Java锁机制](M:\qq截图有用\Java锁机制.png)

它有多个队列，当多个线程一起访问某个对象监视器的时候，对象监视器会将这些线程存储在不同的容器中。

1. Contention List：竞争队列，所有请求锁的线程首先被放在这个竞争队列中；
2. Entry List：Contention List中那些有资格成为候选资源的线程被移动到Entry List中；
3. Wait Set：哪些调用wait方法被阻塞的线程被放置在这里；
4. OnDeck：任意时刻，最多只有一个线程正在竞争锁资源，该线程被成为OnDeck；
5. Owner：当前已经获取到所资源的线程被称为Owner；
6. !Owner：当前释放锁的线程。

ContentionList并不是真正意义上的一个队列。仅仅是一个虚拟队列，它只有Node以及对应的Next指针构成，并没有Queue的数据结构。每次新加入Node会在队头进行，通过CAS改变第一个节点为新增节点，同时新增阶段的next指向后续节点，而取数据都在队列尾部进行。

JVM每次从队列的尾部取出一个数据用于锁竞争候选者（OnDeck），但是并发情况下，ContentionList会被大量的并发线程进行CAS访问，为了降低对尾部元素的竞争，JVM会将一部分线程移动到EntryList中作为候选竞争线程。Owner线程会在unlock时，将ContentionList中的部分线程迁移到EntryList中，并指定EntryList中的某个线程为OnDeck线程（一般是最先进去的那个线程）。Owner线程并不直接把锁传递给OnDeck线程，而是把锁竞争的权利交给OnDeck，OnDeck需要重新竞争锁。这样虽然牺牲了一些公平性，但是能极大的提升系统的吞吐量，在JVM中，也把这种选择行为称之为“竞争切换”。

OnDeck线程获取到锁资源后会变为Owner线程，而没有得到锁资源的仍然停留在EntryList中。如果Owner线程被wait方法阻塞，则转移到WaitSet队列中，直到某个时刻通过notify或者notifyAll唤醒，会重新进去EntryList中。

处于ContentionList、EntryList、WaitSet中的线程都处于阻塞状态，该阻塞是由操作系统来完成的（Linux内核下采用pthread_mutex_lock内核函数实现的）。该线程被阻塞后则进入内核调度状态，会导致系统在用户和内核之间进行来回切换，严重影响锁的性能。为了缓解上述性能问题，JVM引入了自旋锁(上边已经介绍过自旋锁)。原理非常简单，如果Owner线程能在很短时间内释放锁资源，那么哪些等待竞争锁的线程可以稍微等一等（自旋）而不是立即阻塞，当Owner线程释放锁后可立即获取锁，进而避免用户线程和内核的切换。但是Owner可能执行的时间会超过设定的阈值，争用线程在一定时间内还是获取不到锁，这是争用线程会停止自旋进入阻塞状态。基本思路就是先自旋等待一段时间看能否成功获取，如果不成功再执行阻塞，尽可能的减少阻塞的可能性，这对于占用锁时间比较短的代码块来说性能能大幅度的提升！

Synchronized在线程进入ContentionList时，等待的线程会先尝试自旋获取锁，如果获取不到就进入ContentionList，这明显对于已经进入队列的线程是不公平的，还有一个不公平的事情就是自旋获取锁的线程还可能直接抢占OnDeck线程的锁资源。

#### Lock锁简介：

与synchronized不同的是，Lock锁是纯Java实现的，与底层的JVM无关。在java.util.concurrent.locks包中有很多Lock的实现类，常用的有ReentrantLock、ReadWriteLock（实现类ReentrantReadWriteLock），其实现都依赖java.util.concurrent.AbstractQueuedSynchronizer类（简称AQS）

#### Lock的实现：

获取锁：首先判断当前状态是否允许获取锁，如果是就获取锁，否则就阻塞操作或者获取失败，也就是说如果是独占锁就可能阻塞，如果是共享锁就可能失败。另外如果是阻塞线程，那么线程就需要进入阻塞队列。当状态位允许获取锁时就修改状态，并且如果进了队列就从阻塞队列中移除。

```java
while (synchronization state does not allow acquire) {
    enqueue current thread if not already queued;
    possibly block current thread;
}
dequeue current thread if it was queued;
```

**释放锁**:这个过程就是修改状态位，如果有线程因为状态位阻塞的话，就唤醒队列中的一个或者更多线程。

```java
update synchronization state;
if (state may permit a blocked thread to acquire)
    unlock one or more queued threads;
```

阻塞和唤醒，JDK1.5之前的API中并没有阻塞一个线程，然后在将来的某个时刻唤醒它（wait/notify是基于synchronized下才生效的，在这里不算），JDK5之后利用JNI在LockSupport 这个类中实现了相关的特性！

![img](https://img-blog.csdn.net/20180804113234989?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQxMTgxNjE5/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

3、 有序队列：在AQS中采用ＣＬＨ队列来解决队列的有序问题。

我们来看下ReentrantLock的调用过程

经过源码分析，我们看到ReentrantLock把所有的Lock都委托给Sync类进行处理，该类继承自AQS，其类关系图如下

![img](https://img-blog.csdn.net/20180804113230394?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQxMTgxNjE5/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

其中Sync又有两个final static的子类NonfairSync和FairSync用于支持非公平锁和公平锁。我们先来挑一个看下对应Reentrant.lock()的调用过程（默认为非公平锁）

![img](https://img-blog.csdn.net/20180804113234703?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQxMTgxNjE5/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

这些模版很难让我们直观的看到整个调用过程，但是通过上面的过程图和AbstractQueuedSynchronizer的注释可以看出，AbstractQueuedSynchronizer抽象了大多数Lock的功能，而只把tryAcquire(int)委托给子类进行多态实现。tryAcquire用于判断对应线程事都能够获取锁，无论成功与否，AbstractQueuedSynchronizer都将处理后面的流程。

简单来讲，AQS会把所有请求锁的线程组成一个CLH的队列，当一个线程执行完毕释放锁(Lock.unlock())的时候，AQS会激活其后继节点，正在执行的线程不在队列当中，而那些等待的线程全部处于阻塞状态，经过源码分析，我们可以清楚的看到最终是通过LockSupport.park()实现的，而底层是调用sun.misc.Unsafe.park()本地方法，再进一步，HotSpot在Linux中中通过调用pthread_mutex_lock函数把线程交给系统内核进行阻塞。其运行示意图如下

![img](https://img-blog.csdn.net/20180804113233537?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQxMTgxNjE5/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

与synchronized相同的是，这个也是一个虚拟队列，并不存在真正的队列示例，仅存在节点之前的前后关系。（注：原生的CLH队列用于自旋锁，JUC将其改造为阻塞锁）。和synchronized还有一点相同的是，就是当获取锁失败的时候，不是立即进行阻塞，而是先自旋一段时间看是否能获取锁，这对那些已经在阻塞队列里面的线程显然不公平（非公平锁的实现，公平锁通过有序队列强制线程顺序进行），但会极大的提升吞吐量。如果自旋还是获取失败了，则创建一个节点加入队列尾部，加入方法仍采用CAS操作，并发对队尾CAS操作有可能会发生失败，AQS是采用自旋循环的方法，直到CAS成功！下面我们来看下锁的实现细节！

锁的实现依赖与lock()方法，Lock()方法首先是调用acquire(int)方法，不管是公平锁还是非公平锁

```java
public final void acquire(int arg) {
         if (!tryAcquire(arg) && acquireQueued(addWaiter(Node.EXCLUSIVE), arg))
             selfInterrupt();
}
```

Acquire()方法默认首先调用tryAcquire(int)方法，而此时公平锁和不公平锁的实现就不一样了。

#### Sync.NonfairSync.TryAcquire（非公平锁）

nonfairTryAcquire方法是lock方法间接调用的第一个方法，每次调用都会首先调用这个方法，我们来看下对应的实现代码：

```java
final boolean nonfairTryAcquire(int acquires) {
        final Thread current = Thread.currentThread();
        int c = getState();
        if (c == 0) {
            if (compareAndSetState(0, acquires)) {
                setExclusiveOwnerThread(current);
                return true;
            }
        }
        else if (current == getExclusiveOwnerThread()) {
            int nextc = c + acquires;
            if (nextc < 0) // overflow
                throw new Error("Maximum lock count exceeded");
            setState(nextc);
            return true;
        }
        return false;
    }
```

该方法首先会判断当前线程的状态，如果c==0 说明没有线程正在竞争锁。（反过来，如果c!=0则说明已经有其他线程已经拥有了锁）。如果c==0，则通过CAS将状态设置为acquires(独占锁的acquires为1)，后续每次重入该锁都会+1，每次unlock都会-1，当数据为0时则释放锁资源。其中精妙的部分在于：并发访问时，有可能多个线程同时检测到c为0，此时执行compareAndSetState(0, acquires))设置，可以预见，如果当前线程CAS成功，则其他线程都不会再成功，也就默认当前线程获取了锁，直接作为running线程，很显然这个线程并没有进入等待队列。如果c!=0，首先判断获取锁的线程是不是当前线程，如果是当前线程，则表明为锁重入，继续+1，修改state的状态，此时并没有锁竞争，也非CAS，因此这段代码也非常漂亮的实现了偏向锁。