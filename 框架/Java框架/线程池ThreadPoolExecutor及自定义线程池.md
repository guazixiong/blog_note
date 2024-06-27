# 线程池ThreadPoolExecutor及自定义线程池



## **线程池参数**

1. **corePoolSize**:线程池核心线程数量，核心线程不会被回收，即使没有任务执行，也会保持空闲状态。
2. **maximumPoolSize**:**线程池允许最大的线程数**，当线程数量达到corePoolSize，且**workQueue队列塞满任务**了之后，继续**创建线程**,优先级提高。
3. **keepAliveTime**:超过corePoolSize之后的**存活时间**。
4. **unit**:keepAliveTime的单位。
5. **workQueue**:当前线程数超过corePoolSize时，新的任务会处在等待状态，并存在workQueue中
6. **threadFactory:创建线程的工厂类**
7. **handler**:线程池执行**拒绝策略**.当线数量达到maximumPoolSize大小，并且workQueue也已经塞满了任务的情况下，线程池会调用handler拒绝策略来处理请求。

##  **常见的拒绝策略:**

- AbortPolicy：为线程池默认的拒绝策略，该策略直接抛异常处理。
- DiscardPolicy：直接抛弃不处理。
- DiscardOldestPolicy：丢弃队列中最老的任务。
- CallerRunsPolicy：将任务分配给当前执行execute方法线程来处理。

## 自定义线程池

在**线程池的基础**上,自定义线程池

通过ThreadPoolExecutor创建线程池.

```java
        Executors.newFixedThreadPool(10);
        Executors.newCachedThreadPool();
        Executors.newSingleThreadExecutor();
        Executors.newWorkStealingPool();
        Executors.newScheduledThreadPool(10);
```
newCachedThreadPool()源码
```java
    public static ExecutorService newCachedThreadPool() {
        return new ThreadPoolExecutor(0, Integer.MAX_VALUE,
                                      60L, TimeUnit.SECONDS,
                                      new SynchronousQueue<Runnable>());
    }
```
newFixedThreadPool()源码
```java
    public static ExecutorService newFixedThreadPool(int nThreads) {
        return new ThreadPoolExecutor(nThreads, nThreads,
                                      0L, TimeUnit.MILLISECONDS,
                                      new LinkedBlockingQueue<Runnable>());
    }
```
共同点:

```java
        return new ThreadPoolExecutor(nThreads, nThreads,
                                      0L, TimeUnit.MILLISECONDS,
                                      new LinkedBlockingQueue<Runnable>());
```

依据源码自定义线程池:

```java
new ThreadPoolExecutor(corePoolSize, maximumPoolSize, keepAliveTime, unit, workQueue);
```

自定义线程,实现Runnable
```java
public class MyTask implements Runnable {

    //自定义线程ID
    private int taskId;
    //线程名称
    private String taskName;

    public MyTask(int taskId, String taskName) {
        this.taskId = taskId;
        this.taskName = taskName;
    }

    public int getTaskId() {
        return taskId;
    }

    public void setTaskId(int taskId) {
        this.taskId = taskId;
    }

    public String getTaskName() {
        return taskName;
    }

    public void setTaskName(String taskName) {
        this.taskName = taskName;
    }
    public void run() {
        try {
            System.out.println("当前线程Id-->" + taskId + ",任务名称-->" + taskName);
            Thread.sleep(5 * 1000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }
}
```
main函数
```java

public class CustomerThreadPool {

    public static void main(String[] args) {
        //这里使用的是有界队列,不是无界队列
        ThreadPoolExecutor pool = new ThreadPoolExecutor(2, 2, 60, TimeUnit.SECONDS, new ArrayBlockingQueue<>(3));

        MyTask task1 = new MyTask(1, "任务1");
        MyTask task2 = new MyTask(2, "任务2");
        MyTask task3 = new MyTask(3, "任务3");
        MyTask task4 = new MyTask(4, "任务4");
        MyTask task5 = new MyTask(5, "任务5");
        MyTask task6 = new MyTask(6, "任务6");

        
        pool.execute(task1);
        pool.execute(task2);
        pool.execute(task3);
        pool.execute(task4);
        pool.execute(task5);
    }
}
```

```bash
当前线程Id-->5,任务名称-->任务5
当前线程Id-->1,任务名称-->任务1
当前线程Id-->2,任务名称-->任务2
当前线程Id-->3,任务名称-->任务3
当前线程Id-->4,任务名称-->任务4
```
原因在于,新建的有界队列只有3,除去当前执行的进程,线程池还能存放3个进程,因此当线程5放入的时候,队列满,则会新建线程去执行,优先级提高

## 参考

> 多线程线程池和自定义线程池 - 简书
> https://www.jianshu.com/p/0d548f0646fc

> (1条消息)Java线程池ThreadPoolExecutor及自定义线程池_Java_零度的博客专栏-CSDN博客
> https://blog.csdn.net/zmx729618/article/details/78839284