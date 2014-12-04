---
layout: post
title: 关于 ExecutorService 使用技巧和注意事项
category : java
tags : [Java, Concurrent, ExecutorService]
---
{% include JB/setup %}

## 1、为池中的线程命名  ##

当我们为运行中的 JVM 做线程转储或者在程序调试时，池中线程默认的命名格式是 `pool-N-thread-M`，`N` 代表线程池的序号（每次新建一个线程池，`N` 就会加1），`M` 是该线程池里的线程编号。例如 `pool-2-thread-3` 代表虚拟机创建的第二个线程池中的第三个线程。参见[Executors.defaultThreadFactory()](https://docs.oracle.com/javase/8/docs/api/java/util/concurrent/Executors.html#defaultThreadFactory "Executors.defaultThreadFactory()")。所以想为线程重新命名还是有点小麻烦的，因为 JDK 把它隐藏在了 [ThreadFactory](https://docs.oracle.com/javase/8/docs/api/java/util/concurrent/ThreadFactory.html "ThreadFactory") 中。为此 Guava 提供了一个帮助方法：

{% highlight java linenos %}
import com.google.common.util.concurrent.ThreadFactoryBuilder;
 
final ThreadFactory threadFactory = new ThreadFactoryBuilder()
    .setNameFormat("Orders-%d")
    .build();
final ExecutorService executorService = Executors.newFixedThreadPool(10, threadFactory);
{% endhighlight %}

## 2、根据上下文切换线程名字 ##

这在某些情况下是非常有用的，例如在 DUMP 线程栈的时候是只有类名、方法名等信息，但没有参数值和本地变量值等。但通过修改线程名我们就可以维持一些额外信息了，这样可以便于我们跟踪一些运行缓慢的任务等。

{% highlight java linenos %}
private void process(String messageId) {
    executorService.submit(() -> {
        final Thread currentThread = Thread.currentThread();
        final String oldName = currentThread.getName();
        currentThread.setName("Processing-" + messageId);
        try {
            //real logic here...
        } finally {
            currentThread.setName(oldName);
        }
    });
}
{% endhighlight %}

## 3、安全关闭 ##

在线程池中一般会有大量未执行的任务。当应用关闭的时候，我们就需要注意了：队列中的任务和运行中的任务该如何处理呢。这地方有两种方式：等待所有的任务运行完成（`shutdown()`）或者直接丢弃（`shutdownNow()`），取决于你的使用场景。

例如：我们提交了大量的任务并希望在它们执行完成后尽快返回，那么调用 `shutdown()`:

{% highlight java linenos %}
private void sendAllEmails(List<String> emails) throws InterruptedException {
    emails.forEach(email ->
            executorService.submit(() ->
                    sendEmail(email)));
    executorService.shutdown();
    final boolean done = executorService.awaitTermination(1, TimeUnit.MINUTES);
    log.debug("All e-mails were sent so far? {}", done);
}
{% endhighlight %}

在这个场景中，我们发送了一批电子邮件，每一封是一个任务。提交完任务后我们关闭了线程池，然后它就可以不再接受新任务了。然后我们等待任务完成，最多等一分钟。如果仍然有未完成的任务，`awaitTermination()` 也会返回 `false`。但这不影响任务执行。

使用 `shutdownNow()` 时：

{% highlight java linenos %}
final List<Runnable> rejected = executorService.shutdownNow();
log.debug("Rejected tasks: {}", rejected.size());
{% endhighlight %}

这样所有队列中的任务都会被取消并返回，执行中的任务会继续。

## 4、控制任务队列长度 ##

不合适的线程池大小会导致系统变慢，稳定性降低，内存溢出等问题。线程数过小，队列会快速增大，进而消耗大量内存。线程过多又会因为过多的线程上下文切换拖慢系统速度。所以维持任务队列的稳定也是相当重要的，对于过多的任务只需要拒绝就可以了：

{% highlight java linenos %}
final BlockingQueue<Runnable> queue = new ArrayBlockingQueue<>(100);
executorService = new ThreadPoolExecutor(n, n, 0L, TimeUnit.MILLISECONDS, queue);
{% endhighlight %}

这段代码相当于 `Executors.newFixedThreadPool(n)`，只是用固定大小的 `ArrayBlockingQueue` 代替了不限定大小的 `LinkedBlockingQueue`。在这种情况下，如果队列大小达到100后，提交新任务会抛出 `RejectedExecutionException`。而且现在由于队列是自己申请的，于是我们也可以调用它的 `size()` 方法获取队列大小记日志做监控什么的。

## 5、记得处理异常 ##

先看看下面这段代码：

{% highlight java linenos %}
executorService.submit(() -> {
    System.out.println(1 / 0);
});
{% endhighlight %}

我已经被这种问题坑过太多次了：它不会打印任何东西，也不会提示 `java.lang.ArithmeticException: / by zero`，什么都没有。如果它是你自己申请的一个 `java.lang.Thread` 实例，那么 `UncaughtExceptionHandler` 会被调用。但在线程池中你必须要小心。如果提交的是 `Runnable`，那么必须要用 `try-catch` 块包起来然后做好日志。如果提交的是 `Callable<Integer>`，那么需要调用 `get()` 方法获取到异常。

## 6、监测队列等待时间 ##

监测队列大小很重要。但在有些时候获取任务从提交到执行间隔的时间也是很重要的。当线程池中存在空闲线程时这个时间应该接近0，当任务增多时可能增长。如果线程池不是固定大小的，提交新任务也可能需要申请新线程，这也会花费一点时间。为了监测这个时间，我们可以这么做：

{% highlight java linenos %}
public class WaitTimeMonitoringExecutorService implements ExecutorService {
 
    private final ExecutorService target;
 
    public WaitTimeMonitoringExecutorService(ExecutorService target) {
        this.target = target;
    }
 
    @Override
    public <T> Future<T> submit(Callable<T> task) {
        final long startTime = System.currentTimeMillis();
        return target.submit(() -> {
            final long queueDuration = System.currentTimeMillis() - startTime;
            log.debug("Task {} spent {}ms in queue", task, queueDuration);
            return task.call();
            }
        );
    }
 
    @Override
    public <T> Future<T> submit(Runnable task, T result) {
        return submit(() -> {
            task.run();
            return result;
        });
    }
 
    @Override
    public Future<?> submit(Runnable task) {
        return submit(new Callable<Void>() {
            @Override
            public Void call() throws Exception {
                task.run();
                return null;
            }
        });
    }
 
    //...
 
}
{% endhighlight %}

当然这个实现不完整，只是一个思路。当任务提交到线程池时，我们开始计时，任务被取走并开始执行时计时结束。

## 7、维持调用栈 ##

响应式编程越来越多，当然这很好也很强大。但异常栈就不那么友好了，它们几乎毫无作用。举个例子，某个异常信息可能如下：

{% highlight java linenos %}
java.lang.NullPointerException: null
    at com.nurkiewicz.MyTask.call(Main.java:76) ~[classes/:na]
    at com.nurkiewicz.MyTask.call(Main.java:72) ~[classes/:na]
    at java.util.concurrent.FutureTask.run(FutureTask.java:266) ~[na:1.8.0]
    at java.util.concurrent.ThreadPoolExecutor.runWorker(ThreadPoolExecutor.java:1142) ~[na:1.8.0]
    at java.util.concurrent.ThreadPoolExecutor$Worker.run(ThreadPoolExecutor.java:617) ~[na:1.8.0]
    at java.lang.Thread.run(Thread.java:744) ~[na:1.8.0]
{% endhighlight %}

很明显是 `MyTask` 的 76 行抛出了这个异常。但我们看不到是谁提交了这个任务，因为异常栈中只显示到了 `Thread` 和 `ThreadPoolExecutor`。在这种情况下我们应该怎么做让之前的调用栈也打印出来呢？

我们可以这么做：
{% highlight java linenos %}
public class ExecutorServiceWithClientTrace implements ExecutorService {
 
    protected final ExecutorService target;
 
    public ExecutorServiceWithClientTrace(ExecutorService target) {
        this.target = target;
    }
 
    @Override
    public <T> Future<T> submit(Callable<T> task) {
        return target.submit(wrap(task, clientTrace(), Thread.currentThread().getName()));
    }
 
    private <T> Callable<T> wrap(final Callable<T> task, final Exception clientStack, String clientThreadName) {
        return () -> {
            try {
                return task.call();
            } catch (Exception e) {
                log.error("Exception {} in task submitted from thrad {} here:", e, clientThreadName, clientStack);
                throw e;
            }
        };
    }
 
    private Exception clientTrace() {
        return new Exception("Client stack trace");
    }
 
    @Override
    public <T> List<Future<T>> invokeAll(Collection<? extends Callable<T>> tasks) throws InterruptedException {
        return tasks.stream().map(this::submit).collect(toList());
    }
 
    //...
 
}
{% endhighlight %}

然后打印出来的栈信息就是这样了：

{% highlight java linenos %}
Exception java.lang.NullPointerException in task submitted from thrad main here:
java.lang.Exception: Client stack trace
    at com.nurkiewicz.ExecutorServiceWithClientTrace.clientTrace(ExecutorServiceWithClientTrace.java:43) ~[classes/:na]
    at com.nurkiewicz.ExecutorServiceWithClientTrace.submit(ExecutorServiceWithClientTrace.java:28) ~[classes/:na]
    at com.nurkiewicz.Main.main(Main.java:31) ~[classes/:na]
    at sun.reflect.NativeMethodAccessorImpl.invoke0(Native Method) ~[na:1.8.0]
    at sun.reflect.NativeMethodAccessorImpl.invoke(NativeMethodAccessorImpl.java:62) ~[na:1.8.0]
    at sun.reflect.DelegatingMethodAccessorImpl.invoke(DelegatingMethodAccessorImpl.java:43) ~[na:1.8.0]
    at java.lang.reflect.Method.invoke(Method.java:483) ~[na:1.8.0]
    at com.intellij.rt.execution.application.AppMain.main(AppMain.java:134) ~[idea_rt.jar:na]
{% endhighlight %}

## 8、尽量使用 CompletableFuture ##

在 Java8 中引入了一个新类 [`CompletableFuture`](http://www.javacodegeeks.com/2013/05/java-8-definitive-guide-to-completablefuture.html  "CompletableFuture")。这个类应该多用。调用方式如下：

{% highlight java linenos %}
final CompletableFuture<BigDecimal> future = 
    CompletableFuture.supplyAsync(this::calculate, executorService);
{% endhighlight %}

`CompletableFuture` 继承自 `Future` 所以一切运行正常。同时我们还能使用很多由 `CompletableFuture` 提供的 API。

## 9、同步队列 ##

[`SynchronousQueue`](https://docs.oracle.com/javase/8/docs/api/java/util/concurrent/SynchronousQueue.html "SynchronousQueue") 是一个很有趣的 `BlockingQueue`。它最好的解释是一个大小为0的队列。引用 JavaDoc:

    一种阻塞队列，其中每个插入操作必须等待另一个线程的对应移除操作 ，反之亦然。同步队列没有任何内部容量，甚至连一个队列的容量都没有。不能在同步队列上进行 peek，因为仅在试图要移除元素时，该元素才存在；除非另一个线程试图移除某个元素，否则也不能（使用任何方法）插入元素；也不能迭代队列，因为其中没有元素可用于迭代。

这和线程池有什么关系呢？我们试着用它创建一个 `ThreadPoolExecutor`：

{% highlight java linenos %}
BlockingQueue<Runnable> queue = new SynchronousQueue<>();
ExecutorService executorService = new ThreadPoolExecutor(2, 2, 0L, TimeUnit.MILLISECONDS, queue);
{% endhighlight %}

我们建了一个只有两个线程的线程池。因为 `SynchronousQueue` 的队列大小是 0。所以 `ExecutorService` 只会在有空闲线程时才接受新任务。如果没有空闲进程，那么它会直接拒绝。这可能在某些需要立即后台执行或者抛弃的场景下很有用。

全文完，希望它对你有用。

转载注明出处：[{{page.title}}]({{permalink}})

[原文链接](http://www.nurkiewicz.com/2014/11/executorservice-10-tips-and-tricks.html "http://www.nurkiewicz.com/2014/11/executorservice-10-tips-and-tricks.html")