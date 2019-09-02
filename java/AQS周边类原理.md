# AQS周边类原理
参考文档：https://www.cnblogs.com/zaizhoumo/p/7782941.html

# ReentrantLock
ReentrantLock实现了两种锁方式，默认使用的是非公平锁。 

* lock()：  
直接调用AQS的compareAndSetState(0,1)来设置state，成功则锁定，失败掉用AQS的acquire方法，acquire首先调用NonfairSync.tryAcquire来判定，如果获取锁失败则调用AQS的acquireQueued(addWaiter(Node.EXCLUSIVE),arg))来将当前线程放入Node链表并阻塞当前线程。  
* unlock()：  
调用ReentrantLock.tryRelease(1)来更改state，并调用unparkSuccessor(h)来唤醒Node链表中阻塞的线程。


# ReentrantReadWriteLock
ReentrantReadWriteLock是lock的另一种实现方式，因为ReentrantLock是排它锁，同一时间只允许一个线程访问，而ReentrantReadWriteLock允许多个读线程同时访问，但不允许写线程和读线程，写线程和写线程同时访问。相对于排它锁提高了并发性能。  
ReentrantReadWriteLock内部维护了两个锁，一个用于读，有一个用于写。ReentrantReadWriteLock支持以下功能：  
1. 支持公平和非公平的获取锁的方式。
2. 支持可重入，读线程在获取了读锁后还可以在获取读锁，写线程在获取写锁后还可以获取写锁，或者可以再次获取读锁
3. 支持写锁降级为读锁，但不支持读锁升级为写锁
4. 读取锁和写入锁都支持获取锁期间的中断。
5. condition支持，仅写入锁提供了一个condition实现，读锁不支持condition

ReentrantReadWriteLock也是基于AQS实现的，他实现的重点在于**通过将AQS中state拆分成两部分来分别存储多个读线程和一个写线程状态信息**，其中高16位标识读，低16位标识写。

## 写锁的获取和释放

### 获取写锁
```
//获取写锁
public void lock() {
    sync.acquire(1);
}

//AQS实现的独占式获取同步状态方法
public final void acquire(int arg) {
    if (!tryAcquire(arg) &&
        acquireQueued(addWaiter(Node.EXCLUSIVE), arg))
        selfInterrupt();
}

//自定义重写的tryAcquire方法
protected final boolean tryAcquire(int acquires) {
    Thread current = Thread.currentThread();
    int c = getState();
    int w = exclusiveCount(c);    //取同步状态state的低16位，写同步状态
    if (c != 0) {
        // (Note: if c != 0 and w == 0 then shared count != 0)
        //存在读锁或当前线程不是已获取写锁的线程，返回false
        if (w == 0 || current != getExclusiveOwnerThread())
            return false;
        //判断同一线程获取写锁是否超过最大次数，支持可重入
        if (w + exclusiveCount(acquires) > MAX_COUNT)    //
            throw new Error("Maximum lock count exceeded");
        // Reentrant acquire
        setState(c + acquires);
        return true;
    }
    //此时c=0,读锁和写锁都没有被获取
    if (writerShouldBlock() ||
        !compareAndSetState(c, c + acquires))
        return false;
    setExclusiveOwnerThread(current);
    return true;
}
```

从源代码可以看出，获取写锁的步骤如下：
1. 判断同步状态state是否为0。如果state!=0，说明已经有其他线程获取了读锁或写锁，执行2）；否则执行5）。
2. 判断同步状态state的低16位（w）是否为0。如果w=0，说明其他线程获取了读锁，返回false；如果w!=0，说明其他线程获取了写锁，执行步骤3）。
3. 判断获取了写锁是否是当前线程，若不是返回false，否则执行4）；
4. 判断当前线程获取写锁是否超过最大次数，若超过，抛异常，反之更新同步状态（此时当前线程以获取写锁，更新是线程安全的），返回true。
5. 此时读锁或写锁都没有被获取，判断是否需要阻塞（公平和非公平方式实现不同），如果不需要阻塞，则CAS更新同步状态，若CAS成功则返回true，否则返回false。如果需要阻塞则返回false。

writerShouldBlock() 判断当前线程是否需要阻塞，NofairSync和FairSync中有不同的实现

```
//FairSync中需要判断是否有前驱节点，如果有则返回false，否则返回true。遵循FIFO
final boolean writerShouldBlock() {
    return hasQueuedPredecessors();
}

//NonfairSync中直接返回false，可插队。
final boolean writerShouldBlock() {
    return false; // writers can always barge
}
```

