# 线程库

## 头文件

**头文件**

```cpp
#include <thread>
```

**命名空间**

```cpp
std
```

## 创建线程示例

```cpp
#include <thread>
void fun()
{
	//do some work
}
int main()
{
	std::thread t(fun);
	t.join();
}
```

在 上例 中，函数 fun将会运行于线程对象 t 中， join 函数将会阻塞main线程， 直到 t 线程执行结束

注意：std:: thread 出了 作用域之后将会析 构， 这时如果线程函数还没有执行完则会发生错误， 因此，加个`t.join`, 保证主函数执行完毕前，t线程执行完毕

## thread类的成员函数

### join

```cpp
void join();  //原型
```

等待线程完成其执行，即阻塞当前线程直至*this所标识的线程执行结束

### detach

```cpp
void detach();
```

容许线程从线程句柄独立开来执行 

从 thread 对象分离执行线程，允许执行独立地持续。一旦该线程退出，则由操作系统释放任何分配的资源。

调用 `detach` 后 *this 不再占有任何线程。


### get_id

```cpp
std::thread::id get_id() const noexcept  //noexcept表示当前函数不会抛出
```

返回标识与 *this 关联的线程的std::thread::id，注意这个线程id只是thread类内的那个成员id，非操作系统线程id

示例：

```cpp
#include <iostream>
#include <thread>
#include <chrono>
 
void foo()
{
    std::this_thread::sleep_for(std::chrono::seconds(1));
}
 
int main()
{
    std::thread t1(foo);
    std::thread::id t1_id = t1.get_id();
 
    std::thread t2(foo);
    std::thread::id t2_id = t2.get_id();
 
    std::cout << "t1's id: " << t1_id << '\n';
    std::cout << "t2's id: " << t2_id << '\n';
 
    t1.join();
    t2.join();
}
```

 <img src="img/C++%EF%BC%9A%E5%A4%9A%E7%BA%BF%E7%A8%8B.img/image-20210125110352581.png" alt="image-20210125110352581" style="zoom:67%;" />

### joinable

```cpp
bool joinable() const noexcept;
```

检查 `std::thread` 对象是否标识活跃的执行线程，是返回true，否返回false，只要线程没有被撤销就属于活跃状态，内核对象存在就是活跃

示例：

```cpp
void foo() { std::this_thread::sleep_for(std::chrono::seconds(1)); }

int main()
{
	std::thread t;
	std::cout << std::boolalpha << t.joinable() << std::endl;//false

	t = std::thread(foo);
	std::cout << std::boolalpha << t.joinable() << std::endl;//true

	t.join();
	std::cout << std::boolalpha << t.joinable() << std::endl;//false
}
//boolalpha的作用是使bool型变量按照false、true的格式输出。如不使用该标识符，那么结果会按照1、0的格式输出。
```

### swap

```cpp
void swap( std::thread& other ) noexcept;
```

交换两个thread对象的底层句柄

```cpp
#include <iostream>
#include <thread>
#include <chrono>
 
void foo(){ std::this_thread::sleep_for(std::chrono::seconds(1));}
 
void bar(){	std::this_thread::sleep_for(std::chrono::seconds(1));}
 
int main()
{
    std::thread t1(foo);
    std::thread t2(bar);
 
    std::cout << "thread 1 id: " << t1.get_id() << '\n'
              << "thread 2 id: " << t2.get_id() << '\n';
 
    t1.swap(t2);  //等价于：std::swap(t1, t2);
 
    std::cout << "after std::swap(t1, t2):" << '\n'
              << "thread 1 id: " << t1.get_id() << '\n'
              << "thread 2 id: " << t2.get_id() << '\n';
 
    t1.join();
    t2.join();
}
```

 <img src="img/C++%EF%BC%9A%E5%A4%9A%E7%BA%BF%E7%A8%8B.img/image-20210125105445832.png" alt="image-20210125105445832" style="zoom:67%;" />

**类外的swap**

```cpp
void swap( std::thread &t1, std::thread &t2 ) noexcept;
```

`std::swap(t1, t2);`就等价于`t1.swap(t2);`

## this_thread命名空间的函数

### sleep_for

```cpp
template< class Rep, class Period >
void sleep_for( const std::chrono::duration<Rep, Period>& sleep_duration );//参数是睡眠的时长
```

阻塞当前线程使其睡眠`sleep_duration`时间

使用示例：

```cpp
std::this_thread::sleep_for(std::chrono::milliseconds(2000));//阻塞当前线程使其睡眠2000ms
```

### sleep_until

```cpp
template< class Clock, class Duration >
void sleep_until( const std::chrono::time_point<Clock,Duration>& sleep_time );//参数是要阻塞到的时间点
```

