# 半同步半异步线程池

线程池分为：半同步半异步线程池，领导者追随者线程池，本文主要介绍半同步半异步线程池

**同步**：Synchronization，调用某个东西时，调用方得等待这个调用返回结果才能继续往后执行

**异步**：Asynchronization，和同步相反 调用方不会理解得到结果，而是在调用发出后调用者可用继续执行后续操作，被调用者通过状态来通知调用者，或者通过回掉函数来处理这个调用

![image-20210223104542195](img/C++%EF%BC%9A%E7%BA%BF%E7%A8%8B%E6%B1%A0.img/image-20210223104542195.png)

![image-20210223102225820](img/C++%EF%BC%9A%E7%BA%BF%E7%A8%8B%E6%B1%A0.img/image-20210223102225820.png)

**银行模拟半同步半异步模型：**

![image-20210223130634822](img/C++%EF%BC%9A%E5%8D%8A%E5%90%8C%E6%AD%A5%E5%8D%8A%E5%BC%82%E6%AD%A5%E7%BA%BF%E7%A8%8B%E6%B1%A0.img/image-20210223130634822.png)

同步：从门口进来的客户向排号机添加请求的时候是同步的（同步处理客户逻辑），我用排号机的时候你不能用，你得等我用完了再用

异步：每一个窗口从排号机获得请求之后进行的处理是异步的（异步处理I/O事件），窗口1与窗口2之间没有依赖关系，也没有共享资源，1在处理的时候，2也可以在处理，3也可以在处理，（处理是指从排号机获取请求之后，对请求进行的的处理，排号机本身是需要锁保护的）

**半同步半异步分层模型：**

 <img src="img/C++%EF%BC%9A%E7%BA%BF%E7%A8%8B%E6%B1%A0.img/image-20210223103938617.png" alt="image-20210223103938617" style="zoom:50%;" />

- 同步服务层：它处理来自上层的任务请求，上层的请求可能是并发的，这些请求不是马上就会处理，而是将这些任务放到一个同步队列中，等待处理
- 同步排队层：来自上层的的任务请求都会加到排同步队列中等待处理
- 异步服务层：这一层中会有多个线程同时处理排队层中的任务，异步服务层从同步排队层中取出任务并行的处理

**半同步半异步线程池活动图：**

 <img src="img/C++%EF%BC%9A%E7%BA%BF%E7%A8%8B%E6%B1%A0.img/image-20210223105545210.png" alt="image-20210223105545210" style="zoom:50%;" />

**编写同步队列：**

银行职员从缓冲队列中take请求，客户往缓冲队列中Add请求

![image-20210223130537847](img/C++%EF%BC%9A%E5%8D%8A%E5%90%8C%E6%AD%A5%E5%8D%8A%E5%BC%82%E6%AD%A5%E7%BA%BF%E7%A8%8B%E6%B1%A0.img/image-20210223130537847.png)

