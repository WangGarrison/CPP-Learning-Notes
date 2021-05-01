# RAII

**1. 什么是RAII**

RAII（Resource Acquisition Is Initialization）是由 c++之父 Bjarne Stroustrup 提出的，中文翻译为资源获取即初始化，使用局部对象来管理资源的技术称为资源获取即初始化；这里的资源主要是指操作系统中有限的东西如内存、网络套接字，互斥量，文件句柄等等，局部对象是指存储在栈的对象，它的生命周期是由操作系统来管理的，无需人工介入  

**2. RAII 的原理**  

资源的使用一般经历三个步骤：

​	a. 获取资源 （创建对象）

​	b. 使用资源

​	c. 销毁资源（析构对象）

但是资源的销毁往往是程序员经常忘记的一个环节，所以程序界就想如何在程序员中让资源自动销毁呢？解决问题的方案是： RAII，它充分的利用了 C++语言局部对象自动销毁的特性来控制资源的生命周期。给一个简单的例子来看下局部对象的自动销毁的特性 

```cpp
#include<string>
#include <iostream>
using namespace std;
class Student
{
public:
	Student(const string name = "", int age = 0) :
	s_name(name), s_age(age) { cout << "Init a Student!" << endl;}
	~Student() {cout << "Destory a Student!" << endl; }
	const string& getname() const { return this->s_name;}
	int getage() const { return this->s_age; }
private:
	const string s_name;
	int s_age;
};
int main()
{
	Student s("zhangsan"); // 必须是局部对象
	return 0;
}
```

 <img src="img/C++%EF%BC%9A%E6%99%BA%E8%83%BD%E6%8C%87%E9%92%88.img/image-20201128194158564.png" alt="image-20201128194158564" style="zoom:50%;" />

从 Student class 可以看出，当我们在 main 函数中声明一个局部对象的时候，会自动调用构造函数进行对象的初始化，当整个 main 函数执行完成后，自动调用析构函数来销毁对象，整个过程无需人工介入，由操作系统自动完成；于是，很自然联想到，当我们在使用资源的时候，在构造函数中进行初始化，在析构函数中进行销毁。

整个 RAII 过程总结四个步骤：

1. 设计一个类封装资源

2. 在构造函数中初始化

3. 在析构函数中执行销毁操作

4. 使用时声明一个该类的对象    

# 智能指针

### 为什么要使用智能指针？裸指针的缺点

裸指针：int *p这样的常规指针

裸指针的缺点：

1、难以区分指向的是单个对象还是一组对象（数组）

```cpp
int *p = new int;
int *ps = new int[10];
```

2、使用完指针之后无法判断是否应该销毁指针，因为无法判断指针是否“拥有”指向的对象

3、在已经确定需要销毁指针的情况下，也无法确定使用delete关键字删除，还是有其他特殊的销毁机制，例如通过将指针传入某个特定的销毁函数来销毁指针

4、即便已经确定了销毁指针的方法，由于1的原因，仍然无法确定到底是用delete（销毁单个对象）还是delete[]（销毁一个数组）

5、假设上述的问题都解决了，也很难保证在代码的所有路径中（分支结构、异常导致的跳转），有且仅有一次销毁指针操作；任何一条路径遗漏都可能导致内存泄漏，而销毁多次则会导致未定义的行为

6、理论上没有方法来分辨一个指针是否处于悬挂状态

原始指针带来了上述一系列问题，C++发明智能指针来解决这些问题

C++里面的四个智能指针: auto_ptr, unique_ptr,shared_ptr, weak_ptr 其中后三个是 C++11 支持，第一个已经被 C++11 弃用。

C++的 auto_ptr 所做的事情，就是动态分配对象以及当对象不再需要时自动执行清理。  

# auto_ptr

```cpp
namespace yhp
{
    template<class T>
    class auto_Ptr
    {
    private:
        T *Ptr;
    public:
        auto_ptr(T *p):_Ptr(p) {} 
        ~auto_ptr() { delete _Ptr; }
        
        T *get() const
        {
            return _Ptr;
        }
        T & operator*() const //重载解引用符
        {
            return (*get());
        }
        T * operator->() const //重载指向符
        {
            return get();
        }
    };
};

void fun()
{
    yhp::auto_ptr<int> iap(new int(10));
}
```

把构建的10的地址给p, p再去初始化_Ptr, _Ptr就指向了10这个空间的地址，局部函数结束时，自动调用析构函数，delete掉\_Ptr, 就实现了指针的自动销毁

对于如下代码：

![image-20201128143427108](img/C++%EF%BC%9A%E6%99%BA%E8%83%BD%E6%8C%87%E9%92%88.img/image-20201128143427108.png)

oap是智能指针类的对象，点访问的就是它自己的方法，而它重载了->，->访问的就是Object类的方法

`oap->Value();`里->其实被用了两次，operator->返回的是个T类型(推演成Object类型)的地址，之后又用->取这个地址里的方法，但不能->->这样写

添加_Owns后的代码：

```cpp
namespace yhp
{
    template<class T>
    class auto_Ptr
    {
    private:
        bool _Owns;  //拥有则为true
        T *Ptr; 
    public:
        typedef T element_type;
    public:
        explicit auto_ptr(T *p = nullptr):_Ptr(p),_Owns(p != nullptr) {} 
        auto_ptr(const auto_ptr & _Y): _Owns(_Y._Owns),_Ptr(_Y.release())//拷贝构造函数
        {
        }
        auto_ptr & operator=(const auto_ptr _Y)
        {
            if(this != _Y)
            {
                if(_Ptr != _Y.get())
                {
                    if(_Owns)
                    {
                        delete _Ptr;
                    }
                    _Owns = _Y.Owns;
				}
                else if(_Y.Owns)
                {
                    _Owns = true;
                }
                _Ptr = _Y.release;
            }
            return *this;
		}
        
        ~auto_ptr() 
        {
            if(_Owns)//拥有这个资源，才能释放
            {
                delete _Ptr;
            }
            _Ptr = NULL;
        }
        
        T *get() const
        {
            return _Ptr;
        }
        T & operator*() const //重载解引用符
        {
            return (*get());
        }
        T * operator->() const //重载指向符
        {
            return get();
        }
        T * release() const  // 释放拥有权
        {
            ((auto_ptr<T> *)this)->_Owns = false;
            return _Ptr;
        }
    };
};

void fun()
{
    yhp::auto_ptr<int> iap(new int(10));
}
```

![image-20201128152857935](img/C++%EF%BC%9A%E6%99%BA%E8%83%BD%E6%8C%87%E9%92%88.img/image-20201128152857935.png)

![image-20201128191838911](img/C++%EF%BC%9A%E6%99%BA%E8%83%BD%E6%8C%87%E9%92%88.img/image-20201128191838911.png)

### auto_ptr正式的仿写代码

```cpp
namespace yhp
{
	template<class _Ty>
	class auto_ptr
	{
	public:
		typedef _Ty element_type;
		explicit auto_ptr(_Ty *_P = 0): _Owns(_P != 0), _Ptr(_P) {}
		auto_ptr(const auto_ptr<_Ty>& _Y): _Owns(_Y._Owns), _Ptr(_Y.release()){}
		auto_ptr<_Ty>& operator=(const auto_ptr<_Ty>& _Y)
		{
			if (this != &_Y)
			{
				if (_Ptr != _Y.get())
				{
					if (_Owns)
					{
						delete _Ptr;
					}
					_Owns = _Y._Owns;
				}
				else if (_Y._Owns)
				{
					_Owns = true;
				}
				_Ptr = _Y.release();
			}
			return (*this);
		}
		~auto_ptr()
		{
			if (_Owns)
			{
				delete _Ptr; // delete []_Ptr;
            }
            _Ptr = NULL;
		}
		_Ty& operator*() const
		{
			return (*get());
		}
		 _Ty *operator->() const
		{
			return (get());
		}
		_Ty *get() const
		{
			return (_Ptr);
		}
		_Ty *release() const
		{
			((auto_ptr<_Ty> *)this)->_Owns = false;
			return (_Ptr);
		}
	private:
		bool _Owns;
		_Ty *_Ptr;
	};
}
```