阻塞当前线程，直至抵达指定的 `sleep_time`时间点

### yield

```cpp
void yield() noexcept;
```

让当前线程让渡出自己的CPU时间片(给其他线程使用)，避免一个线程频繁与其他线程争抢CPU时间片, 从而导致多线程处理性能下降

使用示例：

```cpp
#include <iostream>
#include <chrono>
#include <thread>

void little_sleep(std::chrono::microseconds us)
{
	auto start = std::chrono::high_resolution_clock::now();  //获取当前时间点
	auto end = start + us;
	do
	{
		std::this_thread::yield();
	} while (std::chrono::high_resolution_clock::now() < end);  //end之前一直让渡时间片
}

int main()
{
	auto  start = std::chrono::high_resolution_clock::now();

	little_sleep(std::chrono::microseconds(100));

	auto elapsed = std::chrono::high_resolution_clock::now() - start;//间隔的时间

	std::cout << "waited for " 
			 << std::chrono::duration_cast<std::chrono::microseconds>(elapsed).count() 
		     << " microseconds\n";
}
```

 <img src="img/C++%EF%BC%9A%E5%A4%9A%E7%BA%BF%E7%A8%8B.img/image-20210125112709829.png" alt="image-20210125112709829" style="zoom:67%;" />

### get_id

```cpp
std::thread::id get_id() noexcept;
```

与上面thread类成员函数的get_id是同一个函数，只是因为它还是this_thread命名空间的，所以这里单独列出

返回当前线程的std::thread::id，同样注意这个id不是操作系统的那个线程id

使用示例：

```cpp
#include <iostream>
#include <thread>
#include <chrono>
#include <mutex>
 
std::mutex g_display_mutex;
 
void foo()
{
    std::thread::id this_id = std::this_thread::get_id();
 
    g_display_mutex.lock();
    std::cout << "thread " << this_id << " sleeping...\n";
    g_display_mutex.unlock();
 
    std::this_thread::sleep_for(std::chrono::seconds(1));
}
 
int main()
{
    std::thread t1(foo);
    std::thread t2(foo);
 
    t1.join();
    t2.join();
}
```

 <img src="img/C++%EF%BC%9A%E5%A4%9A%E7%BA%BF%E7%A8%8B.img/image-20210125113306231.png" alt="image-20210125113306231" style="zoom:67%;" />

## 线程参数

线程可以接受任意个数的参数，如下代码：

```cpp
#include <thread>
#include<string>
#include <iostream>

void func(int i, double d, const std::string& s)
{
	std::cout << i << ", " << d << ", " << s << std::endl;
}
int main()
{
	std::thread t(func, 1, 2, "test");
	t.join();
	return 0;
}
```

## 移动线程

线程不能复制，但能移动，例如：

```cpp
#include <thread>

void func()	{} 
int main()
{
    std::thread t1(func);
    std::thread t2(std::move(t1));
    
    t2.join();
    return 0;
}
```

# 互斥量mutex

## 头文件

```cpp
#include <mutex>
```

`mutex` 类是能用于保护共享数据免受多个线程同时访问的同步原语。

## mutex使用示例

```cpp
#include <thread>
#include <iostream>
#include <mutex>

int g_count = 0;
std::mutex g_mtx;

void fun()
{
	g_mtx.lock();
	for (int i = 0; i < 1000000; ++i)
	{	
		++g_count;	
	}
	g_mtx.unlock();
}

int main()
{
	std::thread t1(fun);
	std::thread t2(fun);
	t1.join();
	t2.join();

	std::cout << g_count << std::endl;
}

```

![image-20210125120711988](img/C++%EF%BC%9A%E5%A4%9A%E7%BA%BF%E7%A8%8B.img/image-20210125120711988.png)

## mutex类的成员函数

### lock

```cpp
void lock();
```

锁定互斥。若另一线程已锁定互斥，则到 `lock` 的调用将阻塞执行，直至获得锁。

注意：通常不直接调用 `lock()` ：用 `std::unique_lock`或 `std::lock_guard` 管理排他性锁定。

### try_lock

```cpp
bool try_lock();
```

尝试锁定互斥。立即返回。成功获得锁时返回 true ，否则返回 false

### unlock

```cpp
void unlock();
```

解锁互斥。互斥必须被当前执行线程所锁定，否则行为未定义。

注意：通常不直接调用 `lock()` ：用 `std::unique_lock`或 `std::lock_guard` 管理排他性锁定。


## std::lock_guard

- 创建即加锁，作用域结束自动析构并解锁，无需手工解锁
- 不能中途解锁，必须等待作用域结束才解锁
- 不能复制

利用RAII风格自动解锁

类 `lock_guard` 是互斥体包装器，为在作用域块期间占有互斥提供便利 RAII 风格

