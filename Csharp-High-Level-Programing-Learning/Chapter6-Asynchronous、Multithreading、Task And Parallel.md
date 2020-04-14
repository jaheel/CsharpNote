## 第六章 异步、多线程、任务和并行

### 建议71：区分异步和多线程应用场景

CLR提供的异步编程模型就是让我们充分利用硬件的DMA功能来释放CPU的压力。

**优化前：**

```
private void buttonGetPage_Click(object sender,EventArgs e)
{
   Thread t=new Thread(()=>
   {
   var request=HttpWebRequest.Create("某个URL");
   var response=request.GetResponse();
   var stream=response.GetResponseStream();
   using(StreamReader reader=new StreamReader(stream))
   {
      var content=reader.ReadLine();
      textBoxPage.Text=content;
   }
   });
   t.start();
}
```

​       该段代码实现了单击Button获取某个网页的内容并显示出来，如果该网页内容过多，或当前网络状况不好，获取网页的过程就将持续很长时间。

```
   var request=HttpWebRequest.Create("某个URL");
   var response=request.GetResponse();
   var stream=response.GetResponseStream();
```

​       该段代码一直在CPU中实现，直到获取网页完全以后才释放CPU资源，工作线程一直被占用，效率低。使用**异步模式**改变它，代码如下：

```
private void buttonGetPage_Click(object sender,EventArgs e)
{
   var request=HttpWebRequest.Create("某个URL");
   request.BeginGetResponse(this.AsyncCallbackImp1,request);
}

public void AsyncCallbackImp1(IAsyncResulit ar)
{
   WebRequest request=ar.AsyncState as WebRequest;
   var response=rquest.EndGetResponse(ar);
   var stream=response.GetResponseStream();
   using(StreamReader reader=new StreamReader(stream))
   {
      var content=reader.ReadLine();
      textBoxPage.Text=content;
   }
}
```

​        使用线程池进行管理，新起异步操作后，CLR会将工作丢给线程池中的某个工作线程来完成。开始I/O操作的时候，异步就会将工作线程还给线程池（获取网页的这个工作不会再占用任何CPU资源了）。直到异步完成（获取网页完毕），异步才会通过回调的方式通知线程池，让CLR响应异步完毕。

**异步模式借助于线程池，极大地节约了CPU的资源**



* **计算密集型**工作，采用**多线程**
* **IO密集型**工作，采用**异步机制**



### 建议72：在线程同步中使用信号量

锁定使用**关键字lock和类型Monitor**。两者没有本质区别，前者是后者的语法糖。（同步技术）

​        信号同步机制中涉及的类型都继承自抽象类WaitHandle，这些类型有EventWaitHandle(类型化为AutoResetEvent、ManualResetEvent)、Semaphore以及Mutex都继承自WaitHandle。

* **EventWaitHandle维护一个由内核产生的bool型对象**（成为“阻滞状态”），如果其值为false，那么在它上面等待的线程就阻塞。可以调用类型的Set方法将其设置为true，解除阻塞。

* **Semaphore维护一个由内核产生的整型变量**，如果其值为0，则在它上面等待的线程就会阻塞；如果其值大于0，则解除阻塞，同时，每解除一个线程阻塞，其值就减1。

* **Mutex为我们提供了跨应用程序域阻塞和解除阻塞线程的能力**。

> AutoResetEvent和ManualResetEvent的区别是：前者在发送信号完毕后（即调用Set方法），会自动将自己的阻塞状态设置为false,而后者则需要进行手动设定。

### 建议73：避免锁定不恰当的同步对象

​        C#中，让线程同步的另一种编码方式：**线程锁**。原理：锁住一个资源，使得应用程序在此刻只有一个线程访问该资源。（让多线程变成单线程）

> 同步对象在需要同步的多个线程中是可见的同一个对象。
>
> 在非静态方法中，静态变量不应作为同步对象。
>
> 值类型对象不能作为同步对象。
>
> 避免将字符串作为同步对象。
>
> 降低同步对象的可见性。

```
public Form1()
        {
            InitializeComponent();
        }

       AutoResetEvent autoSet = new AutoResetEvent(false);
        List<string> tempList = new List<string>() { "init0", "init1", "init2" };

        private void buttonStartThreads_Click(object sender, EventArgs e)
        {
            object syncObj = new object();

            Thread t1 = new Thread(() =>
            {
                //确保等待t2开始之后才运行下面的代码
                autoSet.WaitOne();
                lock (syncObj)
                {
                    foreach (var item in tempList)
                    {
                        Thread.Sleep(1000);
                    }
                }
            });
            t1.IsBackground = true;
            t1.Start();

            Thread t2 = new Thread(() =>
            {
                //通知t1可以执行代码
                autoSet.Set();
                //沉睡1秒是为了确保删除操作在t1的迭代过程中
                Thread.Sleep(1000);
                lock (syncObj)
                {
                    tempList.RemoveAt(1);
                }
            });
            t2.IsBackground = true;
            t2.Start();
        }
```

**类型的静态方法应当保证线程安全，非静态方法不需实现线程安全**

### 建议74：警惕线程的IsBackground

线程分为前台线程和后台线程，每个线程都有一个IsBackground属性。两者在表现形式上的唯一区别是：如果前台线程不退出，应用程序的进程就会一直存在，必须所有的前台线程全部退出，应用程序才算退出。而后台进程则没有这方面限制，如果应用程序退出，后台线程也会一并退出。