### 构造与析构函数

auto_ptr 在构造时获取对某个对象的所有权(ownership),在析构时释放该对象。我们可以这样使用auto_ptr 来提高代码安全性：  

```cpp
int *p = new int(0);
auto_ptr<int> ap(p);
```

从此我们不必关心应该何时释放 p，也不用担心发生异常会有内存泄漏  

C++11已经舍弃了auto_ptr，因为该智能指针有很多弊端，如下

**注意事项：**

1. 两个auto_ptr 不能同时拥有同一个对象。像这样：

   ```cpp
   int *p = new int(0);
   auto_ptr<int> ap1(p);
   auto_ptr<int> ap2(p);
   ```

   因为 ap1 与 ap2 都认为指针 p 是归它管的，在析构时都试图删除 p, 两次删除同一个对象的行为在C++标准中是未定义的。所以我们必须防止这样使用 auto_ptr.

2. 不能管理数组指针，如下代码：

   ```cpp
   String *sar = new String[10];
   auto_ptr<String> ap(sar);
   ```

   因为 auto_ptr 的析构函数中删除指针用的是 delete,而不是 delete [],所以我们不应该用 auto_ptr 来管理一个数组指针

3. 构造函数的 explicit 关键词有效阻止从一个“裸”指针隐式转换成 auto_ptr 类型  

### 拷贝构造与赋值

与引用计数型智能指针不同的， auto_ptr 要求其对“裸”指针的完全占有性。也就是说一个“裸”指针不能同时被两个以上的 auto_ptr 所拥有。那么，在拷贝构造或赋值操作时，我们必须作特殊的处理来保证这个特性。 auto_ptr 的做法是“==所有权转移==”，即拷贝或赋值的源对象将失去对“裸”指针的所有权，所以，与一般拷贝构造函数，赋值函数不同， auto_ptr 的拷贝构造函数，赋值函数的参数为引用而不是常引用(constreference).当然，一个 auto_ptr 也不能同时拥有两个以上的“裸”指针，所以，拷贝或赋值的目标对象将先释放其原来所拥有的对象 

 **这里的注意点是：**  

1. 因为一个 auto_ptr 被拷贝或被赋值后，其已经失去对原对象的所有权，这个时候，对这个 auto_ptr 的提领(dereference)操作是不安全的。如下:  

   ```cpp
   int *p = new int(0);
   auto_ptr<int> ap1(p);
   auto_ptr<int> ap2 = ap1;  //ap1的所有权也给ap2了，即ap1的_Owns被释放了
   cout<< *ap1; //错误，此时ap1已经失去对p指针的所有权
   ```

   这种情况较为隐蔽的情形出现在将 auto_ptr 作为函数参数按值传递，因为在函数调用过程中在函数的作用域中会产生一个局部对象来接收传入的 auto_ptr(拷贝构造)，这样，传入的实参 auto_ptr 就失去了其对原对象的所有权，而该对象会在函数退出时被局部 auto_ptr 删除。如下：

   ```cpp
   void fun(auto_ptr<int> ap)
   {
   	cout<< * ap;
   }
   
   auto_ptr<int> ap1(new int(0));
   fun(ap1);
   cout<<*ap1;//错误，经过 fun(ap1)函数调用， ap1 已经不再拥有任何对象了。
   ```

   因为这种情况太隐蔽，太容易出错了，所以 auto_ptr 作为函数参数按值传递是一定要避免的。或许大家会想到用 auto_ptr 的指针或引用作为函数参数或许可以，但是仔细想想，我们并不知道在函数中对传入的auto_ptr 做了什么，如果当中某些操作使其失去了对对象的所有权，那么这还是可能会导致致命的执行期错误。也许用 const reference 的形式来传递 auto_ptr 会是一个不错的选择

2. 我们可以看到拷贝构造函数与赋值函数都提供了一个成员模板在不覆盖“正统”版本的情况下实现 auto_ptr的隐式转换。如我们有以下两个类  

   ```cpp
   class Object {};
   
   class Base : public Object{};
   
   auto_ptr<Object> apobj = auto_ptr<Base>(new Base);
   ```

   那么最后一句代码就可以通过，实现从 auto_ptr\<Base>到 auto_ptr\<Object>的隐式转换，因为 Base*可以转换成Object\*类型
   
3. 因为 auto_ptr 不具有值语义(value semantics), 所以 auto_ptr 不能被用在 stl 标准容器中  

   >值语义
   >
   >所谓值语义是一个对象被系统标准的复制方式复制后，与被复制的对象之间毫无关系，可以彼此独立改变互不影响，所有的内置类型（primitive variables）都具有 value semantics。 有人也称它为POD (plain old data), 也就是旧时的老数据（有和 OOP 的新型抽象数据对比之意）  
   >
   >对一个具有值语义的变量赋值可以转换成内存的 bit-wise-copy（按位拷贝） 。
   >
   >什么样的类没有值语义呢？我们不妨称这种型为 none-value-semantics type (NVST).
   >1）有 virtual function 的类
   >2）包含 NVST 成员的类
   >3） NVST 的衍生类（derived classed）
   >4）定义了自己的 operator= 的类
   >5）继承 virtual 基类的衍生类  

   很明显， auto_ptr定义了自己的=重载函数，不符合上述条件，而我们知道 stl 标准容器要用到大量的拷贝赋值操作，并且假设其操作的类型必须符合以上条件，所以 auto_ptr 不能被用在 stl 标准容器中  

### 提领操作(dereference)

提领操作有两个操作， 一个是返回其所拥有的对象的引用， 另一个则是实现了通过 auto_ptr 调用其所拥有的对象的成员。如：  

```cpp
class Base
{
	void fun() {};
}

auto_ptr<A> ap (new Base()); 
(*ap).fun();  
ap->fun();	  
```

当然， 我们首先要确保这个智能指针确实拥有某个对象，否则，这个操作的行为即对空指针的提领是未定义  

### 辅助函数

1. get()返回auto_ptr指向的那个对象的内存地址，用来显式的返回 auto_ptr 所拥有的对象指针。我们可以发现，标准库提供的 auto_ptr 既不提供从“裸”指针到 auto_ptr 的隐式转换(构造函数为 explicit),也不提供从 auto_ptr 到“裸”指针的隐式转换，从使用上来讲可能不那么的灵活，考虑到其所带来的安全性还是值得的  

   ```cpp
   int* p = new int(33);
   
   cout << "the adress of p: "<< p << endl;
   
   auto_ptr ap1§;
   
   cout << "the adress of ap1: " << &ap1 << endl;
   
   cout << "the adress of the object which ap1 point to: " << ap1.get() << endl;
   ```

   输出如下：

   ```cpp
   the adress of p: 00481E00
   
   the adress of ap1: 0012FF68
   
   the adress of the object which ap1 point to: 00481E00
   ```

   第一行与第三行相同，都是int所在的那块内存的地址。第二行是ap1这个类对象本身所在内存的地址

2. reset，重新设置auto_ptr指向的对象，用来接收所有权，如果接收所有权的 auto_ptr 如果已经拥有某对象，必须先释放该对象。类似于赋值操作，但赋值操作不允许将一个普通指针指直接赋给auto_ptr，而reset()允许

   ```cpp
   auto_ptr< string > pstr_auto( new string( “Brontosaurus” ) );
   pstr_auto.reset( new string( “Long -neck” ) );
   ```

   在例子中，重置前pstr_auto拥有"Brontosaurus"字符内存的所有权，这块内存首先会被释放。之后pstr_auto再拥有"Long -neck"字符内存的所有权。

   注：reset(0)可以释放对象，销毁内存。

3.  release()，返回auto_ptr指向的那个对象的内存地址,并释放对这个对象的所有权。用来转移所有权（_Owns)

   用此函数初始化auto_ptr时可以避免两个auto_ptr对象拥有同一个对象的情况（与get函数相比）。release源码如下：

    <img src="img/C++%EF%BC%9A%E6%99%BA%E8%83%BD%E6%8C%87%E9%92%88.img/image-20201204200600203.png" alt="image-20201204200600203" style="zoom:50%;" />
   
   ```cpp
   auto_ptr< string > pstr_auto( new string( “Brontosaurus” ) );
   
   auto_ptr< string > pstr_auto2( pstr_auto.get() ); //这时两个auto_ptr拥有同一个对象
   
   auto_ptr< string > pstr_auto2( pstr_auto.release() ); //release可以首先释放所有权
   ```

