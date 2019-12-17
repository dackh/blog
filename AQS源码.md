# ReentrantLock
## 结构

``` Java
public class ReentrantLock implements Lock, java.io.Serializable {
    private static final long serialVersionUID = 7373984872572414699L;
    /** Synchronizer providing all implementation mechanics */
    private final Sync sync;

    /**
     * Base of synchronization control for this lock. Subclassed
     * into fair and nonfair versions below. Uses AQS state to
     * represent the number of holds on the lock.
     */
    abstract static class Sync extends AbstractQueuedSynchronizer {...}

    /**
     * Sync object for non-fair locks
     */
    static final class NonfairSync extends Sync {...}

    /**
     * Sync object for fair locks
     */
    static final class FairSync extends Sync {...}
```
这里可以看出`ReentranLock`类中一个抽象类`Sync`来继承`AbstractQueuedSynchronizer`类，并且公平锁跟非公平锁分别继承于`Sync`类。

``` Java
    /**
     * Creates an instance of {@code ReentrantLock}.
     * This is equivalent to using {@code ReentrantLock(false)}.
     */
    public ReentrantLock() {
        sync = new NonfairSync();
    }

    /**
     * Creates an instance of {@code ReentrantLock} with the
     * given fairness policy.
     *
     * @param fair {@code true} if this lock should use a fair ordering policy
     */
    public ReentrantLock(boolean fair) {
        sync = fair ? new FairSync() : new NonfairSync();
    }
```
从这ReentrantLock构造方法中可以看出，如果不指定锁类型，那么默认新建非公平锁，同时可以通过参数指定创建公平锁，并赋值给`sync`对象，公平锁跟非公平锁的区别提现在：`...//TODO`，下面我们从源码的角度来分析。