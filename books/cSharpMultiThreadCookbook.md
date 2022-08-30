# C#多线程编程实战

```
tips:格式控制
$"{var, 11}"
```

## 第一章 线程基础

```c#
// 创建线程
Thread t = new Thread(PrintNumbers);
t.Start();

// 暂停线程
Sleep(TimeSpan.FromSeconds(2));

// 线程等待
t.Join(); // 主线程等待线程t完成，阻塞，和上面调用了Thread.Sleep效果相同

// 终止线程
t.Abort(); // 会产生异常ThreadAbortException，非常危险
// 目标线程可通过处理异常并调用t.ResetAbort来拒绝终止
// 因此不推荐用Abort来关闭线程

// 检测线程状态
WriteLine(CurrentThread.ThreadState.ToString());
WriteLine(t.ThreadState.ToString()); // 枚举 Unstarted Running Aborted AbortRequested WaitSleepJoin Stopped

// 线程优先级
t.Priority = ThreadPriority.Highest; // 最高优先级
t.Priority = ThreadPriority.Lowest; // 最低优先级
GetCurrentProcess().ProcessorAffinity = new IntPtr(1); // 让操作系统将所有线程运行在单个CPU核心上

// 前台线程 后台线程
t.IsBackground = true; // 进程会等待所有前台线程完成后结束工作，只剩下后台线程则直接结束工作。

// 线程参数
void Count(int i) { }
t = new Thread(Count);
t.Start(8); // 只能传一个 object
t = new Thread(() => Count(10));

// lock关键字
class CounterWithLock
{
	private readonly object = _syncRoot = new Object();
	public int Count { get; private set; }
	public void Increment() {
		lock(_syncRoot) {
			Count++;
		}
	}
	public void Decrement() {
		lock(_syncRoot) {
			Count--;
		}
	}
}

// Monitor 和 死锁deadlock
new Thread(()=>{
	lock(lock1)
	{
		Sleep(1000);
		lock(lock2);
	}
}).Start();
lock(lock2)
{
	Sleep(1000);
	// 死锁
	lock(lock1)
	{
		// 进不来
	}
	// Monitor
	if (Monitor.TryEnter(lock1, TimeSpan.FromSeconds(5)))
	{
		// 进不来
	}
	else
	{
		// 进入
	}
}
// 实际上 lock是Monitor的语法糖
// lock 等于
bool acquiredLock = false;
try
{
	Monitor.Enter(lockObject, ref acquiredLock);
	// Code that accesses resources that are protected by the lock.
}
finally
{
	if (acquiredLock)
	{
		Monitor.Exit(lockObject);
	}
}
//因此，我们可以直接用Monitor类

// 处理异常
// 在线程中始终使用try/catch代码块是非常重要的，因为不可能在线程代码外捕获异常
t = new Thread(()=>{
	try
	{
		throw new Exception("Boom!");
	}
	catch(Exception ex)
	{
		// 可以捕获异常
	}
});

try
{
	t = new Thread(()=>{
		throw new Excepton("Boom!");
	});
}
catch(Exception ex)
{
	// 不可捕获异常
}
```

# 第二章 线程同步

