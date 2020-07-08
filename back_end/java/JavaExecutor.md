# Java 進階：ExecutorService 線程池

@[TOC](文章目錄)

<!-- TOC -->

- [Java 進階：ExecutorService 線程池](#java-進階executorservice-線程池)
  - [簡介](#簡介)
  - [參考](#參考)
- [正文](#正文)
  - [Thread 野線程](#thread-野線程)
    - [繼承 Thread](#繼承-thread)
    - [實現 Runnable](#實現-runnable)
    - [Lambda 表達式實現 Runnable](#lambda-表達式實現-runnable)
  - [ExecutorService 接口](#executorservice-接口)
    - [ThreadPoolExecutor 實現類](#threadpoolexecutor-實現類)
    - [Executors 工廠方法](#executors-工廠方法)
      - [語法](#語法)
      - [newFixedThreadPool](#newfixedthreadpool)
      - [newCachedThreadPool](#newcachedthreadpool)
      - [newSingleThreadExecutor](#newsinglethreadexecutor)
      - [newScheduledThreadPool](#newscheduledthreadpool)
    - [Methods 方法使用](#methods-方法使用)
- [結語](#結語)

<!-- /TOC -->

## 簡介

Java 語言最有用的功能之一就是`併發(concurrent)`，也就是常聽到的`多線程(Multi-Thread)`。在 Java 中實現線程的基礎就是 `Thread` 類，然而獨立創建的 Thread 類並不討喜，他沒有納入管控，也沒有限制數量，非常容易產生線程爆炸、資源競爭。因此我們希望透過一個`線程池(ThreadPool)`，來統一管理有限數量的線程和任務分配等問題，接下來就要介紹 `ExecutorService` 接口的使用。

## 參考

<table>
  <tr>
    <td>一文秒懂 Java ExecutorService</td>
    <td><a href="https://www.twle.cn/c/yufei/javatm/javatm-basic-executorservice.html">https://www.twle.cn/c/yufei/javatm/javatm-basic-executorservice.html</a></td>
  </tr>
  <tr>
    <td>ExecutorService 的理解与使用</td>
    <td><a href=https://blog.csdn.net/bairrfhoinn/article/details/16848785">https://blog.csdn.net/bairrfhoinn/article/details/16848785</a></td>
  </tr>
  <tr>
    <td>Java多執行緒框架Executor詳解</td>
    <td><a href="https://codertw.com/%E4%BC%BA%E6%9C%8D%E5%99%A8/161869/">https://codertw.com/%E4%BC%BA%E6%9C%8D%E5%99%A8/161869/</a></td>
  </tr>
  <tr>
    <td>Java多執行緒原理及Thread類的使用</td>
    <td><a href="https://www.itread01.com/content/1541919679.html">https://www.itread01.com/content/1541919679.html</a></td>
  </tr>
</table>

# 正文

詳細的 Java 多線程實現和運行原理，以及`鎖(lock)`存機制將留到另一篇獨立介紹，本篇主要專注於`線程池(ThreadPool)`的使用

## Thread 野線程

我們先來複習一下最原本的`線程(Thread)`的寫法：

1. 繼承 Thread 類，重寫 run 方法
2. 實現 Runnable 接口，實現 run 方法

### 繼承 Thread

首先我們先創建一個類繼承 Thread 類，然後`重寫(override)` `run()` 方法，這是線程運行的時候主要執行的方法

- ThreadExtension.java

```java
public class ThreadExtension extends Thread {

    private String name;

    public ThreadExtension(String name) {
        this.name = name;
    }

    @Override
    public void run() {
        for(int i=1 ; i<=10 ; i++) {
            System.out.println(Thread.currentThread().getName() + ":" + name + "-" + i);
        }
    }
}
```

接下來我們創建五個線程並且調用 `start()` 方法啟動線程（調用 run 僅僅只是一般的方法調用並不會啟動多線程）

- TestThread.java

```java
public class TestThread {
    public static void main(String[] args) throws Exception {

        for(int i=1 ; i<=5; i++) {
            new ThreadExtension("test" + i).start();
        }
    }
}
```

```
Thread-0:test1-1
Thread-0:test1-2
Thread-0:test1-3
Thread-0:test1-4
Thread-1:test2-1
Thread-1:test2-2
Thread-1:test2-3
Thread-1:test2-4
Thread-0:test1-5
Thread-0:test1-6
Thread-0:test1-7
Thread-0:test1-8
Thread-0:test1-9
Thread-0:test1-10
Thread-1:test2-5
Thread-1:test2-6
Thread-1:test2-7
Thread-1:test2-8
Thread-1:test2-9
Thread-1:test2-10
Thread-2:test3-1
Thread-2:test3-2
Thread-2:test3-3
Thread-2:test3-4
Thread-2:test3-5
Thread-2:test3-6
Thread-2:test3-7
Thread-2:test3-8
Thread-2:test3-9
Thread-2:test3-10
Thread-3:test4-1
Thread-3:test4-2
Thread-3:test4-3
Thread-3:test4-4
Thread-3:test4-5
Thread-3:test4-6
Thread-3:test4-7
Thread-3:test4-8
Thread-3:test4-9
Thread-3:test4-10
Thread-4:test5-1
Thread-4:test5-2
Thread-4:test5-3
Thread-4:test5-4
Thread-4:test5-5
Thread-4:test5-6
Thread-4:test5-7
Thread-4:test5-8
Thread-4:test5-9
Thread-4:test5-10
```

仔細看到我們創建了五個 Thread，就有 Thread 0-4，並且雖然 `start()` 方法的啟動有先後關係，不過實際上執行的順序卻是不固定的

### 實現 Runnable

第二種實現方式是實現 Runnable 接口（有一個僅有的），並且作為 Thread 的構造函數參數傳入

- RunnableImpl.java

```java
public class RunnableImpl implements Runnable {

    private String name;

    public RunnableImpl(String name) {
        this.name = name;
    }

    @Override
    public void run() {
        for(int i=1 ; i<=10 ; i++) {
            System.out.println(Thread.currentThread().getName() + ":" + name + "-" + i);
        }
    }
}
```

- TestThread.java

```java
public class TestThread {
    public static void main(String[] args) throws Exception {
        for(int i=1 ; i<=5 ; i++) {
            new Thread(new RunnableImpl("test"+i)).start();
        }
    }
}
```

```
Thread-0:test1-1
Thread-0:test1-2
Thread-1:test2-1
Thread-0:test1-3
Thread-1:test2-2
Thread-1:test2-3
Thread-0:test1-4
Thread-0:test1-5
Thread-0:test1-6
Thread-0:test1-7
Thread-0:test1-8
Thread-0:test1-9
Thread-0:test1-10
Thread-1:test2-4
Thread-1:test2-5
Thread-1:test2-6
Thread-1:test2-7
Thread-1:test2-8
Thread-1:test2-9
Thread-1:test2-10
Thread-2:test3-1
Thread-2:test3-2
Thread-2:test3-3
Thread-2:test3-4
Thread-2:test3-5
Thread-2:test3-6
Thread-2:test3-7
Thread-2:test3-8
Thread-2:test3-9
Thread-2:test3-10
Thread-3:test4-1
Thread-3:test4-2
Thread-3:test4-3
Thread-3:test4-4
Thread-3:test4-5
Thread-3:test4-6
Thread-3:test4-7
Thread-3:test4-8
Thread-3:test4-9
Thread-3:test4-10
Thread-4:test5-1
Thread-4:test5-2
Thread-4:test5-3
Thread-4:test5-4
Thread-4:test5-5
Thread-4:test5-6
Thread-4:test5-7
Thread-4:test5-8
Thread-4:test5-9
Thread-4:test5-10
```

多運行幾次就可以看到運行的差異，**再次強調，執行順序並不固定！**

### Lambda 表達式實現 Runnable

由於 `Runnable` 接口使用了 `@FunctionalInterface` 注解，是一個函數式的接口，我們就可以使用 Lambda 表達式來實現

```java
Runnable task = () -> {
    for(int i=0 ; i<5 ; i++) {
        System.out.println(Thread.currentThread().getName() + ":" + i);
    }
};
```

這邊就不展示運行結果了，與上面大同小異

## ExecutorService 接口

我們可以發現，使用 Thread 創建的線程是缺乏管理的，可以建立無限多個線程，互相競爭資源。因此我們更傾向於使用一個`線程池(ThreadPool)`統一管理所有線程，限制最大線程數量、進行任務分配排程、實現定時定期執行等作用，能夠高效的管理資源。

因此我們將使用 `ExecutorService` 接口來使用這些強大的功能。接下來我們將一一介紹
ExecutorService 的實現類，使用 Executors 的工廠方法創建 ExecutorService，以及接口的一些常用方法。

### ThreadPoolExecutor 實現類

`ExecutorService` 接口的主要實現類便是 `ThreadPoolExecutor`、`ScheduledThreadPoolExecutor`，其他還有如 `ForkJoinPool`、`FinalizableDelegatedExecutorService`、`DelegatedScheduledExecutorService` 等裝飾類（使用`裝飾器設計模式 Decorator Pattern`），我們可以在 `Executors` 的工廠方法實現中找到

這邊列出一些 `ThreadPoolExecutor` 的構造方法，我們可以從中看出一些可指定的參數

- ThreadPoolExecutor.java

```java
package java.util.concurrent;

public class ThreadPoolExecutor extends AbstractExecutorService {
    public ThreadPoolExecutor(int corePoolSize, /* 核心線程數 */
                              int maximumPoolSize, /* 最大線程數 */
                              long keepAliveTime, /* 線程保留時間 */
                              TimeUnit unit, /* 保留時間單位 */
                              BlockingQueue<Runnable> workQueue, /* 任務隊列 */
                              ThreadFactory threadFactory, /* 線程工廠 */
                              RejectedExecutionHandler handler) { /* 異常處理 */
        // ...
    }
    // ...
}
```

- ScheduledThreadPoolExecutor.java

```java
package java.util.concurrent;

public class ScheduledThreadPoolExecutor
        extends ThreadPoolExecutor
        implements ScheduledExecutorService {
    public ScheduledThreadPoolExecutor(int corePoolSize,
                                       ThreadFactory threadFactory,
                                       RejectedExecutionHandler handler) {
        super(corePoolSize, Integer.MAX_VALUE, 0, NANOSECONDS,
              new DelayedWorkQueue(), threadFactory, handler);
    }
}
```

### Executors 工廠方法

接下來我們介紹四種 `Executors` 類的工廠方法，用於創建 `ExecutorService` 對象：

#### 語法

| Method                  | Description                                                    |
| ----------------------- | -------------------------------------------------------------- |
| newFixedThreadPool      | 創建一個可複用、固定數量的線程池，用於`固定任務`               |
| newCachedThreadPool     | 可緩存線程池，會自動回收長時間閒置的線程，適合`短期非同步任務` |
| newSingleThreadExecutor | 單線程，確保所有任務依序執行，適合`前後順序固定的任務`         |
| newScheduledThreadPool  | 定時、週期性線程，適合設置`定時或週期性任務`                   |

```java
// newFixedThreadPool 創建固定數量的線程
public static ExecutorService newFixedThreadPool(int nThreads);
public static ExecutorService newFixedThreadPool(int nThreads, ThreadFactory threadFactory)

// newCachedThreadPool 緩存線程池
public static ExecutorService newCachedThreadPool();
public static ExecutorService newCachedThreadPool(ThreadFactory threadFactory);

// newSingleThreadExecutor 單線程
public static ExecutorService newSingleThreadExecutor();
public static ExecutorService newSingleThreadExecutor(ThreadFactory threadFactory);

// newScheduledThreadPool 定時、週期性任務
public static ScheduledExecutorService newScheduledThreadPool(int corePoolSize);
public static ScheduledExecutorService newScheduledThreadPool(int corePoolSize, ThreadFactory threadFactory);
```

我們先定義一個 `Task` 類實現 Runnable 用於後續範例：

- Task.java

```java
public class Task implements Runnable {
    private static int count = 0;
    private int id;

    public Task() {
        id = ++count;
    }

    public static void init() {
        count = 0;
    }

    @Override
    public void run() {
        System.out.println(Thread.currentThread().getName() + ":Task-" + id);
    }
}
```

#### newFixedThreadPool

- 方法定義

```java
public class Executors {
    public static ExecutorService newFixedThreadPool(int nThreads) {
        return new ThreadPoolExecutor(nThreads, nThreads,
                                      0L, TimeUnit.MILLISECONDS,
                                      new LinkedBlockingQueue<Runnable>());
    }
}
```

`newFixedThreadPool` 方法固定線程數量，所有任務將會隨機分配給不同線程執行

- TestThread.java

```java
public class TestThread {
    public static void main(String[] args) throws Exception {
        ExecutorService fixedThreadPool = Executors.newFixedThreadPool(5);
        for(int i=0 ; i<10 ; i++) {
            fixedThreadPool.execute(new Task());
        }
        fixedThreadPool.shutdown();
    }
}
```

```
pool-1-thread-1:Task-1
pool-1-thread-2:Task-2
pool-1-thread-3:Task-3
pool-1-thread-4:Task-4
pool-1-thread-5:Task-5
pool-1-thread-5:Task-6
pool-1-thread-1:Task-7
pool-1-thread-2:Task-8
pool-1-thread-1:Task-10
pool-1-thread-5:Task-9
```

我們可以看到線程池最多存在五個線程，我們一共創建十個任務將隨機分配給不同的線程執行（有關 `execute`、`shutdown` 於下面的[Methods-方法使用](#Methods-方法使用)講解）

#### newCachedThreadPool

- 方法定義

```java
public class Executors {
    public static ExecutorService newCachedThreadPool() {
        return new ThreadPoolExecutor(0, Integer.MAX_VALUE,
                                      60L, TimeUnit.SECONDS,
                                      new SynchronousQueue<Runnable>());
    }
}
```

`newCachedThreadPool` 的線程數量是不固定的，線程等待時間過長將會自動釋放資源，較為靈活

- TestThread.java

```java
public class TestThread {
    public static void main(String[] args) throws Exception {
        ExecutorService cachedThreadPool = Executors.newCachedThreadPool();
        for(int i=0 ; i<10; i++) {
            cachedThreadPool.execute(new Task());
        }
        cachedThreadPool.shutdown();
    }
}
```

```
pool-1-thread-1:Task-1
pool-1-thread-2:Task-2
pool-1-thread-2:Task-3
pool-1-thread-2:Task-4
pool-1-thread-1:Task-5
pool-1-thread-1:Task-6
pool-1-thread-1:Task-9
pool-1-thread-3:Task-8
pool-1-thread-2:Task-7
pool-1-thread-3:Task-10
```

線程池將會自動分配適當的線程數量，以及回收線程資源等功能，較適合短期任務，用完後很快釋放資源的任務

#### newSingleThreadExecutor

- 方法定義

```java
public class Executors {
    public static ExecutorService newSingleThreadExecutor() {
        return new FinalizableDelegatedExecutorService
            (new ThreadPoolExecutor(1, 1,
                                    0L, TimeUnit.MILLISECONDS,
                                    new LinkedBlockingQueue<Runnable>()));
    }
}
```

`newSingleThreadExecutor` 只會建立單獨而唯一的一個線程，所有任務都將透過同一個線程順序執行，就好像 JavaScript 的異步函數隊列一樣（排隊順序執行）

- TestThread.java

```java
public class TestThread {
    public static void main(String[] args) throws Exception {
        ExecutorService singleThreadExecutor = Executors.newSingleThreadExecutor();
        for(int i=0 ; i<10; i++) {
            singleThreadExecutor.execute(new Task());
        }
        singleThreadExecutor.shutdown();
    }
}
```

```
pool-1-thread-1:Task-1
pool-1-thread-1:Task-2
pool-1-thread-1:Task-3
pool-1-thread-1:Task-4
pool-1-thread-1:Task-5
pool-1-thread-1:Task-6
pool-1-thread-1:Task-7
pool-1-thread-1:Task-8
pool-1-thread-1:Task-9
pool-1-thread-1:Task-10
```

**注意！**只有在單線程之下才能確保多個任務是順序執行的，前面的 `newFixedThreadPool`、`newCachedThreadPool` 當指定線程數大於 1 時，就不能保證任務執行的先後順序

#### newScheduledThreadPool

- 方法定義

```java
public class Executors {
    public static ScheduledExecutorService newScheduledThreadPool(int corePoolSize) {
        return new ScheduledThreadPoolExecutor(corePoolSize);
    }
}
```

`newScheduledThreadPool` 創建一個可以定時、定期的線程任務池，與其他工廠方法不同的是它返回一個更為具體的 `ScheduledExecutorService` 類。與其他 `ExecutorService` 使用 `execute` 接受任務不同，`ScheduledExecutorService` 使用另外三個方法來執行定時、定期任務

```java
// 一次性任務，給定延遲後執行
public ScheduledFuture<?> schedule(Runnable command, /* 待執行任務 */
                                    long delay, /* 延遲 */
                                    TimeUnit unit); /* 時間單位 */

// 定期任務，第一次延遲(initialDelay)後執行，之後每次在固定時間(period)執行
public ScheduledFuture<?> scheduleAtFixedRate(Runnable command, /* 待執行任務 */
                                              long initialDelay, /* 第一次延遲 */
                                              long period, /* 執行週期 */
                                              TimeUnit unit); /* 時間單位 */

// 固定間隔任務，第一次延遲(initialDelay)後執行，下次執行與上次執行距離固定間隔(即前一個方法的執行週期 period = 任務執行時間 + delay)
public ScheduledFuture<?> scheduleWithFixedDelay(Runnable command, /* 待執行任務 */
                                                  long initialDelay, /* 第一次延遲 */
                                                  long delay, /* 兩次執行間隔時間 */
                                                  TimeUnit unit); /* 時間單位 */
```

- TestThread.java

```java
public class TestThread {
    public static void main(String[] args) throws Exception {
        ScheduledExecutorService scheduledExecutorService = Executors.newScheduledThreadPool(3);
        for(int i=0 ; i<10; i++) {
            // 一次性任務
            scheduledExecutorService.schedule(new Task(), 1000 - i * 100, TimeUnit.MILLISECONDS);
        }
        // 定期任務
        scheduledExecutorService.scheduleAtFixedRate(new Task(), 1000, 1000, TimeUnit.MILLISECONDS);

        // 固定間隔任務
        scheduledExecutorService.scheduleWithFixedDelay(new Task(), 1200, 1000, TimeUnit.MILLISECONDS);
    }
}
```

```
pool-1-thread-2:Task-10
pool-1-thread-3:Task-9
pool-1-thread-1:Task-8
pool-1-thread-2:Task-7
pool-1-thread-3:Task-6
pool-1-thread-1:Task-5
pool-1-thread-2:Task-4
pool-1-thread-3:Task-3
pool-1-thread-1:Task-2
pool-1-thread-2:Task-1
pool-1-thread-3:Task-11
pool-1-thread-1:Task-12
...
```

- Task 1~10 會在固定延遲後執行
- Task 11 會在每 1 秒執行一次
- Task 12 每兩次執行中間間隔 1 秒

搞清楚 `scheduleAtFixedRate` 和 `scheduleWithFixedDelay` 的差異非常重要

### Methods 方法使用

最後我們來介紹一下 `ExecutorService` 接口的所有方法。在此之前我們需要先知道一個`線程池(ThreadPool)`，也就是一個 `ExecutorService` 的實例在整個生命週期有三個階段：

1. 執行 execute：當我們創建實例之後就進入 execute 階段，這時候我們能夠`提交並執行任務`
2. 關閉 shutdown：調用 `shutdown`、`shutdownNow` 後進入 shutdown 階段，這時候線程池`停止提交新任務`，並且會`等待直到所有任務執行完畢`後才進入 terminated 階段。
3. 終止 terminated：這時候線程池已經停止提交新任務，所以現有任務也都執行完畢了，將會`完全結束線程池並釋放資源`

接下來我們來看看 `ExecutorService` 的方法列表

| Methods          | Description                                            |
| ---------------- | ------------------------------------------------------ |
| execute          | 提交並執行任務                                         |
| submit           | 提交並執行任務，額外返回一個 Future 對象               |
| invokeAny        | 提交並執行多個任務，返回最快成功的一個任務             |
| invokeAll        | 提交並執行多個任務，返回所有任務結果                   |
| shutdown         | 使線程池進入關閉階段(shutdown)                         |
| shutdownNow      | 強制使線程池進入關閉階段(shutdown)，返回未完成任務隊列 |
| awaitTermination | 關閉(shutdown)後，返回所有任務在限定時間內完成並終止   |
| isShutdown       | 檢查是否處於關閉階段                                   |
| isTerminated     | 檢查是否處於終止階段                                   |

```java
// 提交並執行任務
void execute(Runnable command);
Future<?> submit(Runnable task);
<T> Future<T> submit(Runnable task, T result);
<T> Future<T> submit(Callable<T> task);

// 提交多任務隊列
<T> T invokeAny(Collection<? extends Callable<T>> tasks) throws InterruptedException, ExecutionException;
<T> T invokeAny(Collection<? extends Callable<T>> tasks, long timeout, TimeUnit unit) throws InterruptedException, ExecutionException, TimeoutException;
<T> List<Future<T>> invokeAll(Collection<? extends Callable<T>> tasks) throws InterruptedException;
<T> List<Future<T>> invokeAll(Collection<? extends Callable<T>> tasks, long timeout, TimeUnit unit) throws InterruptedException;

// 關閉 ExecutorService
void shutdown();
List<Runnable> shutdownNow();

// 檢查狀態
boolean awaitTermination(long timeout, TimeUnit unit) throws InterruptedException;
boolean isShutdown();
boolean isTerminated();
```

- TestThread.java

```java
public class TestThread {
    public static void main(String[] args) throws Exception {
        ExecutorService fixedThreadPool = Executors.newFixedThreadPool(3);
        Runnable task = new Runnable() {
            private String name = "task-test";
            private int count = 0;
            @Override
            public void run() {
                System.out.println(Thread.currentThread().getName() + ":" + name + ":" + count++);
            }
        };
        List<Callable<String>> tasks = new ArrayList<>();
        for(int i=0 ; i<5 ; i++) {
            tasks.add(new CallableTask());
        }

    //    execute
        fixedThreadPool.execute(task);
        // pool-1-thread-1:task-test:0

    //    submit
        Future<String> future = fixedThreadPool.submit(task, "submit over");
        System.out.println(future.get());
        // pool-1-thread-2:task-test:1
        // submit over

    //    invokeAny
        String result = fixedThreadPool.invokeAny(tasks);
        System.out.println(result);
        // pool-1-thread-3:callable-1
        // callable-1 over

    //    invokeAll
        List<Future<String>> results = fixedThreadPool.invokeAll(tasks);
        for(Future<String> res : results) {
            System.out.println(res.get());
        }
        // pool-1-thread-1:callable-1
        // pool-1-thread-1:callable-2
        // pool-1-thread-3:callable-3
        // pool-1-thread-1:callable-4
        // pool-1-thread-1:callable-5
        // callable-1 over
        // callable-2 over
        // callable-3 over
        // callable-4 over
        // callable-5 over

    //    shutdown
        fixedThreadPool.shutdown();
    //    awaitTermination
        if(!fixedThreadPool.awaitTermination(1000, TimeUnit.MILLISECONDS)) {
            System.out.println("shutdownNow");
    //    shutdownNow
            List<Runnable> incompleteTasks = fixedThreadPool.shutdownNow();
        } else {
            System.out.println("shutdownNormal");
            // shutdownNormal
        }
    }
}
```

這邊還演示了一個 `shutdown`、`awaitTermination`、`shutdownNow` 連用用於關閉 `ExecutorService` 的方法，供參考。

**注意！**`ExecutorService` 並不會自己結束，至少記得要 shutdown 哦！

# 結語

本篇介紹了有關`線程池(ThreadPool)`的`類(ThreadPoolExecutor)`和`接口(ExecutorService)`以及`各種操作(execute、submit、invoke、shutdown)`，有關多線程返回結果的 `Future` 對象將放到下一篇介紹。
