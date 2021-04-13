# Object 中的Wait 和 Thread 中的 Sleep解释


## Object 方法`native void wait(long timeout)`
Causes the current thread to wait until either another thread invokes the notify() method or the notifyAll() method 
for this object, or a specified amount of time has elapsed.
The current thread must own this object's monitor.
会导致当前线程进入等待状态，直到另一个线程触发这个线程对象的`notify()`方法，或` notifyAll()`方法，或者wait等待的时间到了或超时了。
调用`wait()`的线程必须持有这个对象锁（monitor）

This method causes the current thread (call it T) to place itself in the wait set for this object and then to 
relinquish any and all synchronization claims on this object. Thread T becomes disabled for thread scheduling 
purposes and lies dormant until one of four things happens:
这个方法会使当前线程（T）把自己置于等待这个对象的线程集中，并放弃对这个对象的所有串行执行的声明。线程T停止工作进入休眠，并等待直到下面四种情况发生：

+ Some other thread invokes the notify method for this object and thread T happens 
to be arbitrarily chosen as the thread to be awakened.
+ 另一个线程触发notify方法，二线程T刚好被随机选中成为被唤醒线程
+ Some other thread invokes the notifyAll method for this object.
+ 另一个线程触发这个对象的`notifyAll`方法
+ Some other thread interrupts thread T.
+ 另一个线程中断了线程T
+ The specified amount of real time has elapsed, more or less. If timeout is zero, 
however, then real time is not taken into consideration and the thread simply waits until notified.
+ 指定等待的时间已经过去，(可能多一点、或少一点)。如果指定的时间是0，则真实等待的时间就不在考虑之中了，会一直等待，直到对象被通知唤醒。

The thread T is then removed from the wait set for this object and re-enabled for thread scheduling. 
It then competes in the usual manner with other threads for the right to synchronize on the object; 
once it has gained control of the object, all its synchronization claims on the object are restored 
to the status quo ante - that is, to the situation as of the time that the wait method was invoked. 
Thread T then returns from the invocation of the wait method. Thus, on return from the wait method, 
the synchronization state of the object and of thread T is exactly as it was when the wait method was invoked.
A thread can also wake up without being notified, interrupted, or timing out, a so-called spurious wakeup. 
While this will rarely occur in practice, applications must guard against it by testing for the condition 
that should have caused the thread to be awakened, and continuing to wait if the condition is not satisfied. 
In other words, waits should always occur in loops, like this one:
当线程T被重新调度启用时，会重新从等待线程集中移除。然后与其他持有这个对象的线程竞争同步这个对象(锁上对象)的权限。
一旦它获得了对象的控制权，它对对象的所有同步声明都将恢复到之前的状态—即，到调用wait方法时的情况。
然后线程T从wait方法的调用中返回。因此，从wait方法返回时，对象和线程T的同步状态与调用wait方法时完全相同。
线程也可以在不被通知、中断或超时的情况下被唤醒，这就是所谓的伪唤醒。
虽然这种情况在实践中很少发生，但应用程序必须通过测试该条件来防止这种情况发生，这应该会导致线程被唤醒，如果条件不满足，线程将继续等待。
换一种说法，等待总数发生在循环中，就像这样：

```
           synchronized (obj) {
               while (<condition does not hold>)
                   obj.wait(timeout);
               ... // Perform action appropriate to condition
           }
```
       