```c#
// 同步线程使得对共享对象的操作以正确顺序执行是非常重要的
// 一个线程执行操作，其他线程需要依次等待。这种常见问题通常称为线程同步

// 首先，无须共享对象，就无须进行线程同步
// 大多数时候可以通过重新设计程序来移除共享状态，从而去掉复杂的同步构造
// 尽可能避免在多个线程使用单一对象

// 如果必须使用共享的状态，第二种方式是只用原子操作
// 这意味着一个操作只占用一个量子时间。
// 所以只有当操作完成后，其他线程才能执行其他操作。
// 因此，无须实现其他线程等待当前操作完成，这就避免了使用锁，也排除了死锁的情况。

// 如果上面的方式不可行，并且程序的逻辑更加复杂，那么我们不得不用不同的方法来协调线程。

// kernel-mode
// 一种方法是将等待的线程置于阻塞状态。
// 当线程阻塞时，只会占用尽可能少的CPU时间。然而，这将引入至少一次上下文切换 context switch
// 上下文切换，指的是操作系统的线程调度器。
// 该调度器会保存等待的线程状态，并切换到另一个线程，一次恢复等待的线程状态。
// 这需要消耗相当多的资源。
// 如果线程要被挂起很长时间，这样做是值得的。
// 这种方式又被称为内核模式 kernel-mode，因为只有操作系统的内核才能阻止线程使用CPU时间

// user-mode
// 万一线程只需要等待一点点时间，最好只是简单等待，不要阻塞。
// 虽然浪费了CUP时间，但是节省了切换上下文耗费的CPU时间
// 这种方式被称为用户模式 user-mode
// 非常轻量，快速，但是如果线程需要等待较长时间，则会浪费大量CPU

// hybrid
// 为了更好利用这两种模式，可以使用混合模式
// 混合模式现场时 用户模式 等待，如果等了足够长时间，则会切换到 阻塞模式 以节省CPU资源

// 执行基本的原子操作
Interlocked.Increment(ref _count);
Interlocked.Decrement(ref _count);

// 使用Mutex类
// Mutex是一种原始的同步方式，只对一个线程授予对共享资源的独占访问
using (var m = new Mutex(false, "CSharpThreadingCookbook"))
{
    if (!m.WaitOne(TimeSpan.FromSeconds(5), false))
    {
        WriteLine("Second instance is running!");
    }
    else
    {
        WriteLine("Running!");
        ReadLine();
        m.ReleaseMutex();
    }
}
// 启动时，定义了一个互斥量，设置initialOwner为false
// 这意味着如果互斥量已经被创建，则允许程序获取该互斥量
// 如果没有获取互斥量，程序简单显示Running 等待按键 释放互斥量并退出
// 再运行一个同样程序，5秒内会尝试获取互斥量。
// 如果第一个程序此时解锁互斥量，第二个程序会执行。
// 互斥量是全局的操作系统对象，一定要正确关闭互斥量，最好用using代码块包裹互斥量对象

// 使用SemaphoreSlim类
// 该类限制了同时访问同一个资源的线程数量
static SemaphoreSlim _semaphore = new SemaphoreSlim(4);

static void AccessDatabase(string name, int seconds)
{
	WriteLine($"{name} waits to access a database");
	_semaphore.Wait();
	WriteLine($"{name} was granted an access to a database");
	Sleep(TimeSpan.FromSeconds(seconds));
	WriteLine($"{name} is completed");
	_semaphore.Release();
}

static void Main(string[] args)
{
	for (int i = 1; i <= 6; i++)
	{
		string threadName = "Thread " + i;
		int secondsToWait = 2 + 2 * i;
		var t = new Thread(()=>AccessDatabase(threadName, secondsToWait));
		t.Start();
	}
}
// SemaphoreSlim使用了混合模式，等待很短时间无需上下文切换。
// Semaphore是老版本，纯粹使用内核时间，一般没必要使用它，除非是重要场景。
// 可以创建一个具名semaphore，就像一个mutex，从而在不同程序中同步线程。
// SemaphoreSlim不使用Windows内核信号量，也不支持进程间同步。
// 跨进程可以使用Semaphore

// 使用AutoResetEvent类
// 线程间发送通知，通知等待的线程有事发生
private static AutoResetEvent _workerEvent = new AutoResetEvent(false);
private static AutoResetEvent _mainEvent = new AutoResetEvent(false);

static void Process(int seconds)
{
	WriteLine("Starting a long running work...");
	Sleep(TimeSpan.FromSeconds(seconds));
	WriteLine("Work is done");
	_workerEvent.Set();
	WriteLine("Waiting for a main thread to complete its work");
	_mainEvent.WaitOne();
	WriteLine("Starting second operation...");
	Sleep(TimeSpan.FromSeconds(seconds));
	WriteLine("Work is done");
	_workerEvent.Set();
}

static void Main(string[] args)
{
	var t = new Thread(() => Process(10));
	t.Start();
	WriteLine("Waiting for another thread to complete work");
	_workerEvent.WaitOne();
	WriteLine("First operation is completed");
	WriteLine("Performing an operation on a main thread");
	Sleep(TimeSpan.FromSeconds(5));
	_mainEvent.Set();
	WriteLine("Now running the second operation on a second thread");
	_workerEvent.WaitOne();
	WriteLine("Second operation is completed");
}
// 两个线程相互发信号
// AutoResetEvent采用内核时间模式，所以等待时间不能太长
// ManualResetEventSlim更好，使用混合模式

// ManualResetEventSlim工作方式像是人群通过大门
// AutoResetEvent像是旋转门，一次允许一个人通过
// ManualResetEventSlim是ManualResetEvent的混合版本，一直开门知道手动调用Reset关门
// 如果需要全局事件，可以使用EventWaitHandle类，是AutoResetEvent和ManualResetEvent的基类

// CountDownEvent等待信号次数
static CountdownEvent _countdown = new CountdownEvent(2);
_countdown.Signal();
_countdown.Wait();
_countdown.Dispose();

// Barrier类
// Barrier类用于组织多个线程及时在某个时刻碰面。
static Barrier _barrier = new Barrier(2, b =>
	WriteLine($"End of phase {b.CurrentPhaseNumber + 1}"));

static void PlayMusic(string name, string message, int seconds)
{
	for (int i = 0; i < 3; i++)
	{
		WriteLine("----------------------------------------");
		Sleep(TimeSpan.FromSeconds(seconds));
		WriteLine($"{name} starts to {message}");
		Sleep(TimeSpan.FromSeconds(seconds));
		WriteLine($"{name} finishes to {message}");
		_barrier.SignalAndWait();
	}
}

static void Main(string[] args)
{
	var t1 = new Thread(() => PlayMusic("the guitarist", "play an amazing solo", 5));
	var t2 = new Thread(() => PlayMusic("the singer", "sing his song", 2));

	t1.Start();
	t2.Start();
}
// 创建了Barrier类，指定了想要同步两个线程。
// 两个线程任意一个调用了SingleAndWait方法后，会执行一个回调函数，打印出阶段。

// ReaderWriterLockSlim类
// 多线程对一个集合读写，允许多个线程同时读取，独占写
static ReaderWriterLockSlim _rw = new ReaderWriterLockSlim();
static Dictionary<int, int> _items = new Dictionary<int, int>();

static void Read()
{
	WriteLine("Reading contents of a dictionary");
	while (true)
	{
		try
		{
			_rw.EnterReadLock();
			foreach (var key in _items.Keys)
			{
				Sleep(TimeSpan.FromSeconds(0.1));
			}
		}
		finally
		{
			_rw.ExitReadLock();
		}
	}
}

static void Write(string threadName)
{
	while (true)
	{
		try
		{
			int newKey = new Random().Next(250);
			_rw.EnterUpgradeableReadLock();
			if (!_items.ContainsKey(newKey))
			{
				try
				{
					_rw.EnterWriteLock();
					_items[newKey] = 1;
					WriteLine($"New key {newKey} is added to a dictionary by a {threadName}");
				}
				finally
				{
					_rw.ExitWriteLock();
				}
			}
		}
		finally
		{
			_rw.ExitUpgradeableReadLock();
		}
	}
}

static void Main(string[] args)
{
	new Thread(Read) { IsBackground = true }.Start();
	new Thread(Read) { IsBackground = true }.Start();
	new Thread(Read) { IsBackground = true }.Start();
	new Thread(()=>Write("Thread 1")) { IsBackground = true }.Start();
	new Thread(()=>Write("Thread 2")) { IsBackground = true }.Start();

	Sleep(TimeSpan.FromSeconds(30));
}
// 当主程序启动时，同时运行三个线程读取字典数据，另外两个线程写入数据。
// 读锁允许多线程读取数据，写锁在被释放前会阻塞了其他线程的所有操作。
// 获取读锁时，根据当前数据而决定是否获取一个写锁并修改该集合，一旦得到写锁，
// 会阻止阅读者读取数据，从而浪费大量时间。
// 为了最小化阻塞时间，需要EnterUpgradeableReadLock和ExitUpgradeableReadLock
// 先获取读锁后读取数据，如果发现需要修改底层集合，使用EnterWriteLock方法升级
// 然后快速执行一次写操作，最后使用ExitWriteLock释放写锁

// SpinWait
// 使用用户模式等待一段时间，然后切换到内核模式以节省CPU时间
static volatile bool _isCompleted = false;

static void UserModeWait()
{
	while (!_isCompleted)
	{
		Write("*");
	}
	WriteLine();
	WriteLine("Waiting is complete");
}

static void HybridSpinWait()
{
	var w = new SpinWait();
	while (!_isCompleted)
	{
		w.SpinOnce();
		WriteLine(w.NextSpinWillYield);
	}
	WriteLine("Waiting is complete");
}

static void Main(string[] args)
{
	var t1 = new Thread(UserModeWait);
	var t2 = new Thread(HybridSpinWait);

	WriteLine("Running user mode waiting");
	t1.Start();
	Sleep(20);
	_isCompleted = true;
	Sleep(TimeSpan.FromSeconds(1));
	_isCompleted = false;
	WriteLine("Running hybrid SpinWait construct waiting");
	t2.Start();
	Sleep(5);
	_isCompleted = true;
}
// volatile关键字指出，字段可能被同时执行的多个线程修改。
// 字段不会被编译器和处理器优化为只能被单个线程访问。这确保了该字段总是最新。
// SpinWait刚开始使用用户模式，九个迭代以后，开始切换线程为阻塞状态。
// 线程将不会浪费CPU。
```