### 使用auto_ptr

1、将已存在的指向动态内存的普通指针作为参数来构造

```cpp
int* p = new int(33);

auto_ptr<int> api(p);
```

2、直接构造智能指针

```cpp
auto_ptr< int > api( new int( 33 ) );
```

3、拷贝构造，利用已经存在的智能指针来构造新的智能指针

```cpp
auto_ptr< string > pstr_auto( new string( "Brontosaurus" ) );

auto_ptr< string > pstr_auto2( pstr_auto );  //利用pstr_auto来构造pstr_auto2
//因为一块动态内存智能由一个智能指针独享，所以在拷贝构造或赋值时都会发生拥有权转移的过程。在此拷贝构造过程中，pstr_auto将失去对字符串内存的所有权，而pstr_auto2将其获得。对象销毁时，pstr_auto2负责内存的自动销毁
```

4、赋值，利用已经存在的智能指针来构造新的智能指针

```cpp
auto_ptr< int > p1( new int( 1024 ) );

auto_ptr< int > p2( new int( 2048 ) );

p1 = p2;
//在赋值之前，由p1 指向的对象被删除。赋值之后，p1 拥有int 型对象的所有权。该对象值为2048。 p2 不再被用来指向该对象。
```

### 小结使用auto_ptr的注意事项

1. auto_ptr不能指向数组
2. auto_ptr不能共享所有权
3. auto_ptr不能通过复制操作来初始化
4. auto_ptr不能放入容器中来使用
5. auto_ptr不能作为容器的成员
6. 不能把一个原生指针给两个智能指针对象来管理（对所有的智能指针）

**空的auto_ptr 需要初始化吗？**

通常的指针在定义的时候若不指向任何对象，我们用Null给其赋值。对于智能指针，因为构造函数有默认值0，我们可以直接定义空的auto_ptr如下：

```cpp
auto_ptr< int > p_auto_int; //不指向任何对象
```

**防止两个auto_ptr对象拥有同一个对象（一块内存）**

因为auto_ptr的所有权独有，所以下面的代码会造成混乱。

```cpp
int* p = new int(0);
auto_ptr<int> ap1(p);
auto_ptr<int> ap2(p);
```

因为ap1与ap2都认为指针p是归它管的，在析构时都试图删除p,　两次删除同一个对象的行为在C++标准中是未定义的。所以我们必须防止这样使用auto_ptr。

**auto_ptr不能初始化为指向非动态内存**

原因很简单，delete 表达式会被应用在不是动态分配的指针上这将导致未定义的程序行为。

**警惕智能指针auto_ptr作为参数！**

按值传递时，函数调用过程中在函数的作用域中会产生一个局部对象来接收传入的auto_ptr(拷贝构造)，这样，传入的实参auto_ptr就失去了其对原对象的所有权，而该对象会在函数退出时被局部auto_ptr删除。如下例：

```cpp
void f(auto_ptr ap)
{cout<<*ap;}
int main()
{
	auto_ptr ap1(new int(0));
	f(ap1);
	cout<<*ap1; //错误，经过f(ap1)函数调用，ap1指向的对象已经被销毁了
}
```

引用或指针时，不会存在上面的拷贝过程。但我们并不知道在函数中对传入的auto_ptr做了什么，如果当中某些操作使其失去了对对象的所有权，那么这还是可能会导致致命的执行期错误，const reference是auto_ptr智能指针作为参数传递的底线

由于auto_ptr的种种弊端，C11舍弃掉了auto_ptr

以下介绍的是boost库里的智能指针

# unique_ptr

>**unique_ptr** 是 **C++ 11** 提供的用于防止内存泄漏的智能指针中的一种实现， 。unique_ptr对象包装一个原始指针，并负责其生命周期。当该对象被销毁时，会在其析构函数中删除关联的原始指针。
>unique_ptr具有`->`和`*`运算符重载符，因此它可以像普通指针一样使用。
>
>unique_ptr对象始终是关联的原始指针的唯一所有者。我们无法复制unique_ptr对象，它只能移动。
>由于每个unique_ptr对象都是原始指针的唯一所有者，因此在其析构函数中它直接删除关联的指针，不需要任何参考计数。

unique_ptr（唯一） 是一种定义在\<memory>中的智能指针(smart pointer)。不能进行复制操作只能进行移动操作

unique 是独特的、唯一的意思， 故名思议， unique_ptr 可以“独占”地拥有它所指向的对象，它提供一种严格意义上的所有权。  它通过重载强转bool类型，来实现查看是否占有，只要占有，该指针就是true，不占有就是false

unique_ptr 和 shared_ptr 类型指针有很大的不同： shared_ptr 允许多个指针指向同一对象，而 unique_ptr 在某一时刻只能有一个指针指向该对象(两个 unique_ptr 不能指向同一个对象)。unique_ptr 保存指向某个对象的指针，当它本身被删除或者离开其作用域时会自动释放其指向对象所占用的资源  、

### 如何创建 unique_ptr  

unique_ptr 不像 shared_ptr 一样拥有标准库函数 make_shared 来创建一个 shared_ptr 实例。要想创建一个 unique_ptr，我们需要将一个 new 操作符返回的指针传递给 unique_ptr 的构造函数。  

```cpp
int main() 
{
	// 创建一个 unique_ptr 实例
	unique_ptr<int> pInt(new int(5));
    cout << *pInt;
    unique_ptr<int> up1(new int());    //okay,直接初始化
	//unique_ptr<int> up2 = new int();   //error! 构造函数是explicit
	//unique_ptr<int> up3(up1);          //error! 不允许拷贝
}
```

### 无法进行复制构造和赋值

unique_ptr封锁掉了拷贝构造与=，如何禁掉默认的函数？把函数写在私有里边，或者用=delete

所以unique_ptr不支持普通的拷贝和赋值操作。  

```cpp
 int main() 
 {
	// 创建一个 unique_ptr 实例
	unique_ptr<int> pInt(new int(5));
	unique_ptr<int> pInt2(pInt); // 报错
	unique_ptr<int> pInt3;
	pInt3 = pInt; // 报错
}
```

### 仿写的代码

