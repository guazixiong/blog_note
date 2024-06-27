# Java中的wait()和notify()方法：为什么必须在synchronized块中使用

Java多线程编程中的`wait()`和`notify()`方法是协调不同线程之间的交互的重要工具。这些方法使一个线程能够暂停执行（通过`wait()`）直到另一个线程通知某个条件已经改变（通过`notify()`或`notifyAll()`）。理解这些方法及其使用上的要求对于编写可靠和高效的多线程程序至关重要。

## 为什么wait()和notify()需要在synchronized块中使用

这些方法必须在`synchronized`块或方法中调用，主要基于以下几个原因：

### 1. 确保线程安全

使用`synchronized`块确保在任何时候只有一个线程可以访问关键的代码段。这是因为`wait()`和`notify()`操作通常涉及到修改或检查共享资源的状态。如果多个线程同时修改资源，可能会导致数据不一致和程序错误。

### 2. 正确的锁管理

- **调用`wait()`**：线程通过`wait()`进入等待状态前必须持有锁。在等待期间，线程会释放锁，允许其他线程进入`synchronized`块并有机会修改条件状态或调用`notify()`。
- **调用`notify()`或`notifyAll()`**：线程必须持有相同的对象锁来调用这些方法，以确保唤醒操作只在资源状态安全地更新后才进行。

### 3. 避免`IllegalMonitorStateException`

如果线程尝试在不持有适当对象锁的情况下调用这些方法，Java运行时会抛出`IllegalMonitorStateException`。这个设计强制开发者在合适的同步环境中使用`wait()`和`notify()`，避免潜在的线程间干扰和资源竞争。



### wait()的情境与死锁风险

1. **正确释放和获取锁**：`wait()`方法的设计意图是让当前线程放弃对对象的锁定，并且阻塞，直到其他线程调用同一个对象的`notify()`或`notifyAll()`方法。如果线程在未持有对象锁的情况下调用`wait()`：
   - 该线程不会释放任何锁（因为它没有锁可释放），从而导致它自身无法继续执行。
   - 同时，如果其他需要这个锁来继续执行的线程因为这个未释放的锁而阻塞，就可能发生死锁。
2. **保证操作顺序**：在线程调用`wait()`之前，它通常会检查某种状态是否满足，然后决定是否需要等待。这个检查和决定等待的过程需要在一个原子操作中完成（即没有其他线程可以介入）。如果在没有锁的情况下进行，就可能导致状态在检查和调用`wait()`之间被其他线程改变，从而引入错误。

### notify()的情境与必要性

1. **确保操作的有效性**：当线程调用`notify()`或`notifyAll()`时，它必须确保已经或即将改变其他线程等待的条件。持有锁可以保证在通知其他线程之前，状态已经安全地改变，避免提前或错误地通知。
2. **防止错误的唤醒**：如果没有持有锁的情况下进行通知，可能存在通知“丢失”的情况，即在调用`notify()`之后，其他线程还没有开始等待就已经错过了通知，导致程序逻辑错误或永久阻塞。

### 为何必须在持有锁的情况下调用`notify()`

1. **同步访问共享资源**：
   - 在多线程编程中，`notify()`通常用于告知一个或多个等待线程某个条件已经成立，可以继续执行。但是，这个“条件”的改变通常涉及对共享资源的修改。
   - 如果一个线程在不持有锁的情况下调用`notify()`，那么它可能在另一个线程还在修改共享资源的过程中就发出了通知。这可能导致接收通知的线程在共享资源尚未完全更新好的情况下继续执行，从而引发数据不一致或竞争条件。
2. **确保可见性和有序性**：
   - 锁的机制不仅保证了互斥（即一次只有一个线程可以执行代码块），还保证了内存的可见性。这意味着在持有锁时所做的更改对于随后获得同一锁的其他线程是可见的。
   - 如果一个线程在调用`notify()`之前修改了某个条件，并且这个修改必须对其他线程可见，那么在调用`notify()`之前必须确保这些修改已经被刷新到主内存中。只有在持有锁的情况下，才能保证这种刷新和可见性。
3. **避免通知丢失**：
   - 如果在没有同步的情况下调用`notify()`，可能存在“通知丢失”的问题。即通知可能在等待状态线程实际进入等待之前发出，导致该线程可能永远不会被唤醒。
   - 通过在修改条件后，在同一同步块中调用`notify()`，可以确保当通知发出时，有线程正在或即将等待那个条件。

## 示例代码

以下是一个简单的示例，展示如何在Java中正确使用`wait()`和`notify()`方法：

```java
public class WaitNotifyExample {
    private static final Object lock = new Object();

    public static void main(String[] args) {
        Thread waitingThread = new Thread(() -> {
            synchronized (lock) {
                try {
                    System.out.println("Thread is waiting for notification.");
                    lock.wait();
                    System.out.println("Thread was notified and is running again.");
                } catch (InterruptedException e) {
                    Thread.currentThread().interrupt();
                }
            }
        });

        Thread notifyingThread = new Thread(() -> {
            synchronized (lock) {
                try {
                    Thread.sleep(1000); // Simulate work
                    System.out.println("Thread is notifying.");
                    lock.notify();
                } catch (InterruptedException e) {
                    Thread.currentThread().interrupt();
                }
            }
        });

        waitingThread.start();
        notifyingThread.start();
    }
}
```

## 总结

正确使用`wait()`和`notify()`方法不仅关乎代码的正确性，也是实现高效并发处理的关键。通过强制在`synchronized`块中使用这些方法，Java提供了一种机制，确保线程在访问和修改共享资源时能够保持正确的同步。这有助于开发者编写出更安全、更稳定的多线程应用。