---

# 第三章 线程池

```c#
// 保持线程中的操作都是短暂的非常重要。
// 不要在线程池中放入长时间运行的操作，或者阻塞工作线程。
// 这会导致性能问题和难以调试的错误。
// 重要：线程池的用途是执行运行时间短的操作。
// 线程池中的工作线程都是后台线程。这意味着当前所有前台线程完成后，所有后台线程停止。
```

```c#
// 在线程池中调用委托
// 如何异步地执行委托，异步编程模型Asynchronous Programming Model APM
private delegate string RunOnThreadPool(out int threadId);

private static void Callback(IAsyncResult ar)
{
	WriteLine("Starting a callback...");
	WriteLine($"State passed to a callbak: {ar.AsyncState}");
	WriteLine($"Is thread pool thread: {CurrentThread.IsThreadPoolThread}");
	WriteLine($"Thread pool worker thread id: {CurrentThread.ManagedThreadId}");
}

private static string Test(out int threadId)
{
	WriteLine("Starting...");
	WriteLine($"Is thread pool thread: {CurrentThread.IsThreadPoolThread}");
	Sleep(TimeSpan.FromSeconds(2));
	threadId = CurrentThread.ManagedThreadId;
	return $"Thread pool worker thread id was: {threadId}";
}

static void Main(string[] args)
{
	int threadId = 0;

	RunOnThreadPool poolDelegate = Test;

	var t = new Thread(() => Test(out threadId));
	t.Start();
	t.Join();

	WriteLine($"Thread id : {threadId}");

	IAsyncResult r = poolDelegate.BeginInvoke(out threadId, Callback,
		"a delegate asynchronous");
	r.AsyncWaitHandle.WaitOne();

	string result = poolDelegate.EndInvoke(out threadId, r);

	WriteLine($"Thread pool worker thread id: {threadId}");
	WriteLine(result);

	Sleep(TimeSpan.FromSeconds(2));
}
//首先，用旧的方式创建lambda
//然后，定义了一个委托，调用BeginInvoke执行委托
//BeginInvoke立刻返回，结果，当工作线程执行异步操作时
//我们仍可以继续其他操作
//当需要异步操作结果时，可以用result.IsCompleted轮询结果
//本例中，使用AsyncWaitHandle属性等待直到操作完成。
//操作完成后，得到的结果可用EndInvoke方法获得
//    事实上，AsyncWaitHandle.WaitOne不必要
//    注释掉，代码照样成功运行。
//    因为，EndInvoke方法事实上会等待异步操作完成。
//    调用EndInvoke非常重要，因为该方法会将任何未处理的 异常抛回到调用线程中
//    一定要确保Begin和End方法始终调用
// 使用BeginOperationName/EndOperationName方法和IAsyncResult对象等方式
// 被称为异步编程模型APM模式，这样的方法对 称为异步方法
// 但是在现代编程中，更推荐使用 任务并行库 TaskParallelLibrary TPL
```

