# c# Task
当方法运行时遇到某个await关键字，编译器会开辟一个独立的线程，将其后的运行代码会放入该线程中运行。

简介
.NET 4包含新名称空间System.Threading.Tasks，它包含的类抽象出了线程功能。在后台使用ThreadPool。任务表示应完成的某个单元的工作。这个单元的工作可以在单独的线程中运行,也可以以同步方式启动一个任务,这需要等待主调线程。使用任务不仅可以获得一个抽象层,还可以对底层线程进行很多控制。
在安排需要完成的工作时,任务提供了非常大的灵活性。例如，可以定义连续的工作——在一个任务完成后该执行什么工作。这可以区分任务成功与否。另外，还可以在层次结构中安排任务。例如，父任务可以创建新的子任务。这可以创建一种依赖关系，这样，取消父任务，也会取消其子任务。

启动任务
要启动任务,可以使用TaskFactory类或Task类的构造函数和Start()方法。Task类的构造函数在创建任务上提供的灵活性较大。
在启动任务时，会创建Task类的一个实例,利用Action或Action <object>委托不带参数或带一个object参数 ,可以指定应运行的代码，这类似于Thread类。下面定义了一个无参数的方法。在实现代码中,把任务的ID写入控制台中:
static void TaskMethod()
{
    Console.WriteLine("running in a task");
    Console.WriteLine("Task id: {0}",Task.CurrentId);
}

第一种方式使用实例化TaskFactory类,在其中把TaskMedlod()方法传递给StartNew()方法 ,就会立即启动任务。第二种方式使用Task类的构造函数。实例化Task对象时,任务不会立即运行,而是指定Created状态。接着调用Task类的Start()方法,来启动任务。使用Task类时,除了调用Start()方法,还可以调用RunSynchronously()方法。这样,任务也会启动,但在调用者的当前线程中它正在运行,调用者需要一直等待到该任务结束。默认情况下,任务是异步运行的。

//using task factory
    TaskFactory tf = new TaskFactory();
    Task t1 = tf.StartNew(TaskMethod);
//using the task factory via a task
    Task t2 = Task.TaskFactory.StartNew(TaskMethod);
//using Task constructor
    Task t3 = new Task(TaskMethod);
    t3.Start();

使用Task类的构造函数和TaskFactory类的StartNew()方法时,都可以传递TaskCreationOptions枚举中的值。设置LongRunning选项,可以通知任务调度器,该任务需要较长时间执行,这样调度器更可能使用新线程。如果该任务应关联到父任务上,而父任务取消了,则该任务也应取消,此时应设置AuachToParent选项。PreferFairness的值表示,调度器应提取出已在等待的第一个任务。如果一个任务在另一个任务内部创建,这就不是默认情况。如果任务使用子任务创建了其他工作,子任务就优先于其他任务。它们不会排在线程池队列中的最后。如果这些任务应以公平的方式与所有其他任务一起处理,就设置该选项为PreferFairness。

Task t4 = new Task(TaskMethod, TaskCreationOptions.PreferFairness);
t4.Start();

连续任务
 通过任务,可以指定在任务完成后,应开始运行另一个特定任务 ,例如,一个使用前一个任务的结果的新任务,如果前一个任务失败了,这个任务就应执行一些清理工作。
任务处理程序或者不带参数或者带一个对象参数,而连续处理程序有一个Task类型的参数,这里可以访问起始任务的相关信息:

static void DoOnFirst()
{
    Console.WriteLine("doing some task {0}",Task.CurrentId);
    Thread.Sleep(3000);
}
static void DoOnSecond(Task t)
{
    Console.WriteLine("task {0} finished", t.Id);
    Console.WriteLine("this task id {0}", Task.CurrentId);
    Console.WriteLine("do some cleanup");
    Thread.Sleep(3000);
}
 连续任务通过在任务上调用ContinueWith()方法来定义。也可以使用TaskFactory类来定义。t1.ContinueWith(DoOnSecond)方法表示,调用DoOnSecond()方法的新任务应在任务t1结束时立即启动。在一个任务结束时,可以启动多个任务,连续任务也可以有另一个连续任务,如下面的例子所示:
Task t1 = new Task(DoOnFirst);
Task t2 = t1.ContinueWith(DoOnSecond);
Task t3 = t1.ContinueWith(DoOnSecond);
Task t4 = t2.ContinueWith(DoOnSecond);

任务层次结构
利用任务连续性,可以在一个任务结束后启动另一个任务。任务也可以构成一个层次结构。一个任务启动一个新任务时,就启动了一个父/子层次结构。
下面的代码段在父任务内部新建一个任务。创建子任务的代码与创建父任务的代码相同,唯一的区别是这个任务从另一个任务内部创建。

static void ParentAndChild()
{
    var parent = new Task(ParentTask);
    parent.Start();
    Thread.Sleep(2000);
    Console.WriteLine(parent.Status);
    Thread.Sleep(4000);
    Console.WriteLine(parent.Status);
}
static void ParentTask()
{
    Console.WriteLine("task id {0}", Task.CurrentId);
    var child = new Task(ChildTask);
    child.Start();
    Thread.Sleep(1000);
    Console.WriteLine("parent started child");
}
static void ChildTask()
{
    Console.WriteLine("child");
    Thread.Sleep(5000);
    Console.WriteLine("child finished");
}