```cpp
#include <thread>
using namespace std;
namespace yhp
{
    template<typename _Tp>
    struct default_delete//删除一个对象
    {
		void operator()(_Tp* ptr) const
        {
            delete ptr;
        }
    };
    template<typename _Tp>
    struct default_delete<_Tp[]>//删除一组对象
    {
		void operator()(_Tp* ptr) const //重载括号运算符
        {
            delete []ptr;
        }
    };
    template<typename _Tp, typename _Dp = default_delete<_Tp>>//_Dp是删除器
    class unique_ptr
    {
    private:
        _Tp * ptr;
        C//禁止掉拷贝构造
        unique_ptr & operator=(const unique_ptr&) = delete;//禁止掉=语句的重载
    public:
        typedef _Tp* pointer;
        typedef _Tp element_type;
        typedef _Dp delete_type;
        
        delete_type get_delete() { return deletype(); }
        
    public:
        unique_ptr():ptr(nullptr) {}
        explicit unique_ptr(_Tp* p): ptr(p) {}
        unique_ptr& operator=(unique_ptr && uq)//移动拷贝函数
		{
    		reset(uq.release());
            return *this;
		}
        unique_ptr(unique_ptr && uq)//转移拷贝构造函数
        {
            ptr = uq.release();
        }
        ~unique_ptr() 
        {
            if(ptr != nullptr)
            {
                get_delete()(ptr);//调动get_delete()得到一个删除器对象，再调动删除器对象的operator()函数删除掉ptr
			}
            ptr = nullptr;
        }
        pointer get()const {return ptr;} //获得当前智能指针所指之物的地址
        _Tp & operator*()const {return *get(); }
        pointer operator->()const {return get(); }
        
        explicit operator bool() const //重载bool强转，explicit禁止隐式转换
        {
            return get() == nullptr ? false : true;
        }
        pointer release()//释放拥有权
        {
            pointer p = ptr;
            ptr = nullptr;
            return p;
        }
        void reset(pointer p = nullptr)//重设拥有的对象
        {
            get_delete()(ptr);//调动get_delete()得到一个删除器对象，再调动删除器对象的operator()函数删除掉ptr
            ptr = p;
		}
        void swap(unique_ptr & u)//交换所指之物
        {
            std::swap(ptr, u.ptr);  //std：标准库  stl：标准模板库
        }
    };
    ///////////////////////第二个版本处理数组类型//////////////////////////////////
    //unique_ptr不但可以指向一个对象，也可以指向一组对象
    template<typename _Tp, typename _Dp>//_Dp是删除器
    class unique_ptr<_Tp[],_Dp>
    {
    private:
        _Tp * ptr;
        unique_ptr(const unique_ptr&) = delete;//禁止掉拷贝构造
        unique_ptr & operator=(const unique_ptr&) = delete;//禁止掉=语句的重载
    public:
        typedef _Tp* pointer;
        typedef _Tp element_type;
        typedef _Dp delete_type;
        
        delete_type get_delete() { return deletype(); }
        
    public:
        unique_ptr():ptr(nullptr) {}
        explicit unique_ptr(_Tp* p): ptr(p) {}
        unique_ptr& operator=(unique_ptr && uq)//移动拷贝函数
		{
    		reset(uq.release());
            return *this;
		}
        unique_ptr(unique_ptr && uq)//转移拷贝构造函数
        {
            ptr = uq.release();
        }
        ~unique_ptr() 
        {
            if(ptr != nullptr)
            {
                get_delete()(ptr);//调动get_delete()得到一个删除器对象，再调动删除器对象的operator()函数删除掉ptr
			}
            ptr = nullptr;
        }
        pointer get()const {return ptr;} //获得当前智能指针所指之物的地址
        _Tp & operator*()const {return *get(); }
        
        //新加的重载[]
        _Tp& operator[](size_t index)
        {
            return ptr[index];
        }
        
        pointer operator->()const {return get(); }
        
        explicit operator bool() const //重载bool强转，explicit禁止隐式转换
        {
            return get() == nullptr ? false : true;
        }
        pointer release()//释放拥有权
        {
            pointer p = ptr;
            ptr = nullptr;
            return p;
        }
        void reset(pointer p = nullptr)//重设拥有的对象
        {
            get_delete()(ptr);//调动get_delete()得到一个删除器对象，再调动删除器对象的operator()函数删除掉ptr
            ptr = p;
		}
        void swap(unique_ptr & u)//交换所指之物
        {
            std::swap(ptr, u.ptr);  //std：标准库  stl：标准模板库
        }
    };
};
```

缺省的拷贝构造、缺省的赋值是按位拷贝、按位赋值

> 按位拷贝、按位赋值：抓住首地址一个字节一个字节的拷贝过去，底层用的memmove(&objb, &obja, sizeof(Object))，如下图：（就相当于内存发生了移动）
>
> ![image-20201130182858264](img/C++%EF%BC%9A%E6%99%BA%E8%83%BD%E6%8C%87%E9%92%88.img/image-20201130182858264.png)

### 可以移动构造和移动赋值

unique_ptr 虽然没有支持普通的拷贝和赋值操作，但却提供了一种移动机制来将指针的所有权从一个 unique_ptr 转移给另一个 unique_ptr。  

如果需要转移所有权，可以使用 std::move()函数。  

```cpp
int main()
{
    unique_ptr<int> pInt(new int(5));
    unique_ptr<int> pInt2 = std::move(pInt);//转移所有权
    //cout<<*pInt<<endl;//出错，pInt为空
    cout<<*pInt2<<endl;
    unique_ptr<int> pInt3 = std::move(pInt2);
}
```

unique_str可以使用move来转移所有权的根本原因是：unique_str有移动拷贝和移动赋值函数

移动拷贝：拷贝构造函数参数用两个&&，移动拷贝使得返回局部对象的时候不去创建临时对象，而是直接把局部的那个返回给主函数，移动拷贝减少了临时对象的创建，如下代码：

```cpp
unique_ptr& operator=(unique_ptr && uq)
{
    this->ptr = uq.ptr;
    uq.ptr = nullptr;
    return *this;
}
```

移动拷贝构造函数和移动赋值函数如下：

```cpp
Object(Object && obj) : value(obj.value)//移动拷贝
{
    cout<<"Move Create Object"<<this<<endl;
}
Object & operator=(Object&& obj)//移动赋值	
{
    if(this != &obj)
    {
        value = obj.value;
    }
    cout<<"Move operator = "<<this<<" "<<&obj<<endl;
}
```

两个&&符号称之为右值引用，不是引用的引用，std::move调动的就是移动拷贝函数

普通的构造函数只是建立对象，普通的赋值只是对对象与对象的赋值，而移动拷贝、移动赋值主要针对的是临时的对象，移动拷贝、赋值减少了临时对象的构建

### 左值引用，右值引用

```cpp
int main()
{
    int a = 10;
    int &b = a;//正确，左值引用
    
    //int &c = 10;//错误，右值不可这样引用，因为10在常量区，只能读10的值，不能改变10的值
    int &&c = 10;//正确，右值引用
    const int& x = 10;//正确，常引用
}
```

右值引用和常引用的区别：

```cpp
const int& x = 10;//常引用
/*
底层做的事情如下：
int tmp = 10;
const int &x = tmp;
所以常引用是不能对x进行改变的
*/

int &&c = 10;//右值引用
/*
底层做的事情如下：
int tmp = 10;
int &c = tmp;
所以可以改变c的值
*/
```

### 可以返回 unique_ptr 

unique_ptr 不支持拷贝操作，但却有一个例外：可以从函数中返回一个 unique_ptr。  可以返回的根本原因：unique_ptr有移动拷贝函数，可以直接把临时的给主函数里

```cpp
unique_ptr<int> clone(int a)
{
	unique_ptr<int> pInt(new int(a));
	return pInt; // 返回 unique_ptr
}
int main()
{
	int x = 5;
	unique_ptr<int> reta = clone(x);//正确
	unique_ptr<int> retb;
	retb = clone(x);//正确
    
	cout << *reta << endl;
	cout<< *retb<<endl;
	return 0;
}
```


### unique_ptr使用场景

1. 为动态申请的资源提供异常安全保证  

   我们先来看看下面这一段代码：  

   ```cpp
   void func()
   {
       int *p = new int(5);
       ......
       delete p;
   }
   ```

   这是我们传统的写法：当我们动态申请内存后，有可能我们接下来的代码由于抛出异常或者提前退出（if 语句）而没有执行 delete 操作。
   解决的方法是使用 unique_ptr 来管理动态内存，只要 unique_ptr 指针创建成功，最后其析构函数都会被调用。确保动态资源被释放  

   ```cpp
   void func()
   {
       unique_ptr pInt(new int(5));
       ......
   }
   ```

2. 返回函数内动态申请资源的所有权

   示例如下：

   ```cpp
   unique_ptr<int> func(int x)
   {
       unique_ptr<int> pInt(new int(x));
       return pInt;//返回unique_ptr
   }
   int main()
   {
       int a = 5;
       unique_ptr<int> ret = func(a);
       cout<<*ret<<endl;
       //函数结束，自动释放资源
   }
   ```

3. 在容器中保存指针（与stl联合使用）

   示例如下：

   ```cpp
   int main()
   {
       vector< unique_ptr<int> > vec;
       unique<int> pInt(new int(5));
       vec.push_back(std::move(pInt));//使用移动语义
   }
   ```

4. 管理动态数组

   标准库提供了一个可以管理动态数组的unique_ptr版本

   ```cpp
   int main()
   {
       unique_ptr<int[]> p(new int[5]{1,2,3,4,5});
       p[0] = 0;//unique_ptr重载了[]
   }
   ```

# shared_ptr