(For more information on this topic, see Section 3.2.3 in Doug Lea's "Concurrent Programming in Java (Second Edition)" 
(Addison-Wesley, 2000), or Item 50 in Joshua Bloch's "Effective Java Programming Language Guide" (Addison-Wesley, 2001).
(有关此主题的更多信息，请参见Doug Lea的“Java并发编程(第二版)”中的3.2.3节。(Addison-Wesley, 2000)，
或者Joshua Bloch的“有效的Java编程语言指南”(Addison-Wesley, 2001)中的第50项。)

If the current thread is interrupted by any thread before or while it is waiting, then an InterruptedException is thrown. 
This exception is not thrown until the lock status of this object has been restored as described above.
Note that the wait method, as it places the current thread into the wait set for this object, unlocks only this object; 
any other objects on which the current thread may be synchronized remain locked while the thread waits.
This method should only be called by a thread that is the owner of this object's monitor. 
See the notify method for a description of the ways in which a thread can become the owner of a monitor.
如果当前线程在等待过程中被其他线程打断，就会抛出`InterruptedException`异常。
直到该对象的锁状态恢复上述状态后，才会抛出此异常。注意，`wait()`方法在将当前线程放入该对象的等待集中时，只解锁该对象;
在线程等待时，当前线程可能被同步的任何其他对象都保持锁定状态。此方法应仅由该对象的monitor的所有者线程调用。
请参阅`notify`方法，以获取线程成为监视器所有者的方式的描述。


## Object 方法`final void notify()`

Wakes up a single thread that is waiting on this object's monitor. If any threads are waiting on this object, 
one of them is chosen to be awakened. The choice is arbitrary and occurs at the discretion of the implementation. 
A thread waits on an object's monitor by calling one of the wait methods.
唤醒一个在等待当前对象的(等待获取monitor)的线程, 如果有多个线程正在等待这个对象，就会唤起其中的一个唤醒，这种选择是随意的，由执行的自由裁决量权决定。
一个线程会在调用对象的`wait()`方法后进入等待获取当前对象的monitor的线程集合中。

The awakened thread will not be able to proceed until the current thread relinquishes the lock on this object. 
The awakened thread will compete in the usual manner with any other threads that might be actively competing to synchronize on this object; 
for example, the awakened thread enjoys no reliable privilege or disadvantage in being the next thread to lock this object.
This method should only be called by a thread that is the owner of this object's monitor. 
A thread becomes the owner of the object's monitor in one of three ways:
唤醒的线程在获取对象锁之前，是不能执行任何动作的，直到前一个线程放弃对象锁。
唤醒的线程会与其他一起被唤醒的线程共同竞争占有当前对象，进行同步操作。
例如,唤醒的线程在占有当前对象的monitor之前不会有任何优劣或是特权，notify方法只能被占有当前对象`monitor`的线程调用。
线程成为当前对象`monitor`有以下几种方式

+ By executing a synchronized instance method of that object.
+ 执行对象实例中同步方法(一般被`synchronized`标记)
+ By executing the body of a synchronized statement that synchronizes on the object.
+ 执行同步代码块(一般被`synchronized`标记)
+ For objects of type Class, by executing a synchronized static method of that class.
+ 对于Class类型的对象，可以执行该类的同步静态方法。

Only one thread at a time can own an object's monitor.
只能有一个线程占有一个实例对象的`monitor`

## Object 方法`final void notifyAll()`
Wakes up all threads that are waiting on this object's monitor. A thread waits on an object's monitor by calling one of the wait methods.
唤醒所有在此对象的监视器上等待的线程。线程通过调用一个等待方法在对象的监视器上等待。

The awakened threads will not be able to proceed until the current thread relinquishes the lock on this object. The awakened threads will compete in the usual manner with any other threads that might be actively competing to synchronize on this object; for example, the awakened threads enjoy no reliable privilege or disadvantage in being the next thread to lock this object.
在当前线程放弃对该对象的锁之前，被唤醒的线程将不能继续执行。被唤醒的线程将以通常的方式与任何其他可能正在主动竞争同步这个对象的线程竞争;例如，被唤醒的线程在成为下一个锁定该对象的线程时没有任何可靠的特权或劣势。

This method should only be called by a thread that is the owner of this object's monitor. See the notify method for a description of the ways in which a thread can become the owner of a monitor.
此方法应仅由该对象的监视器的所有者线程调用。请参阅notify方法，以获取线程成为监视器所有者的方式的描述。


## Thread 方法 `native void sleep(long millis) throws InterruptedException`

Causes the currently executing thread to sleep (temporarily cease execution) for the specified number of milliseconds, 
subject to the precision and accuracy of system timers and schedulers. 
The thread does not lose ownership of any monitors.
使当前正在执行的线程进入指定时间的睡眠（暂时停止执行）状态，实际需要遵从系统计时器和调度器的精度和准确性
线程不会放弃对象monitor的占有权