如果父任务在子任务之前结束,父任务的状态就显示为WaitingForChildrenToComplete。只要子任务也结束时,父任务的状态就变成RanToCompletion。当然,如果父任务用TaskCreationOptions枚举中的DetachedFromParent创建子任务时,这就无效。

任务的结果
任务结束时,它可以把一些有用的状态信息写到共享对象中。这个共享对象必须是线程安全的。另一个选项是使用返回某个结果的任务。使用Task类的泛型版本,就可以定义返回某个结果的任务的返回类型。
为了返回某个结果任务调用的方法可以声明为带任意返回类型。示例方法TaskWithResult()利用一个元组返回两个int值。该方法的输入可以是void或object类型,如下所示:

来自 <http://www.itread01.com/content/1495044023.html> 


用Task类的话，相对就比较简单了，至少程式码看起来很舒服。也就意味着维护也比较方便
    Task t1 = Task.Factory.StartNew( delegate { MyMethodA(); }); 
    Task t2 = Task.Factory.StartNew( delegate { MyMethodB(); }); 
    t1.Wait(); 
    t2.Wait();

上面的方法是一个一个的执行完毕，获取不是我们想要的，我们一般是想要他们一起同时执行，提高程序处理事情的效率。
    Task t1 = Task.Factory.StartNew( delegate { MyMethodA(); }); 
    Task t2 = Task.Factory.StartNew( delegate { MyMethodB(); }); 
    Task.WaitAll(t1, t2);

来自 <https://tw.saowen.com/a/813d031661416f02bed4c3c485882f7bf9df1fe3c855a558269d5c7abded073f> 

取消Task
我们一开始就描述了CancellationTokenSource这个物件对Task的取消执行。一般是用不到这个方法的，一般会正常的退出所执行的程式码，如使用bool IsExit之类的来进行一个控制。而不是中途强制中断程式码。

来自 <https://tw.saowen.com/a/813d031661416f02bed4c3c485882f7bf9df1fe3c855a558269d5c7abded073f> 


 static void Main(string[] args)
        {
            Task parent = Task.Factory.StartNew(() =>
            {
                Console.WriteLine("I am a parent");
                Task.Factory.StartNew(() =>        // Detached task
                {
                    Thread.Sleep(2000);
                    Console.WriteLine("I am detached");
                });
                Task.Factory.StartNew(() =>        // Child task
                {
                    Thread.Sleep(3000);
                    Console.WriteLine("I am a child");
                }, TaskCreationOptions.AttachedToParent);
            });
            parent.Wait();
            Console.ReadLine();
        }

如果你等待你一个任务结束，你必须同时等待任务里面的子任务结束。这一点很重要，尤其是你在使用继续的时候。（后面会介绍），
等待任务
在线程池内建的方法中无法实现的等待，在任务中可以很简单的实现了

  static void Main()
        {
            var t1 = Task.Run(() => Go(null));
            var t2 = Task.Run(() => Go(123));
            Task.WaitAll(t1, t2);//等待所有Task結束
            //Task.WaitAny(t1, t2);//等待任意Task結束
        }
        static void Go(object data)   // data will be null with the first call.
        {
            Thread.Sleep(5000);
            Console.WriteLine("Hello from the thread pool! " + data);
        }

注意：
当你呼叫一个Wait方法时，当前的执行绪会被阻塞，直到任务返回。但是如果Task还没有被执行，这个时候系统可能会用当前的执行绪来执行呼叫Task，而不是新建一个，这样就不需要重新建立一个执行绪，并且阻塞当前执行绪。这种做法节省了建立新执行绪的开销，也避免了一些执行绪的切换。但是也有缺点，当前执行绪如果和被呼叫的Task同时想要获得一个lock，就会导致死锁。Task 
异常处理
当等待一个任务完成的时候（呼叫等待或者或者访问结果属性的时候），任务任务中没有处理的异常会被封装成AggregateException重新丢掷，InnerExceptions属性封装了各个任务没有处理的异常。

来自 <https://tw.saowen.com/a/37def47d4a5399fc86ec7596f1ffd02ac14889ab53bf8254fae8924f8139ce06> 


using System;
using System.Threading;
using System.Threading.Tasks;
namespace MultiThreadTest
{
    class Program
    {
        static void Main(string[] args)
        {
            var cancelSource = new CancellationTokenSource();
            CancellationToken token = cancelSource.Token;
            Task task = Task.Factory.StartNew(() =>
            {
                // Do some stuff...
                token.ThrowIfCancellationRequested();  // Check for cancellation request
                // Do some stuff...
            }, token);
            cancelSource.Cancel();
            try
            {
                task.Wait();
            }
            catch (AggregateException ex)
            {
                if (ex.InnerException is OperationCanceledException)
                    Console.Write("Task canceled!");
            }
            Console.ReadLine();
        }
    }
}


取消任务
如果想要支持取消任务，那么在建立任务的时候，需要传入一个CancellationTokenSouce 