shared_ptr 是一个引用计数（会记录有多少个shared_ptr指向同一个对象）智能指针，用于共享对象的所有权也就是说它允许多个指针指向同一个对象。这一点与原始指针一致。 

### 使用示例

```cpp
#include <iostream>
#include <memory>
using namespace std;

class Object
{};

int main()
{
	shared_ptr<Object> pObj(new Object(20));
	cout << (*pObj).Value() << endl;
	cout << "pObj的引用计数：" << pObj.use_count() << endl<<endl;

	shared_ptr<Object> pObj2 = pObj;
	cout << (*pObj2).Value() << endl;
	cout << "pObj的引用计数：" << pObj.use_count() << endl;
	cout << "pObj2的引用计数：" << pObj2.use_count() << endl<<endl;
}
```

 <img src="img/C++%EF%BC%9A%E6%99%BA%E8%83%BD%E6%8C%87%E9%92%88.img/image-20201210082805566.png" alt="image-20201210082805566" style="zoom:50%;" />

从上面这段代码中，我们对 shared_ptr 指针有了一些直观的了解。
一方面，跟 STL 中大多数容器类型一样， shared_ptr 也是模板类，因此在创建 shared_ptr 时需要指定其指向的类型。另一方面， ==正如其名一样， shared_ptr 指针允许让多个该类型的指针共享同一堆分配对象。 同时shared_ptr 使用经典的“引用计数”方法来管理对象资源， 每个 shared_ptr 对象关联一个共享的引用计数==

对于 shared_ptr 在拷贝和赋值时的行为是，每个 shared_ptr 都有一个关联的计数值，通常称为引用计数。无论何时我们拷贝一个 shared_ptr，计数器都会递增。  例如，当用一个 shared_ptr 初始化另一个 shred_ptr，或将它当做参数传递给一个函数以及作为函数的返回值时，它所关联的计数器就会递增  

当我们给 shared_ptr 赋予一个新值或是 shared_ptr 被销毁（例如一个局部的 shared_ptr 离开其作用域）时，计数器就会递减  

==shared_ptr 对象的计数器变为 0，它就会自动释放自己所管理的对象。==

对比我们上面的代码可以看到：当我们将一个指向 Object 对象的指针交给 pObj 管理后， 其关联的引用计数为
1。 接下来，我们用 pObj 初始化 pObjt2， 两者关联的引用计数值增加为 2。 随后，函数结束， pObj 和 PObj2 相继离开函数作用域， 相应的引用计数值分别自减 1 最后变为 0，于是 Object 对象被自动释放（调用其析构函数）

### 创建shared_ptr

最安全和高效的创建shared_ptr的方法是调用 make_shared 库函数，该函数会在堆中分配一个对象并初始化，最后返回指向此对象的share_ptr 实例。如果你不想使用 make_ptr，也可以先明确 new 出一个对象，然后把其原始指针传递给 share_ptr 的构造函数，如下代码：

```cpp
#include <iostream>
#include <string>
#include <memory>
using namespace std;

int main()
{
	shared_ptr<string> pstr = make_shared<string>(10, 'y');
	cout << *pstr << endl << endl;
    
	int *p = new int(10);
	shared_ptr<int> pInt(p);
	cout << *p << endl << endl;
}
```

 <img src="img/C++%EF%BC%9A%E6%99%BA%E8%83%BD%E6%8C%87%E9%92%88.img/image-20201210083943197.png" alt="image-20201210083943197" style="zoom: 67%;" />

### make_shared

C++11 中引入了智能指针, 同时还有一个模板函数 `std::make_shared` 可以返回一个指定类型的 `std::shared_ptr`

make_shared函数的主要功能是在动态内存中分配一个对象并初始化它，返回指向此对象的shared_ptr;由于是通过shared_ptr管理内存，因此是一种安全分配和使用动态内存的方法。

使用示例：

```cpp
//p1指向一个值为"9999999999"的string  
shared_ptr<string> p1 = make_shared<string>(10, '9');    
  
shared_ptr<string> p2 = make_shared<string>("hello");    
  
shared_ptr<string> p3 = make_shared<string>();   
```

从上述例子我们可以看出以下几点： 

 1）make_shared是一个模板函数； 

 2）make_shared模板的使用需要以“显示模板实参”的方式使用，如上题所示make_shared<string>(10, 9),如果不传递显示 模板实参string类型，make_shared无法从(10, '9')两个模板参数中推断出其创建对象类型。 

 3）make_shared在传递参数格式是可变的，==参数传递为生成类型的构造函数参数，因此在创建shared_ptr<T>对象的过程中调用了类型T的某一个构造函数==。

### shared_ptr对象创建方法的讨论

通常有两种方法去初始化一个std::shared_ptr

- 通过它自己的构造函数

  ```cpp
  shared_ptr<Object> pObja(new Object(20));
  ```

- 通过std::make_shared

  ```cpp
  shared_ptr<Object> pobjb = make_shared<Object>(20);
  ```

这两种方法有哪些不同的特性呢？

**1、通过自己的构造函数**

shared_ptr是非侵入式的，即计数器的值并不存储在shared_ptr内，它其实是存储在其他地方——在堆上的。当一个shared_ptr由一块内存的原生指针创建的时候，这个计数器也就随之产生。这个计数器结构的内存会一直存在——直到所有的shared_ptr和weak_ptr都被销毁的时候。这里注意，当所有的shared_ptr都被销毁时，这块内存就已经被释放了，但是可能还有weak_ptr存在，也就是说计数器的销毁有可能发生在内存对象销毁很久后才能发生。如下图：

 <img src="img/C++%EF%BC%9A%E6%99%BA%E8%83%BD%E6%8C%87%E9%92%88.img/image-20201230094916284.png" alt="image-20201230094916284" style="zoom:50%;" />

通过上图，可以知道当创建一个shared_ptr管理一块原生内存（原生内存：代指这个时候还没有其他shared_ptr指向这块内存）时，在堆上实际发生了两次内存分配：一次是给Object分配内存，一次是给引用计数器分配内存

在C++中，内存分配和回收是最慢的单次操作，鉴于此，有一种办法将两次内存分配合二为一，即std::makeshared

**2、通过std::make_shared**

make_shared可以同时为计数器和原生内存分配内存空间，并把二者的内存视作整体管理，如下图：

 <img src="img/C++%EF%BC%9A%E6%99%BA%E8%83%BD%E6%8C%87%E9%92%88.img/image-20201230100101867.png" alt="image-20201230100101867" style="zoom:50%;" />

make_shared只对内存操纵一次，因此make_shared效率要比通过构造函数创建高

**std::make_shared的好处**

- 减少操纵内存的次数

- 增大Cache(高速缓存)局部性，使用makeshared，计数器的内存和Object内存就在堆上排排坐，这样所有要访问这两个内存的操作就会比另一种方案减少一半的cache misses

  > 引入Cache（高速缓存）的理论基础是程序局部性原理，包括时间局部性和空间局部性。即最近被CPU访问的数据，短期内CPU还要访问；被CPU访问的数据附近的数据，CPU短期内还要访问。因此如果将刚刚访问的数据缓存在Cache中，那下次访问时，可以直接从Cache中取，其速度可以得到数量级的提高。
  >
  > CPU要访问的数据在Cache中已经缓存，称为命中(Hit)，否则称为缺失(Miss)

- 执行顺序以及异常安全性，如下代码：

  ```cpp
  struct Object
  {
      int i;
  };
  void doSomething(std::shared_ptr<Object>, double d);
  double couldThrowException();//这个函数可能抛出异常
  int main()
  {
      doSomething(std::shared_ptr<Object>(new Object{1024}), couldThrowException());
      
      return 0;
  }
  ```

  上边的代码中，doSomething函数被调用之前，参数列表有三件事要去完成：构造并给Object分配内存，构造shared_ptr，调用couldThrowException，C++17中引入了更加严格的鉴别函数参数构造顺序的方法，但是在那之前，这三件事构造顺序应该是：构建Object，调用couldThrowException，构造shared_ptr。

  这样的构建顺序，如果couldThrowException抛出了异常，那么第三步就不会发生了，即shared_ptr不会被构建出来，就没有智能指针去管理步骤1开辟的内存，那么智能指针就相当于失效了，因为它压根就没被构建出来

  使用make_shared可以解决上述问题，makeshared让构建Object和构造shared_ptr紧挨在一起，代码如下：执行顺序（函数参数压栈从右向左）先去调用couldThrowException，接着Object和智能指针一起构建

  ```cpp
  int main()
  {
      doSomething(std::make_shared<Object>(10), couldThrowException());
      
      return 0;
  }
  ```