```cpp
//半同步半异步线程池
#include <list>
#include <mutex>
#include <thread>
#include <condition_variable>
#include <memory>  //智能指针进行内存管理
#include <atomic>    //原子操作
#include <functional>
#include <iostream>
using namespace std;

class Task  //客户：可以派生出各种不同的客户类型：普通客户、VIP客户、开户客户、销户客户等等
{
public:
	Task() { cout << "Create Task" << endl; }
	virtual ~Task() { cout << "Destroy Task" << endl; }
	virtual void run()//模拟任务正在处理
	{
		cout << "Task " << endl;
	}
};

class OpenTask : public Task	//开户客户
{
public:
	virtual void run()
	{
		cout << "open a count" << endl;//模拟正在开户
		this_thread::sleep_for(std::chrono::seconds(5));  
	}
};
class DestroyTask :public Task  //销户客户
{
	virtual void run()
	{
		cout << "cancel a count" << endl;//模拟正在销户
		this_thread::sleep_for(std::chrono::seconds(10));
	}
};

template<class T>
class SyncQueue  //同步队列
{
public:
	SyncQueue(int maxSize) :m_maxSize(maxSize), m_needStop(false) {}
	~SyncQueue() { Stop(); }

	void Put(const T & x)  //往缓冲队列添加任务的供外部使用的接口
	{
		Add(x);
	}
	void Put(const T && x)
	{
		Add(std::forward<T> (x));
	}

	//从缓冲队列获取任务，每次取一个
	bool Take(T & t)  
	{
		std::unique_lock<std::mutex> locker(m_mutex);  //加锁保护缓冲队列

		 //没有停止并且缓冲队列已经空了，阻塞，while循环防止虚假唤醒
		while (!m_needStop && !NotEmpty()) 
		{
			m_notEmpty.wait(locker);  //阻塞，直到add线程添加任务后notify它
		}

		if (m_needStop)  //队列需要停止工作
		{
			return false;
		}
		t = m_queue.front();  //取缓冲队列第一个元素
		m_queue.pop_front();
		m_notFull.notify_one();   //每次取出任务后，要唤醒有可能因队满阻塞的添加任务的线程
		return true;
	}

	//把队列中的任务全部取出来
	bool Take(std::list<T> &list)
	{
		std::unique_lock<std::mutex> locker(m_mutex);
		while (!m_needStop && !NotEmpty())
		{
			m_notEmpty.wait(locker);
		}
		if (m_needStop)
		{
			return false;
		}
		list = std::move(m_queue);//把队列中元素全部移走
		m_notFull.notify_one();
		return true;
	}

	void Stop()//停止工作
	{
		{
			std::lock_guard<std::mutex> locker(m_mutex);
			m_needStop = true;
		}
		m_notFull.notify_all();   //唤醒所有等待的线程，继续向下执行if (m_needStop) return;
		m_notEmpty.notify_all();


	}
	bool Empty() const  //缓冲队列为空
	{
		std::lock_guard<std::mutex> locker(m_mutex);
		return m_queue.empty();
	}
	bool Full() const  //缓冲队列为满
	{
		std::lock_guard<std::mutex> locker(m_mutex);
		return m_queue.size() >= m_maxSize;
	}
	size_t Size() const  //缓冲队列可容纳事物的大小
	{
		std::lock_guard<std::mutex> locker(m_mutex);
		return m_queue.size();
	}
	size_t Count() const
	{
		std::lock_guard<std::mutex> locker(m_mutex);
		return m_queue.size();
	}

private:
	bool Is_Full() const  //判满
	{
		bool full = (m_queue.size() >= m_maxSize);
		if (full)
		{
			cout << "缓冲队列满了，需要等待..." << endl;
		}
		return full;
	}
	bool Is_Empty() const  //判空
	{
		bool empty = m_queue.empty();
		if (empty)
		{
			cout << "缓冲队列空了，需要等待...  需要等待的异步层id：" << this_thread::get_id() << endl;	 
		}
		return empty;
	}
	bool NotFull() const { return !Is_Full(); }  //没有满
	bool NotEmpty() const { return !Is_Empty(); } //没有空
	template<class U>

	//向缓冲队列添加任务
	void Add(U && x)  
	{
		std::unique_lock<std::mutex> locker(m_mutex);  //加锁保护缓冲队列

		//没有停止并且满了，阻塞，while循环防止虚假唤醒
		while (!m_needStop && !NotFull())  
		{														  
			m_notFull.wait(locker); //直到take线程从满的缓冲队列取出元素，唤醒该线程，
		}	

		if (m_needStop)  //队列需要停止工作
		{
			return;
		}
		m_queue.push_back(std::forward<U>(x));  //完美转发向对列添加
		m_notEmpty.notify_one();   //每次添加任务后，要唤醒有可能阻塞的take线程											  
	}
private:
	std::list<T> m_queue;							//缓冲队列，放任务
	std::mutex   m_mutex;							//互斥量
	std::condition_variable  m_notEmpty;  //非空条件变量，notEmpty意为不为空就继续执行，不为空才满足条件
	std::condition_variable  m_notFull;      //未满条件变量
	size_t m_maxSize;								//队列最大元素个数
	bool m_needStop;                               //为true时，同步队列停止工作

};

//线程池
//-----------------------------------------------------------------
const int MaxTaskCount = 5;
class ThreadPool
{
public:
	ThreadPool(int numThreads = std::thread::hardware_concurrency()) :m_queue(MaxTaskCount)//thread::hardware_concurrency 获取硬件支持并发数
	{
		Start(numThreads);
	}
	~ThreadPool()
	{
		Stop();
	}
	void Stop()
	{
		std::call_once(m_flag, [this]() { StopThreadGroup(); });  //call_once不管有多少个线程只执行一次
	}
	void AddTask(std::shared_ptr<Task> && task)  //向同步队列中添加任务
	{
		m_queue.Put(task);
	}
	void AddTask(const std::shared_ptr<Task> && task)
	{
		m_queue.Put(task);
	}

private:
	void Start(int numThreads)  //创建numThreads个线程
	{
		m_running = true;
		for (int i = 0; i < numThreads; ++i)
		{
			//创建工作线程
			m_threadgroup.push_back(std::make_shared<std::thread>(&ThreadPool::RunInThread, this));
		}
	}
	void RunInThread()  //运行的线程函数
	{
		while (m_running)
		{
			//从缓冲队列中获取请求去调用run
			std::list<std::shared_ptr<Task>> list;
			m_queue.Take(list);
			for (auto &task : list)
			{
				if (!m_running)
				{
					return;
				}
				task->run();
			}
		}
	}
	void StopThreadGroup()  //停止工作线程组
	{
		m_queue.Stop();
		m_running = false;
		for (auto & thread : m_threadgroup)
		{
			if (thread)
			{
				thread->join();
			}
		}
		m_threadgroup.clear(); 
	}
private:
	std::list<std::shared_ptr<std::thread>>  m_threadgroup;  //工作线程组，银行职员
	SyncQueue<std::shared_ptr<Task>> m_queue;  //同步队列，排号机
	std::atomic_bool m_running;  //是否处于运行
	std::once_flag m_flag;  //多线程环境只执行一次
};

//测试----------------------------------------------
void funa(ThreadPool & pool)  //向同步队列中添加任务
{
	for (int i = 0; i < 10; ++i)
	{
		auto thdId = this_thread::get_id();
		cout << "同步线程1的 ID ：" << thdId << endl;
		pool.AddTask(std::shared_ptr<Task>(new OpenTask()));
	}
}

void funb(ThreadPool & pool)  //向同步队列中添加任务
{
	for (int i = 0; i < 10; ++i)
	{
		auto thdId = this_thread::get_id();
		cout << "同步线程2的 ID ：" << thdId << endl;
		pool.AddTask(std::shared_ptr<Task>(new Task()));
	}
}

int main()
{
	ThreadPool pool(2);

	this_thread::sleep_for(std::chrono::seconds(2));

	std::thread thd1(funa, std::ref(pool));  //同步线程
	std::thread thd2(funb, std::ref(pool));

	this_thread::sleep_for(std::chrono::seconds(2));

	pool.Stop();
	thd1.join();
	thd2.join();
	getchar();
	return 0;
}
```

