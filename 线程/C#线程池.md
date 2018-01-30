# 线程池
在多线程的程序中，经常会出现两种情况：
一种情况：   应用程序中，线程把大部分的时间花费在等待状态，等待某个事件发生，然后才能给予响应 
                  这一般使用ThreadPool（线程池）来解决；
另一种情况：线程平时都处于休眠状态，只是周期性地被唤醒 
                  这一般使用Timer（定时器）来解决；
将任务添加进线程池:
ThreadPool.QueueUserWorkItem(new WaitCallback(方法名));
重载
ThreadPool.QueueUserWorkItem(new WaitCallback(方法名), 参数);
因为ThreadPool是静态类 所以不需要实例化.

ThreadPool.SetMaxThreads 方法
public static bool SetMaxThreads(
	int workerThreads,
	int completionPortThreads
)
参数:
workerThreads
类型：System.Int32 
线程池中辅助线程的最大数目。
completionPortThreads
类型：System.Int32 
线程池中异步 I/O 线程的最大数目。

1. 线程池的启动和终止不是我们程序所能控制的
2. 线程池中的线程执行完之后是没有返回值的.
总之一句话, 我们不知道线程池他干了什么, 那么我们该怎么解决 任务完成问题呢?
操作系统提供了一种”信号灯”(ManualResetEvent)
ManualResetEvent 允许线程通过发信号互相通信。通常，此通信涉及一个线程在其他线程进行之前必须完成的任务。当一个线程开始一个活动（此活动必须完成后，其他线程才能开始）时，它调用 Reset 以将 ManualResetEvent 置于非终止状态，此线程可被视为控制 ManualResetEvent。调用 ManualResetEvent 上的 WaitOne 的线程将阻止，并等待信号。当控制线程完成活动时，它调用 Set 以发出等待线程可以继续进行的信号。并释放所有等待线程。一旦它被终止，ManualResetEvent 将保持终止状态（即对 WaitOne 的调用的线程将立即返回，并不阻塞），直到它被手动重置。可以通过将布尔值传递给构造函数来控制 ManualResetEvent 的初始状态，如果初始状态处于终止状态，为 true；否则为 false。
详细见MSDN: http://msdn.microsoft.com/zh-cn/library/system.threading.manualresetevent.aspx
主要使用了
eventX.WaitOne(Timeout.Infinite, true);  阻止当前线程，直到当前 WaitHandle 收到信号为止。
eventX.Set(); 将事件状态设置为终止状态，允许一个或多个等待线程继续。

线程池
1.概念
线程池可以看做容纳线程的容器；一个应用程序最多只能有一个线程池；
ThreadPool静态类通过QueueUserWorkItem()方法将工作函数排入线程池；每排入一个工作函数，就相当于请求创建一个线程
2.作用
线程池是为突然大量爆发的线程设计的，通过有限的几个固定线程为大量的操作服务，减少了创建和销毁线程所需的时间，从而提高效率。
如果一个线程的时间非常长，就没必要用线程池了(不是不能作长时间操作，而是不宜。)，况且我们还不能控制线程池中线程的开始、挂起、和中止。
3.什么时候使用线程池
ThreadPool中的线程不用手动开始，也不能手动取消，你要做的只是把工作函数排入线程池，剩下的工作由系统自动完成，也就是说我们不能控制线程池中的线程。
如果想对线程进行更多的控制，那么就不适合使用线程池。

在以下情况下不宜使用ThreadPool类而应该使用单独的Thread类：
1/ 线程执行需要很长时间；
2/ 需要为线程指定详细的优先级；
3/ 在执行过程中需要对线程进行操作，比如睡眠，挂起等。
所以ThreadPool适合于并发运行若干个运行时间不长且互不干扰的函数。

![Alt text](https://github.com/liangwei0101/StudyNotes/blob/master/images/线程池.png)