**std::makeshared的坏处**

- make_shared函数必须有权利调用目标类型的构造函数

- 目标内存（Object的内存块）的生存期问题

  即便被shared_ptr管理的目标对象都被释放了，shared_ptr的计数器还会一直持续存在，直到最后一个指向目标内存的weak_ptr被销毁。这个时候，如果我们使用make_shared函数，问题就来了：程序自动地把被管理对象占用的内存和计数器占用的堆上的内存视作一个整体来管理，这就意味着，即使被管理的对象被析构了，空间还在，内存可能并没有归还，因为它在等着所有的weak_ptr都被清除后和计数器所占用的内存一起归还。

  假如目标对象占用的内存比较大，那就意味着这块大内存被无意义地锁了一段时间

   <img src="img/C++%EF%BC%9A%E6%99%BA%E8%83%BD%E6%8C%87%E9%92%88.img/image-20201230104500531.png" alt="image-20201230104500531" style="zoom:50%;" />

  蓝色阴影就是被shared_ptr管理的对象的内存，它在等待着weak_ptr的计数器变为0，和上边引用计数的内存（橙色部分）一起归还

综合make_shared的好处与坏处，创建shared智能指针时，最好使用make_shared，在享受它带来的好处时，也要留意它的缺陷

### 访问所指对象

shared_ptr 的使用方式与普通指针的使用方式类似，既可以使用解引用操作符*获得原始对象进而访问其各个成
员，也可以使用指针访问符->来访问原始对象的各个成员，示例如下：

```cpp
#include <iostream>
#include <string>
#include <memory>
using namespace std;
class Object
{
private:
	int value;
public:
	Object(int x) :value(x)
	{
		cout << "Create Object!" << endl;
	}
	~Object()
	{
		cout << "Destroy Object!" << endl;
	}
	int &Value()
	{
		return value;
	}
	const int &Value()const
	{
		return value;
	}
};

int main()
{
	shared_ptr<Object> pobj(new Object(10));
    
	cout<<(*pobj).Value()<<endl;
	cout << pobj->Value()<<endl;
	
	return 0;
}
```

 <img src="img/C++%EF%BC%9A%E6%99%BA%E8%83%BD%E6%8C%87%E9%92%88.img/image-20201210085443928.png" alt="image-20201210085443928" style="zoom:67%;" />

### 拷贝和赋值操作

==拷贝==：我们可以用一个 shared_ptr 对象来初始化另一个 share_ptr 实例，该操作会增加其引用计数值，示例如下：

```cpp
int main()
{
	shared_ptr<string> pStr = make_shared<string>(10, 'a');
	cout << pStr.use_count() << endl;  //1

	shared_ptr<string> pStr2(pStr);
	cout << pStr.use_count() << endl;  //2
	cout << pStr2.use_count() << endl; //2
}
```

 <img src="img/C++%EF%BC%9A%E6%99%BA%E8%83%BD%E6%8C%87%E9%92%88.img/image-20201210085836676.png" alt="image-20201210085836676" style="zoom:67%;" />

==赋值==：如果 shared_ptr 实例 p 和另一个 shared_ptr 实例 q 所指向的类型相同或者可以相互转换，我们还可以进行诸如 p = q 这样赋值操作。该操作不会递减 p 的引用计数值，不会递增 q 的引用计数值，示例如下：

```cpp
int main()
{
	shared_ptr<Object> p = make_shared<Object>("a object");
	shared_ptr<Object> q = make_shared<Object>("b object");

	cout << p.use_count() << endl;
	cout << q.use_count() << endl;

	p = q;
	cout << endl << endl;

	cout << p.use_count() << endl;
	cout << q.use_count() << endl;
	
	return 0;
}
```

 <img src="img/C++%EF%BC%9A%E6%99%BA%E8%83%BD%E6%8C%87%E9%92%88.img/image-20201210092133274.png" alt="image-20201210092133274" style="zoom:67%;" />

### 检查引用计数  

shared_ptr 提供了两个函数来检查其共享的引用计数值，分别是 ==unique()==和 ==use_count()==

在前面，我们已经多次使用过 use_count()函数，该函数返回当前指针的引用计数值。 值得注意的是 use_count()函数可能效率很低，应该只把它用于测试或调试。

unique()函数用来测试该 shared_ptr 是否是原始指针唯一拥有者， 也就是 use_count()的返回值为 1 时返回 true，否则返回 false  

### 仿函数

重载()--仿函数

调动方式像函数，但不是函数

```cpp
template<class T>
struct Add
{
  	T operator()(const T &a, const T &b)//重载()符号, 仿函数
  	{
     	return a+b; 
	}
};
int main()
{
    Add<int> add;
    int x = Add<int>()(12,23);
    int x = add(12,23);//add是对象名，add
    /*上面这句就等价于：
    int x = add.operator()(12,23); //operator()当函数名字使用
    进一步：int x = operator(&add, 12, 23);
    */
    
    int x = Add<int>()(12,23);
    //Add是类型名，加括号是构造函数，因此这句先构建一个Add类型的无名对象，再调用该对象的operator()函数，即x = Add<int>().operator()(12,23);   
}
```

### 仿写 shared_ptr

```cpp
#include <memory>
#include <atomic>
#include <iostream>
using namespace std;

template <typename T>  //引用计数类
class RefCnt  
{
private:
	T* mPtr;
	std::atomic<int> mCnt; //原子操作
public:
	RefCnt(T* ptr = nullptr) : mPtr(ptr),mCnt(1)
	{
	}
    int load()const {return mCnt.load();}//获得当前原子对象原子值(mCnt)，瞬间值
    void reset(int ref=1){mCnt = ref;}  //重置引用计数(mCnt)的值
	void addRef() { ++mCnt; }  //引用计数加1
	int delRef() { return --mCnt; }  //引用计数-1
	~RefCnt() {}
};

template <typename T>  //删除器
class MyDeletor  
{
public:
	void operator()(T* ptr) const  //重载()--仿函数
	{
        if(ptr == nullptr)	return;
		delete ptr; // 释放单个资源
	};
};

template <typename T, typename Deletor = MyDeletor<T> >  //Deletor模板类默认类型是MyDeletor<T>
class Shared_ptr
{
public:
    Deletor get_deletor()//获取删除器对象
    {
        return Deletor();
    }
	Shared_ptr(T* ptr = nullptr) : mPtr(ptr)//构造函数
	{
		mpRefCnt = new RefCnt<T>(ptr);
	}
	~Shared_ptr()//析构函数
	{
		if (0 == mpRefCnt->delRef())//如果引用计数-1等于0，说明没有人再引用这个对象
		{
            get_deletor()(mPtr);
            delete mpRefCnt;
        }
        mpRefCnt = nullptr;
        mPtr=nullptr;	
	}
    shared_ptr(const shared_ptr & _Y):mptr(_Y.mptr),mpRefCnt(_Y.mpRefCnt)//拷贝构造函数
    {
            mpRefCnt->addRef();//引用计数+1
	}
	Shared_ptr& operator=(const Shared_ptr& _Y)
	{
     /*
     	赋值语句要处理的情况有：
     	智能指针b赋值给智能指针a： a = b;
     	a.obj = b.obj //a.obj表示a里原来有对象
     	a.obj = b.null
     	a.null = b.obj
     	a.null = b.null
     	解决方案：认为空指针也是一种对象，空也是一个物体，智能指针指向空对象也要有引用计数，因此统一认为是a.obj=b.obj
     */
		 if(this!= &_Y)//防止自赋值
         {
             if(0 == mpRefCnt->delRef())//自己(X)本来持有的对象计数器-1，若引用计数为0了，则删除掉原本持有的对象
             {
                 get_deletor()(mPtr);
                 delete mpRefCnt;
                 mPtr = nullptr;
                 mpRefCnt = nullptr;
             }
             mPtr = _Y.mPtr;
             mpRefCnt = _Y.mpRefCnt;
             mpRefCnt->addRef();
         }
	}
    long use_count() const {return mpRefCnt->load();}
    T *get() const {return mPtr;}
    T & operator* ()const {return *get();}
    T * operator->()const {return get();}
    void reset(T * p=nullptr)//用p重构本智能指针
    {
        if(0 == mpRefCnt->delRef())
        {
            get_deletor()(mPtr);
            delete mpRefCnt;
        }
        mPtr = p;
        mpRefCnt = new mpRefCnt(mPtr);
    }
    operator bool() const {return get()!=nullptr;}//重载bool类型 if(pObjb) {....}
    bool unique()const //是否唯一占有，唯一占有返回true
    {
        return use_count() == 1;
	}
private:
	T* mPtr;
	RefCnt<T>* mpRefCnt; // 面试：为什么定义为指针，如果定义成对象呢：RefCnt<T> mpRefCnt;
    					 // 答：如果定义成对象，就失去了共享这一特性，定义成对象形式，有几个sharedptr就有几个mpRefCnt，各自在维护各自的计数，引用计数就不共享了
};

int main()
{
	Shared_ptr<int> p1(new int(10));
	Shared_ptr<int> p2 = p1;
	Shared_ptr<int> p3;
	p3 = p1;
	*p1 = 20;
	cout << *p2 << " " << *p3 << endl;
	return 0;
}
```