```c#
// 向线程池中放入异步操作
private static void AsyncOperation(object state)
{
	WriteLine($"Operation state: {state ?? "(null)"}");
	WriteLine($"Worker thread id: {CurrentThread.ManagedThreadId}");
	Sleep(TimeSpan.FromSeconds(2));
}

static void Main(string[] args)
{
	const int x = 1;
	const int y = 2;
	const string lambdaState = "lambda state 2";

	ThreadPool.QueueUserWorkItem(AsyncOperation);
	Sleep(TimeSpan.FromSeconds(1));

	ThreadPool.QueueUserWorkItem(AsyncOperation, "async state");
	Sleep(TimeSpan.FromSeconds(1));

	ThreadPool.QueueUserWorkItem(state =>
	{
		WriteLine($"Operation state: {state}");
		WriteLine($"Worker thread id: {CurrentThread.ManagedThreadId}");
		Sleep(TimeSpan.FromSeconds(2));
	}, "lambda state");

	ThreadPool.QueueUserWorkItem(_ =>
	{
		WriteLine($"Operation state: {x + y}, {lambdaState}");
		WriteLine($"Worker thread id: {CurrentThread.ManagedThreadId}");
		Sleep(TimeSpan.FromSeconds(2));
	}, "lambda state");

	Sleep(TimeSpan.FromSeconds(2));
}
//可以用闭包，所以之前的回调机制冗余又过时，不再需要了。
```

---