创建 `lock_guard` 对象时，它试图接收给定互斥的所有权。控制离开创建 `lock_guard` 对象的作用域时，销毁 `lock_guard` 并释放互斥。

**创建lock_guard对象**

```cpp
std::mutex g_mtx;
std::lock_guard<std::mutex> lock(g_mtx);
```

**使用示例：**

```cpp
#include <thread>
#include <iostream>
#include <mutex>

int g_count = 0;
std::mutex g_mtx;

void fun()
{
	std::lock_guard<std::mutex> lock(g_mtx);
	for (int i = 0; i < 1000000; ++i)
	{	
		++g_count;	
	}
}

int main()
{
	std::thread t1(fun);
	std::thread t2(fun);
	t1.join();
	t2.join();

	std::cout << g_count << std::endl;
}
```

## std::unique_lock

unique_lock 是 lock_guard 的升级加强版，它具有 lock_guard 的所有功能，同时又具有其他很多方法，使用起来更强灵活方便，能够应对更复杂的锁定需要。

类 unique_lock 是通用互斥包装器，允许延迟锁定、锁定的有时限尝试、递归锁定、所有权转移和与条件变量一同使用。

- 创建时可以不锁定（通过指定第二个参数为std::defer_lock），而在需要时再锁定
- 可以随时加锁解锁
- 作用域规则同 lock_grard，析构时自动释放锁
- 不可复制，可移动
- 当条件变量需要该类型的锁作为参数，必须使用unique_lock

**创建uniqiue_lock对象**

```cpp
std::mutex mtx;
std::unique_lock<std::mutex> lock1(g_mtx);
```

**使用示例：**

```cpp
#include <thread>
#include <iostream>
#include <mutex>

int g_count = 0;
std::mutex g_mtx;

void fun()
{
	std::unique_lock<std::mutex> lock(g_mtx);
	for (int i = 0; i < 1000000; ++i)
	{	
		++g_count;	
	}
}

int main()
{
	std::thread t1(fun);
	std::thread t2(fun);
	t1.join();
	t2.join();

	std::cout << g_count << std::endl;
}
```

**lock_guard与unique_lock主要区别**

unique_lock和lock_guard都是管理锁的辅助类工具，都是RAII风格；它们是在定义时获得锁，在析构时释放锁。它们的主要区别在于==unique_lock锁机制更加灵活，可以再需要的时候进行lock或者unlock调用，不非得是析构或者构造时==。它们的区别可以通过成员函数就可以一目了然

![image-20210129164038307](img/C++%EF%BC%9A%E7%BA%BF%E7%A8%8B%E5%BA%93%E3%80%81%E4%BA%92%E6%96%A5%E9%87%8F%E7%AD%89.img/image-20210129164038307.png)

# 条件变量