注意：

- 删除(delete)空指针在任何情况下都是安全的，可以多次delete空指针

- 把空指针也看成一个对象，即对象可以为nullptr，这样很多问题就迎刃而解了

### shared_ptr 的线程安全性  

- (shared_ptr）的引用计数本身是线程安全（引用计数是原子操作）  
- 多个线程同时读同一个 shared_ptr 对象是线程安全的  
- 如果是多个线程对同一个 shared_ptr 对象进行读和写，则需要加锁  
- 多线程读写 shared_ptr 所指向的同一个对象，不管是相同的 shared_ptr 对象，还是不同的 shared_ptr 对象，也需要加锁保护  

具体分析一下为什么？

答：“因为 shared_ptr 有两个数据成员，读写操作不能原子化”使得多线程读写同一个 shared_ptr 对象需要加锁  

**shared_ptr的数据结构**

shared_ptr 是引用计数型（reference counting）智能指针，几乎所有的实现都采用在堆（heap）上放个计数值
（count）  

![image-20201210093414818](img/C++%EF%BC%9A%E6%99%BA%E8%83%BD%E6%8C%87%E9%92%88.img/image-20201210093414818.png)

**示例1**：

```cpp
class Object
{
private:
	int value;
public:
	Object(int x = 0) :value(x) {}
	~Object() {}
	int& Value() { return value; }
	const int& Value() const { return value; }
};

int main()
{
	shared_ptr<Object> apa(new Object(10));
	shared_ptr<Object> apb = apa;
	return 0;
}
```

![image-20201210093602548](img/C++%EF%BC%9A%E6%99%BA%E8%83%BD%E6%8C%87%E9%92%88.img/image-20201210093602548.png)

**示例2**：

有 3 个 shared_ptr<Object> 对象 apa、 gx、 apb;  

```cpp
shared_ptr<Object> gx(new Object(1)); // 线程之间共享的 shared_ptr 对象
shared_ptr<Object> apa; // 线程 A 的局部变量
shared_ptr<Objcet> apb(new Object(2)); // 线程 B 的局部变量
```

一开始，各安其事：  

![image-20201210094019169](img/C++%EF%BC%9A%E6%99%BA%E8%83%BD%E6%8C%87%E9%92%88.img/image-20201210094019169.png)

线程 A 执行 apa = gx; （即 read gx），以下完成了步骤 1，还没来及执行步骤 2。这时切换到了 B 线程。  

![image-20201210094208240](img/C++%EF%BC%9A%E6%99%BA%E8%83%BD%E6%8C%87%E9%92%88.img/image-20201210094208240.png)

同时编程 B 执行 gx = apb; （即 write gx），两个步骤一起完成了。先是步骤 1：  

![image-20201210094232838](img/C++%EF%BC%9A%E6%99%BA%E8%83%BD%E6%8C%87%E9%92%88.img/image-20201210094232838.png)

再是步骤 2：  

![image-20201210094245937](img/C++%EF%BC%9A%E6%99%BA%E8%83%BD%E6%8C%87%E9%92%88.img/image-20201210094245937.png)

这时 Object 1 对象已经销毁， apa.mPtr 成了空悬指针！
最后回到线程 A，完成步骤 2：  

![image-20201210094304840](img/C++%EF%BC%9A%E6%99%BA%E8%83%BD%E6%8C%87%E9%92%88.img/image-20201210094304840.png)

总结：多线程无保护地读写 gx，造成了“apa 是空悬指针”的后果。这正是多线程读写同一个 shared_ptr 对象必须加锁的原因。  

**其他问题**

如果把 shared_ptr 放到 unordered_set 中，或者用于 unordered_map 的 key，那么要小心 hash table 退化为
链表。 但是==其 hash_value 是 shared_ptr 隐式转换为 bool 的结果==。也就是说，如果不自定义 hash 函数，那么
unordered_{set/map} 会退化为链表  

为什么要尽量使用 make_shared()？ 申请被管理对象以及引用计数的内存；调用适当的构造函数初始化对象；返回一个 shared_ptr。

为了节省一次内存分配，原来 shared_ptr<Object> x(new Object（10） ); 需要为 Object 对象 和 RefCnt 各分配一次内存，现在用 make_shared() 的话，可以一次分配一块足够大的内存，供 Object 和 RefCnt 对象容身。不过Object 的构造函数所需的参数要传给 make_shared()，后者再传给 Object：： Object()，这只有在 C++11 里通过perfect forwarding(完美转发) 才能完美解决。  

### 环形/循环引用问题

我们知道 shared_ptr 是采用引用计数的智能指针，多个 shared_ptr 实例可以指向同一个动态对象，并维护了一个共享的引用计数器。

对于引用计数法实现的计数，总是避免不了循环引用（或环形引用）的问题， shared_ptr 也不例外  

示例：

```cpp
#include <iostream>
#include <memory>
using namespace std;
class Child;

class Parent
{
public:
	shared_ptr<Child> child;
	~Parent() { cout << "Bye Parent" << endl; }
	void hi() const { cout << "Hello" << endl; }
};
class Child
{
public:
	shared_ptr<Parent> parent;
	~Child() { cout << "Bye Child" << endl; }
};
int main()
{
	shared_ptr<Parent> parent = make_shared < Parent > ();
	shared_ptr<Child> child = make_shared<Child>();
	parent->child = child;
	child->parent = parent;
	child->parent->hi();

	return 0;
}
```

 <img src="img/C++%EF%BC%9A%E6%99%BA%E8%83%BD%E6%8C%87%E9%92%88.img/image-20201210100729934.png" alt="image-20201210100729934" style="zoom:67%;" />

上面代码的运行结果，只打印出” Hello”，而并没有打印出"Bye Parent"或"Bye Child"，说明 Parent 和 Child 的析构函数并没有调用到。这是因为 Parent 和 Child 对象内部，具有各自指向对方的 shared_ptr，加上 parent 和 child 这两个shared_ptr，说明每个对象的引用计数都是 2。当程序退出时，即使 parent 和 child 被销毁，也仅仅是导致引用计数变为了 1，因此并未销毁 Parent 和 Child 对象。  

![image-20201210100848671](img/C++%EF%BC%9A%E6%99%BA%E8%83%BD%E6%8C%87%E9%92%88.img/image-20201210100848671.png)

```cpp
parent->child = child;
child->parent = parent;
```

![image-20201210100904939](img/C++%EF%BC%9A%E6%99%BA%E8%83%BD%E6%8C%87%E9%92%88.img/image-20201210100904939.png)

为了解决类似这样的问题， C++11 引入了 weak_ptr，来打破这种循环引用。  

# weak_ptr

### weak_ptr是什么

weak_ptr 是为了配合 shared_ptr 而引入的一种智能指针，它指向一个由 shared_ptr 管理的对象而不影响所指对
象的生命周期，也就是将一个 weak_ptr 绑定到一个 shared_ptr 不会改变 shared_ptr 的引用计数。

不论是否有 weak_ptr 指向，一旦最后一个指向对象的 shared_ptr 被销毁，对象就会被释放。从这个角度看，
weak_ptr 更像是 shared_ptr 的一个助手而不是智能指针。  

weak_ptr并不拥有动态对象的管辖权，weak_ptr指向shared_ptr的目标也不会增加计数器的值，weak_ptr拥有一套不纳入计数器的指针系统

### 创建weak_ptr

当我们创建一个 weak_ptr 时，需要用一个 shared_ptr 实例来初始化 weak_ptr，由于是弱共享， weak_ptr 的创建并不会影响 shared_ptr 的引用计数值。  

示例：

```cpp
int main()
{
    shared_ptr<int> sp(new int(5));
    cout<<"创建前sp的引用计数："<<sp.use_count()<<endl;//use_count==1
    
    weak_ptr<int> wp(sp);
    cout<<"创建后sp的引用计数："<<sp.use_count()<<endl;//use_count==1
}
```

 <img src="img/C++%EF%BC%9A%E6%99%BA%E8%83%BD%E6%8C%87%E9%92%88.img/image-20201210102057369.png" alt="image-20201210102057369" style="zoom:67%;" />

### 判断指向的对象是否存在  

既然 weak_ptr 并不改变其所共享的 shared_ptr 实例的引用计数，那就可能存在 weak_ptr 指向的对象被释放掉这种情况。这时，我们就不能使用 weak_ptr 直接访问对象。那么我们如何判断 weak_ptr 指向对象是否存在呢？

C++中提供了 lock 函数来实现该功能。
如果对象存在， lock()函数返回一个指向共享对象的 shared_ptr，否则返回一个空 shared_ptr    

weak_ptr 还提供了 expired()函数来判断所指对象是否已经被销毁，弱指针指向的对象不存在，expired函数返回1

 <img src="img/C++%EF%BC%9A%E6%99%BA%E8%83%BD%E6%8C%87%E9%92%88.img/image-20201224221309885.png" alt="image-20201224221309885" style="zoom:50%;"/>

```cpp
class Object
{
private:
	int value;
public:
	Object(int x) :value(x) { cout << "create object" << endl; }
	~Object() { cout << "Destroy object" << endl; }
	int GetValue() const { return value; }
};

int main()
{
	shared_ptr<Object> sp(new Object(10));
	weak_ptr<Object> wp(sp);
	// sp.reset(); 
	if (shared_ptr<Object> pa = wp.lock())
	{
		cout<< pa->GetValue() << endl;
	}
	else
	{ //weak_ptr 还提供了 expired()函数来判断所指对象是否已经被销毁
		cout<<wp.expired()<<endl;//指向的对象不存在，expired函数返回1
		cout << "wp 引用的对象为空" << endl;
	}
	return 0;
}
```

 没有reset，weak_ptr指向的对象存在，执行if的运行结果：

 <img src="img/C++%EF%BC%9A%E6%99%BA%E8%83%BD%E6%8C%87%E9%92%88.img/image-20201210102627636.png" alt="image-20201210102627636" style="zoom:67%;" />

 打开reset，weak_ptr指向的对象不存在，执行else的执行结果：

 <img src="img/C++%EF%BC%9A%E6%99%BA%E8%83%BD%E6%8C%87%E9%92%88.img/image-20201224221915768.png" alt="image-20201224221915768" style="zoom:60%;" />

由此可见，weak_ptr就像一个观察器，它在观察shared_ptr指向的对象是否存在，如果引用的对象存在，则可以通过观察器使用到共享指针指向的对象，如果共享指针引用的对象已经消失，那么通过观察器就没办法再使用这个对象

### 使用 weak_ptr  

weak_ptr 并没有重载 operator->和 operator *操作符，因此不可直接通过 weak_ptr 使用对象，典型的用法是
调用其 lock 函数来获得 shared_ptr 示例，进而访问原始对象  

```cpp
shared_ptr<Object> sp(new Object(10));
weak_ptr<Object> wp(sp);
shared_ptr<Object> pa = wp.lock();
```

### 打破循环引用

最后，我们来看看如何使用 weak_ptr 来改造最前面的代码，打破循环引用问题。 

==weak_ptr之所以可以打破循环引用，是因为：将一个 weak_ptr 绑定到一个 shared_ptr 不会改变 shared_ptr 的引用计数==

图解如下：把Parent类与Child类内的智能指针成员改为weak_ptr类型即可 ![image-20201224225750241](img/C++%EF%BC%9A%E6%99%BA%E8%83%BD%E6%8C%87%E9%92%88.img/image-20201224225750241.png)

代码如下：

```cpp
class Child;
class Parent
{
public:
	weak_ptr<Child> child;
	~Parent() { cout << "Bye Parent" << endl; }
	void hi()const { cout << "Hello" << endl; }
};
class Child
{
public:
	weak_ptr<Parent> parent;
	~Child() { cout << "Bye Child" << endl; }
};
int main()
{
	shared_ptr<Parent> parent = make_shared<Parent>();
	shared_ptr<Child> child = make_shared<Child>();
	parent->child = child;
	child->parent = parent;
	//child->parent->hi(); child->parent是weak_ptr，weak_ptr没有重载->运算符，所以不能这样输出，要用lock返回shared_ptr类型才能用->
	child->parent.lock()->hi();

	return 0;
}
```

运行结果如下：Parent 和 Child 对象析构函数如愿被调用

![image-20201224224453255](img/C++%EF%BC%9A%E6%99%BA%E8%83%BD%E6%8C%87%E9%92%88.img/image-20201224224453255.png)

### 非heap资源时的释放问题

删除器：

```cpp
// 对象删除器
struct Close_File
{
	Close_File() {}
	void operator()(FILE * ptr) const
	{
		fclose(ptr);
	}
};
int main()
{
	int a = 10 , b = 20;
	unique_ptr<FILE,Close_File> uq_file(fopen("tulun.txt","w"));
	if(!uq_file)
	{
		cout<<"fopen file failure "<<endl;
		exit(EXIT_FAILURE);
	}
	fprintf(uq_file.get(),"a = %d b = %d \n",a,b);
	return 0;
}
```

示例2：

```cpp
class My_Write_File 
{
public:
	My_Write_File(string file) 
    {
		fp = fopen(file.c_str(), "w");
	}
	~My_Write_File() 
    {
		printf("MyFile close\n");
		fclose(fp);
	}
private:
	FILE *fp;
};
int main()
{
	std::shared_ptr<My_Write_File> fp(new My_Write_File("test.txt"));
	//...
	return 0;
}
```

# 附录：

### 给 shared_ptr 添加自定义删除器的几种方式  

```cpp
#include<memory>
#include<iostream>
using namespace std;

class Object
{
private:
	int value;
public:
	explicit Object(int x = 0) :value(x) { cout << "Constructor Object" << endl; }
	~Object() { cout << "Destructor Object" << endl; }
	int& Value() { return value; }
	const int& Value() const { return value; }
};

void deleter(Object* p)
{
	cout << "function called Deleter " << endl;
	delete[]p;
}
struct Deleter
{
	void operator()(Object* p)
	{
		cout << "function Object Deleter " << endl;
		delete[]p;
	}
};

int main()
{
	//shared_ptr<Object> sp_a(new Object[5]);
	shared_ptr<Object> sp_b(new Object[5], deleter);
	shared_ptr<Object> sp_c(new Object[5], Deleter());
	shared_ptr<Object> sp_d(new Object[5], [](Object* p) 
    {
		cout << "lambda Deleter " << endl;
		delete[]p;
	});
	shared_ptr<Object> sp_e(new Object[5], default_delete<Object[]>());
}
```

