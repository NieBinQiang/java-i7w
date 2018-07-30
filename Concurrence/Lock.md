## java锁机制
java中有两种方法实现锁机制：
- `synchronized`
- 比`ynchronized`更加强大和灵活的`Lock`
`Lock`确保当一个线程位于代码的临界区时，另一个线程不进入临界区，相当于`synchronized`，`Lock`接口及其实现类提供了更加强大、灵活的锁机制。

## Lock的使用

```java
public class ThreadTest {
    Lock lock = new Lock();
    public void test(){
        lock.lock();
        //do something
        lock.unlock();
    }

```
lock()方法会对`Lock`实例对象进行加锁，因此所有对该对象调用lock()方法的线程都会被阻塞，直到该`Lock`对象的unlock()方法被调用。
    
```java
public class Lock{
    private boolean isLocked = false;
 
    public synchronized void lock() throws InterruptedException{
        while(isLocked){
            wait();
        }
        isLocked = true;
    }
 
    public synchronized void unlock(){
        isLocked = false;
        notify();
    }

```
## 锁的公平性
- **公平**，所有线程均可以公平地获得`CPU`运行机会，公平性的对立面是饥饿。
- **饥饿**，如果一个线程因为其他线程在一直抢占着`CPU`而得不到`CPU`运行时间，那么我们就称该线程被“饥饿致死”。

### 饥饿的原因
- **高优先级线程吞噬所有的低优先级线程的`CPU`时间**。
- **线程被永久堵塞在一个等待进入同步块的状态**。`java`的同步代码块并不会保证进入它的线程的先后顺序。这就意味着理论上存在一个或者多个线程在试图进入同步代码区时永远被堵塞着，因为其他线程总是不断优于他获得访问权，导致它一直得到不到`CPU`运行机会被“饥饿致死”。
- **线程在等待一个本身也处于永久等待完成的对象**。存在这样一个风险：一个等待线程从来得不到唤醒，因为其他等待线程总是能被获得唤醒。

### 锁的可重入性
“可重入”意味着自己可以再次获得自己的内部锁，而不需要阻塞。

`java`多线程的可重入性的实现是通过每个锁**关联一个请求计数器和一个占有它的线程**，当计数为0时，认为该锁是没有被占有的，那么任何线程都可以获得该锁的占有权。当某一个线程请求成功后，`JVM`会记录该锁的持有线程 并且将计数设置为1，如果这时其他线程请求该锁时则必须等待。当该线程再次请求请求获得锁时，计数会+1；当占有线程退出同步代码块时，计数就会-1，直到为0时，释放该锁。这时其他线程才有机会获得该锁的占有权。

## Lock的架构
![image](https://img-blog.csdn.net/20150810172836095?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQv/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)