> 条件变量内容转自[C++11条件变量使用详解](https://blog.csdn.net/c_base_jin/article/details/89741247)

`condition_variable` 类是同步原语，能用于阻塞一个线程，或同时阻塞多个线程，直至另一线程修改共享变量（*条件*）并通知 `condition_variable` 。

在C++11中，我们可以使用条件变量（condition_variable）实现多个线程间的同步操作，条件变量主要包括两个动作：

- 当条件不满足时，相关线程被一直阻塞
- 直到某种条件出现，这些线程才会被唤醒

为了防止竞争，条件变量的使用总是和一个互斥锁结合在一起；通常情况下这个锁是std::mutex，并且管理这个锁 只能是 std::unique_lock\<std::mutex> RAII模板类。

- 等待条件成立使用的是condition_variable类成员wait 、wait_for 或 wait_until
- 给出信号使用的是condition_variable类成员notify_one或者notify_all函数

## 头文件

```cpp
#include <condition_variable>
```

## 使用示例

## condition_variable的成员函数

### notify_one

```cpp
void notify_one() noexcept;
```

若任何线程在 *this 上等待，则调用 `notify_one` 会唤醒一个阻塞的线程

### notify_all

```cpp
void notify_all() noexcept;
```

唤醒当前等待于 *this 的全部线程 

### wait

```cpp
void wait( std::unique_lock<std::mutex>& lock );

template< class Predicate >
void wait( std::unique_lock<std::mutex>& lock, Predicate pred );//Predicate 谓词函数，可以普通函数或者lambda表达式	
```

`wait` 导致当前线程阻塞直至条件变量被通知，或虚假唤醒发生，可循环wait直至满足某条件

wait做的三件事：

- 阻塞
- 让锁
- 再次持有锁时返回

**wait函数都在会阻塞时，自动释放锁权限，即调用unique_lock的成员函数unlock()，以便其他线程能有机会获得锁**。这就是条件变量只能和unique_lock一起使用的原因，unique_lock锁机制相比于lock_guard更加灵活，可以在需要的时候进行lock或者unlock调用

## 虚假唤醒

在正常情况下，wait类型函数返回时要么是因为被唤醒，要么是因为超时才返回，但是在实际中发现，因为操作系统的原因，wait类型在不满足条件时，它也会返回，这就导致了虚假唤醒。

因此，我们一般都是使用带有谓词参数的wait函数，因为这种`(xxx, Predicate pred )`类型的函数等价于：

```cpp
while (!pred()) //while循环，解决了虚假唤醒的问题
{
    wait(lock);
}
```

如果虚假唤醒发生，由于while循环，再次检查条件是否满足，不满足就再次执行wait，解决了虚假唤醒

## 生产者、消费者问题

生产者、消费者问题：共享固定大小缓冲区的两个进程/线程，生产者往里面添加数据，消费者消耗数据

该问题的关键就是要==保证生产者不会在缓冲区满时加入数据，消费者也不会在缓冲区中空时消耗数据==

要解决该问题，就必须让**生产者在缓冲区满时休眠（要么干脆就放弃数据），等到下次消费者消耗缓冲区中的数据的时候，生产者才能被唤醒，开始往缓冲区添加数据**。

同样，**也可以让消费者在缓冲区空时进入休眠，等到生产者往缓冲区添加数据之后，再唤醒消费者**。

示例代码：

```cpp
std::mutex g_cvMutex;
std::condition_variable g_cv;

//缓存区
std::deque<int> g_data_deque;
//缓存区最大数目
const int  MAX_NUM = 30;
//数据
int g_next_index = 0;

//生产者，消费者线程个数
const int PRODUCER_THREAD_NUM  = 3;
const int CONSUMER_THREAD_NUM = 3;

void  producer_thread(int thread_id)
{
	 while (true)
	 {
	     std::this_thread::sleep_for(std::chrono::milliseconds(500));
	     //加锁
	     std::unique_lock <std::mutex> lk(g_cvMutex);
	     //当队列未满时，继续添加数据
	     g_cv.wait(lk, [](){ return g_data_deque.size() <= MAX_NUM; });
	     g_next_index++;
	     g_data_deque.push_back(g_next_index);
	     std::cout << "producer_thread: " << thread_id << " producer data: " << g_next_index;
	     std::cout << " queue size: " << g_data_deque.size() << std::endl;
	     //唤醒其他线程 
	     g_cv.notify_all();
	     //自动释放锁
	 }
}

void  consumer_thread(int thread_id)
{
    while (true)
    {
        std::this_thread::sleep_for(std::chrono::milliseconds(550));
        //加锁
        std::unique_lock <std::mutex> lk(g_cvMutex);
        //检测条件是否达成
        g_cv.wait( lk,   []{ return !g_data_deque.empty(); });
        //互斥操作，消息数据
        int data = g_data_deque.front();
        g_data_deque.pop_front();
        std::cout << "\tconsumer_thread: " << thread_id << " consumer data: ";
        std::cout << data << " deque size: " << g_data_deque.size() << std::endl;
        //唤醒其他线程
        g_cv.notify_all();
        //自动释放锁
    }
}

int main()
{
    std::thread arrRroducerThread[PRODUCER_THREAD_NUM];
    std::thread arrConsumerThread[CONSUMER_THREAD_NUM];

    for (int i = 0; i < PRODUCER_THREAD_NUM; i++)
    {
        arrRroducerThread[i] = std::thread(producer_thread, i);
    }

    for (int i = 0; i < CONSUMER_THREAD_NUM; i++)
    {
        arrConsumerThread[i] = std::thread(consumer_thread, i);
    }

    for (int i = 0; i < PRODUCER_THREAD_NUM; i++)
    {
        arrRroducerThread[i].join();
    }

    for (int i = 0; i < CONSUMER_THREAD_NUM; i++)
    {
        arrConsumerThread[i].join();
    }
	return 0;
}
```

 <img src="img/C++%EF%BC%9A%E7%BA%BF%E7%A8%8B%E5%BA%93%E3%80%81%E4%BA%92%E6%96%A5%E9%87%8F%E3%80%81%E6%9D%A1%E4%BB%B6%E5%8F%98%E9%87%8F.img/image-20210224210114717.png" alt="image-20210224210114717" style="zoom:50%;" />

# 参考文献

> 《深入应用C++11：代码优化与工程级应用》
>
> [cppreference.com](https://zh.cppreference.com/w/cpp/thread/thread)
>
> [c++11中的lock_guard和unique_lock使用浅析](https://blog.csdn.net/guotianqing/article/details/104002449)
>
> [C++11条件变量使用详解](https://blog.csdn.net/c_base_jin/article/details/89741247)