**IsBackground属性默认是false**

* 实际编码中**应更多地使用后台线程**。只有在非常关键的工作中，如线程正在执行事务或占有的某些托管资源需要释放时，才使用前台线程。

### 建议75：警惕线程不会立即启动

**线程之间的调度占有一定的时间和空间开销，并且，它不是实时。**

注意：**同步代码**



### 建议76：警惕线程的优先级

C#线程有5个优先级：Highest、AboveNormal、Normal、BelowNormal和Lowest

Windows系统是 **基于优先级的抢占式调度系统**

### 建议77：正确停止线程

### 建议78：应避免线程数量过多

### 建议79：使用ThreadPool或BackgroundWorker代替Thread

线程的空间开销：

①线程内核对象（Thread Kernel Object)

②线程环境块（Thread Environment Block）

③用户模式栈（User Mode Stack）

④内核模式栈（Kernel Mode Stack）



如果我们要多线程编码，不应想到：

```
Thread t=new Thread(()=>
{
   //工作代码
}
);
```

应想到依赖线程池：

```
ThreadPool.QueueUserWorkItem((objState)=>
{
    //工作代码
},null);
```

**线程池技术能让我们重点关注业务的实现，而不是线程的性能测试**

BackgroundWorker类型内部使用了线程池技术，同时，还给工作线程和UI线程提供了交互的能力。

Thread和ThreadPool默认都没有提供这种交互能力，而BackgroundWorker却通过事件提供了这种能力。（能力包括：报告进度、支持完成回调、取消任务、暂停任务等）

**从事Winform或WPF开发的人员，可考虑使用BackgroundWorker。**



### 建议80：用Task代替ThreadPool

* ThreadPool不支持线程的取消、完成、失败通知等交互性操作
* ThreadPool不支持线程执行的先后次序



Task具备以下属性，可以让我们查询任务完成时的状态：

* IsCanceled 因为被取消而完成
* IsCompleted 成功完成
* IsFaulted 因为发生异常而完成

**ContinueWith方法可以在一个任务完成的时候发起一个新任务**



只要涉及多线程内容的，都将一并使用Task来完成



### 建议81：使用Parallel简化同步状态下Task的使用

Parallel主要提供3个有用的方法：For、ForEach、Invoke。

* **For方法**：主要用于处理针对数组元素的并行操作

```
static void Main(string[] args)
        {
            int[] nums = new int[] { 1, 2, 3, 4 };
            Parallel.For(0, nums.Length, (i) =>
            {
                Console.WriteLine("针对数组索引{0}对应的那个元素{1}的一些工作代码……", i, nums[i]);
            });
            Console.ReadKey();
        }
```

**如果我们的输出必须是同步的或者说必须是顺序输出的，则不应使用Parallel的方式**

* **ForEach方法**：主要用于处理泛型集合元素的并行操作

```
static void Main(string[] args)
        {
            List<int> nums = new List<int> { 1, 2, 3, 4 };
            Parallel.ForEach(nums, (item) =>
            {
                Console.WriteLine("针对集合元素{0}的一些工作代码……", item);
            });
            Console.ReadKey();
        }
```

* **Invoke方法**：简化了启动一组并行操作，隐式启动的就是Task，该方法接受Params Action[]参数

```
static void Main(string[] args)
        {
            Parallel.Invoke(() =>
            {
                Console.WriteLine("任务1……");
           },
                () =>
              {
                    Console.WriteLine("任务2……");
               },
               () =>
                {
                    Console.WriteLine("任务3……");
                });
            Console.ReadKey();
        }
```

**由于所有任务都是并行的，所以不保证先后次序**



### 建议82：Parallel简化但不等同于Task默认行为

### 建议83：小心Parallel中的陷阱

### 建议84：使用PLINQ

让LINQ支持并行计算

传统的LINQ计算的单线程的，PLINQ则是并发的、多线程的。

```
static void Main(string[] args)
        {
            List<int> intList = new List<int>() { 0, 1, 2, 3, 4, 5, 6, 7, 8, 9 };
            var query = from p in intList select p;
            Console.WriteLine("以下是LINQ顺序输出：");
            foreach (int item in query)
            {
                Console.WriteLine(item.ToString());
            }
            Console.WriteLine("以下是PLINQ并行输出：");
            var queryParallel = from p in intList.AsParallel() select p;
            foreach (int item in queryParallel)
            {
                Console.WriteLine(item.ToString());
            }
        }
```

**AsOrdered方法可以对并行计算后的队列进行重新组合，以便保持顺序**：

```
var queryParallel=from p in intList.AsParallel().AsOrdered() select p;
foreach(int item in queryParallel)
{
   Console.WriteLine(item.ToString());
}
```

**Take查询方法**:

```
foreach(int item in queryParallel.Take(5))
{
   Console.WriteLine(item.ToString());
}
```

在顺序查询中，会返回前5个元素。但在PLINQ中，会选出5个无序的元素。



### 建议85：Task中的异常处理

### 建议86：Parallel中的异常处理

### 建议87：区分WPF和WinForm的线程模型

### 建议88：并行并不总是速度更快

### 建议89：在并行方法体重谨慎使用锁

如果方法体的全部内容都需要同步运行，就完全不应该使用并行