### 释放写锁
```
//写锁释放
public void unlock() {
    sync.release(1);
}

//AQS提供独占式释放同步状态的方法
public final boolean release(int arg) {
    if (tryRelease(arg)) {
        Node h = head;
        if (h != null && h.waitStatus != 0)
            unparkSuccessor(h);
        return true;
    }
    return false;
}

//自定义重写的tryRelease方法
protected final boolean tryRelease(int releases) {
    if (!isHeldExclusively())
        throw new IllegalMonitorStateException();
    int nextc = getState() - releases;    //同步状态减去releases
    //判断同步状态的低16位（写同步状态）是否为0，如果为0则返回true，否则返回false.
    //因为支持可重入
    boolean free = exclusiveCount(nextc) == 0;    
    if (free)
        setExclusiveOwnerThread(null);
    setState(nextc);    //以获取写锁，不需要其他同步措施，是线程安全的
    return free;
}
```

## 读锁的获取与释放
读锁是一个可重入的共享锁，采用AQS提供的共享式获取同步状态的策略。

### 获取读锁
```
public void lock() {
    sync.acquireShared(1);
}

//使用AQS提供的共享式获取同步状态的方法
public final void acquireShared(int arg) {
    if (tryAcquireShared(arg) < 0)
        doAcquireShared(arg);
}

//自定义重写的tryAcquireShared方法，参数是unused，因为读锁的重入计数是内部维护的
protected final int tryAcquireShared(int unused) {
    Thread current = Thread.currentThread();
    int c = getState();
    //exclusiveCount(c)取低16位写锁。存在写锁且当前线程不是获取写锁的线程，返回-1，获取读锁失败。
    if (exclusiveCount(c) != 0 &&
        getExclusiveOwnerThread() != current)
        return -1;
    int r = sharedCount(c);    //取高16位读锁，
    //readerShouldBlock（）用来判断当前线程是否应该被阻塞
    if (!readerShouldBlock() &&
        r < MAX_COUNT &&    //MAX_COUNT为获取读锁的最大数量，为16位的最大值
        compareAndSetState(c, c + SHARED_UNIT)) {
        //firstReader是不会放到readHolds里的, 这样，在读锁只有一个的情况下，就避免了查找readHolds。
        if (r == 0) {    // 是 firstReader，计数不会放入  readHolds。
            firstReader = current;
            firstReaderHoldCount = 1;
        } else if (firstReader == current) {    //firstReader重入
            firstReaderHoldCount++;
        } else {
            // 非 firstReader 读锁重入计数更新
            HoldCounter rh = cachedHoldCounter;    //读锁重入计数缓存，基于ThreadLocal实现
            if (rh == null || rh.tid != current.getId())
                cachedHoldCounter = rh = readHolds.get();
            else if (rh.count == 0)
                readHolds.set(rh);
            rh.count++;
        }
        return 1;
    }
    //第一次获取读锁失败，有两种情况：
    //1）没有写锁被占用时，尝试通过一次CAS去获取锁时，更新失败（说明有其他读锁在申请）
    //2）当前线程占有写锁，并且有其他写锁在当前线程的下一个节点等待获取写锁，除非当前线程的下一个节点被取消，否则fullTryAcquireShared也获取不到读锁
    return fullTryAcquireShared(current);
}
```

从源码可以看出，获取读锁的大致步骤如下：  
1. 通过判断同步状态低16位，如果存在写锁且当前线程不是获取写锁的线程，返回-1，获取读锁失败；否则执行步骤2
2. 通过readerShouldBlock判断当前线程是否被阻塞，如果不应该阻塞则尝试CAS同步状态，否则执行3
3. 第一次获取读锁失败，通过FullTryAcquireShared再次尝试获取锁

readerShouldBlock方法用来判断当前线程是否因该被阻塞，NonfairSync和FairySync中有不同的实现

```
//FairSync中需要判断是否有前驱节点，如果有则返回false，否则返回true。遵循FIFO
final boolean readerShouldBlock() {
    return hasQueuedPredecessors();
}
final boolean readerShouldBlock() {
    return apparentlyFirstQueuedIsExclusive();
}
//当head节点不为null且head节点的下一个节点s不为null且s是独占模式（写线程）且s的线程不为null时，返回true。
//目的是不应该让写锁始终等待。作为一个启发式方法用于避免可能的写线程饥饿，这只是一种概率性的作用，因为如果有一个等待的写线程在其他尚未从队列中出队的读线程后面等待，那么新的读线程将不会被阻塞。
final boolean apparentlyFirstQueuedIsExclusive() {
    Node h, s;
    return (h = head) != null &&
        (s = h.next)  != null &&
        !s.isShared()         &&
        s.thread != null;
}
```

FullTryAcquireShared方法