# std::once_flag、std::call_once

在多线程中，有一种场景是某个任务只需要执行一次，可以用C++11中的std::call_once函数配合std::once_flag来实现。多个线程同时调用某个函数，std::call_once可以保证多个线程对该函数只调用一次

上述线程池中使用到的地方：

```cpp
std::once_flag m_flag;  //多线程环境只执行一次

void Stop()
{
    std::call_once(m_flag, [this]() { StopThreadGroup(); });  
    //call_once不管有多少个线程同时调用只执行一次
}
```

多线程环境只调用一次示例：

```cpp
#include <thread>
#include <mutex>  //call_once头文件
#include <iostream>
using namespace std;

std::once_flag g_flag;

void doSome()
{
	std::call_once(g_flag, []() {cout << "i am in call_once" << endl; });
	cout << "i am not in call_once" << endl;
}

int main()
{
	std::thread t1(doSome);
	std::thread t2(doSome);
	std::thread t3(doSome);

	std::this_thread::sleep_for(std::chrono::seconds(1));

	t1.join();
	t2.join();
	t3.join();
}
```

![image-20210223194133361](img/C++%EF%BC%9A%E5%8D%8A%E5%90%8C%E6%AD%A5%E5%8D%8A%E5%BC%82%E6%AD%A5%E7%BA%BF%E7%A8%8B%E6%B1%A0.img/image-20210223194133361.png)