```
final int fullTryAcquireShared(Thread current) {
    /*
     * This code is in part redundant with that in
     * tryAcquireShared but is simpler overall by not
     * complicating tryAcquireShared with interactions between
     * retries and lazily reading hold counts.
     */
    HoldCounter rh = null;
    for (;;) {
        int c = getState();
        //如果当前线程不是写锁的持有者，直接返回-1，结束尝试获取读锁，需要排队去申请读锁
        if (exclusiveCount(c) != 0) {
            if (getExclusiveOwnerThread() != current)
                return -1;
            // else we hold the exclusive lock; blocking here
            // would cause deadlock.
        //如果需要阻塞，说明除了当前线程持有写锁外，还有其他线程已经排队在申请写锁，故，即使申请读锁的线程已经持有写锁（写锁内部再次申请读锁，俗称锁降级）还是会失败，因为有其他线程也在申请写锁，此时，只能结束本次申请读锁的请求，转而去排队，否则，将造成死锁。
        } else if (readerShouldBlock()) {
            // Make sure we're not acquiring read lock reentrantly
            if (firstReader == current) {
                //如果当前线程是第一个获取了写锁，那其他线程无法申请写锁
                // assert firstReaderHoldCount > 0;
            } else {
                //从readHolds中移除当前线程的持有数，然后返回-1，然后去排队获取读锁。
                if (rh == null) {
                    rh = cachedHoldCounter;
                    if (rh == null || rh.tid != current.getId()) {
                        rh = readHolds.get();
                        if (rh.count == 0)
                            readHolds.remove();
                    }
                }
                if (rh.count == 0)
                    return -1;
            }
        }
        if (sharedCount(c) == MAX_COUNT)
            throw new Error("Maximum lock count exceeded");
        if (compareAndSetState(c, c + SHARED_UNIT)) {
            //示成功获取读锁，后续就是更新readHolds等内部变量，
            if (sharedCount(c) == 0) {
                firstReader = current;
                firstReaderHoldCount = 1;
            } else if (firstReader == current) {
                firstReaderHoldCount++;
            } else {
                if (rh == null)
                    rh = cachedHoldCounter;
                if (rh == null || rh.tid != current.getId())
                    rh = readHolds.get();
                else if (rh.count == 0)
                    readHolds.set(rh);
                rh.count++;
                cachedHoldCounter = rh; // cache for release
            }
            return 1;
        }
    }
}
```

### 释放读锁

```
public  void unlock() {
    sync.releaseShared(1);
}

public final boolean releaseShared(int arg) {
    if (tryReleaseShared(arg)) {
        doReleaseShared();
        return true;
    }
    return false;
}

protected final boolean tryReleaseShared(int unused) {
    Thread current = Thread.currentThread();
    //更新计数
    if (firstReader == current) {
        // assert firstReaderHoldCount > 0;
        if (firstReaderHoldCount == 1)
            firstReader = null;
        else
            firstReaderHoldCount--;
    } else {
        HoldCounter rh = cachedHoldCounter;
        if (rh == null || rh.tid != current.getId())
            rh = readHolds.get();
        int count = rh.count;
        if (count <= 1) {
            readHolds.remove();
            if (count <= 0)
                throw unmatchedUnlockException();
        }
        --rh.count;
    }
    //自旋CAS，减去1<<16
    for (;;) {
        int c = getState();
        int nextc = c - SHARED_UNIT;
        if (compareAndSetState(c, nextc))
            // Releasing the read lock has no effect on readers,
            // but it may allow waiting writers to proceed if
            // both read and write locks are now free.
            return nextc == 0;
    }
}
```


# Semaphore
Semaphore实现同一时间的线程并发量控制，其内部同样有公平和非公平锁实现，默认采用非公平锁。 

* Semaphore(permits)：  
permits同样存放在AQS的state中  
* acquire()：  
调用AQS.acquireSharedInterruptibly(1)，反调Semaphore中的Sync.nonfairTryAcquireShared来对state进行减操作，并根据结果来判断是否需要阻塞，阻塞则调用AQS.doAcquireSharedInterruptibly进行阻塞。  
* release()：  
调用AQS.releaseShared反调Sync.tryReleaseShared对state进行加操作，并返回true，继续调用AQS.doReleaseShared()循环遍历Node链表进行unpark操作。

# CountDownLatch
CountDownLatch的阻塞条件数量存放在AQS的state中，CountDownLatch通过实现AQS中的tryAcquireShared，tryReleaseShared方法来实现阻塞和释放的判断(内部实现就是查询或者更改state值)。并具体通过调用AQS的doAcquireSharedInterruptibly，doReleaseShared来最终实现阻塞和解禁。