底层机理：当该函数第一次被一个线程执行，编译器就把call_once里的once_flag标记一下，当别的线程也执行到std::call_once，检查到里面的once_flag已经被改动，就不执行该函数

注意： 

- once_flag的生命周期，它必须要比使用它的线程的生命周期要长。所以通常定义成全局变量或静态变量比较好。
- 控制只调用一次的前提是用的once_flag是同一个，不同flag没法达到控制的目的

# 线程安全的单例模式

```cpp
#include <thread>
#include <mutex>  //call_once头文件
#include <iostream>
using namespace std;

class Task
{
private:
	Task();
public:
	static Task * task;
	static Task * getInstance();
};

Task* Task::task = nullptr;

Task::Task()
{
	cout << "Create Task " << endl;
}

Task * Task::getInstance()  //懒汉模式：在第一次调用的时候才进行实例化
{
	static std::once_flag flag;  //必须加static，保证flag用的是同一个
	std::call_once(flag, [](){  task = new Task();  });
	return task;
}

void threadfun()
{
	Task *task = Task::getInstance();
	cout << "task's address is " << task << endl;
}
int main()
{
	std::thread t1(threadfun);
	std::thread t2(threadfun);
	std::thread t3(threadfun);

	std::this_thread::sleep_for(std::chrono::seconds(1));

	t1.join();
	t2.join();
	t3.join();

	return 0;
}
```

![image-20210223200415371](img/C++%EF%BC%9A%E5%8D%8A%E5%90%8C%E6%AD%A5%E5%8D%8A%E5%BC%82%E6%AD%A5%E7%BA%BF%E7%A8%8B%E6%B1%A0.img/image-20210223200415371.png)

# static多线程问题

**静态对象在多线程中会创建几次？**

答：一次

**静态对象在多线程环境中线程安全吗？**

答：不安全，虽然静态对象在多线程中只会创建一次，但是该静态对象的值具体是多少是线程不安全的。编译的时候静态对象的空间会被开辟在全局.data区，静态对象有一个标志域，对静态对象的赋值和对标志域的修改是分步操作的，并不是原子操作。

有可能一个线程对静态对象赋了值，但还没改标志域，这时候另一个线程执行到给该对象赋值，发现标志域还没改，便成功赋值，然后改掉标志域，这时候前一个线程接着执行改标志域操作，最后结果本来应该是第一个线程给该对象赋值，结果赋的值确是第二个线程的，线程不安全

```cpp
class Test
{
public:
	Test(int x) { cout << "create test : " << x << endl; }
};

void fun(int x)
{
	static Test test(x);
}
int main()
{
	thread t1(fun, 1);
	thread t2(fun, 2);
	thread t3(fun, 3);

	std::this_thread::sleep_for(std::chrono::seconds(1));

	t1.join();
	t2.join();
	t3.join();
}
```

![image-20210223203937296](img/C++%EF%BC%9A%E5%8D%8A%E5%90%8C%E6%AD%A5%E5%8D%8A%E5%BC%82%E6%AD%A5%E7%BA%BF%E7%A8%8B%E6%B1%A0.img/image-20210223203937296.png)









