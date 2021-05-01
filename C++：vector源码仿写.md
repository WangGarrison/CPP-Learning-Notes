## vector源码仿写

这些代码是老杨仿写自标准模板库的vector，但是与stl中的有差异，去掉了一些现阶段没学的东西

### 预备知识

>预备知识：
>引用不但可以是整型类型的别名，也可以是指针类型的别名，如下代码：
>
>```cpp
>#include <iostream>
>using namespace std;
>
>namespace wang
>{
>	template<class T>
>	void Swap(T &a, T&b)
>	{
>		T c = a;
>		a = b;
>		b = c;
>	}
>}
>int main()
>{
>	int x = 10, y = 20;
>	int *xp = &x;
>	int *yp = &y;
>	wang::Swap(xp, yp);
>	cout << *xp << endl;
>	cout << *yp << endl;
>	return 0;
>}
>```
>![在这里插入图片描述](img/C++%EF%BC%9Avector%E8%80%81%E6%9D%A8%E4%BB%A3%E7%A0%81.img/20201110194124487.png)
>能交换成功，该段代码里，Swap函数实参给的是int\*类型，所以T就推演出是int\*类型，所有引用是指针类型的引用
>![在这里插入图片描述](img/C++%EF%BC%9Avector%E8%80%81%E6%9D%A8%E4%BB%A3%E7%A0%81.img/20201110194509307.png)
>再入对于如下代码：
>```cpp
>int * & r = p;
>```
>从右向左识别，r是一个引用，整型指针的引用
### 填充内存块函数
>在第一个参数指向的数组中构造n个元素，将它们初始化为x值
>```cpp
>#include <iostream>
>using namespace std;
>namespace wang
>{
>	template <class _FI, class Size, class T>
>	_FI uninitialized_fill_n(_FI first, Size n, const T & x)
>	{
>		for (; n--; ++first)
>		{
>			new(&*first) T(x);
>		}
>		return first;
>	}
>}
>int main()
>{
>	int n = 10;
>	int *p = (int *)malloc(sizeof(int)*n);
>	wang::uninitialized_fill_n(p, 10, 23);//给p指向的内存填充10个23
>	return 0;
>}
>```
>![在这里插入图片描述](img/C++%EF%BC%9Avector%E8%80%81%E6%9D%A8%E4%BB%A3%E7%A0%81.img/20201110200840787.png)
>若是想要给一个大小为10的数组ar，前五个位置填充12，后五个位置填充23，则可以这样调用填充函数
>```cpp
>int *p = wang::uninitalized_fill_n(ar,5,12);
>wang::uninitalozed_fill_n(p,5,23);
>```
>或者直接这样调用：
>```cpp
>wang::uninitalozed_fill_n(wang::uninitalozed_fill_n(ar,5,12),5,23);
>```
### 复制内存块函数
>将范围first到last中元素的副本构造到从result开始的范围，并返回目标(result)范围中最后一个元素的迭代器
>```cpp
>//wang的命名空间里
>template<class _II, class _FI>
>_FI uninitialized_copy(_II first, _II last, _FI result)
>{
>	for (; first != last; ++result, ++first;)
>	{
>		*result = *first;//目前只针对内置类型
>		//new(& *result) T(*first)
>	}
>	return result;
>}
>```
>说明：现阶段无法根据result迭代器逆推出它迭代的类型T，所以不能使用定位new去构建对象，以后学了类型的淬取就可以根据迭代器的类型去淬取出它迭代的类型，就可以使用定位new了，再回来补充
>复制过程如下图：
>![在这里插入图片描述](img/C++%EF%BC%9Avector%E8%80%81%E6%9D%A8%E4%BB%A3%E7%A0%81.img/20201110210022477.png)
### 对象与对象的拷贝
>对象与对象的拷贝不同于把对象复制到内存块，对象与对象之间可以之间赋值
>```cpp
>//wang的命名空间里
>template<class _II,class _OUTI>
>_OUTI copy(_II first, _II last, _OUTI result)
>{
>	for(;first != last; ++result, ++first)
>	{
>		*result = *first;
>	}
>	return result;
>}
>```
### 简单填充函数
>空间已经有对象，把val填充到first到last范围内
>```cpp
>template<class _FI, class T>
>void fill(_FI first, _FI last, const T & val)
>{
>	while (first != last)
>	{
>		*first = val;
>		++first;
>	}
>}
>```
### 向后拷贝函数
>将范围first到last中的元素从结尾开始复制到result终止的范围中，函数返回目标范围中第一个元素的迭代器
>```cpp
>template<class _BI,class _BI2>
>_BI2 copy_backward(_BI first, _BI last, _BI2 result)
>{
>	while (last != first)
>	{
>		*(--result) = *(--last);
>	}
>	return result;
>}
>```
>copy_backward拷贝过程示意图如下![在这里插入图片描述](img/C++%EF%BC%9Avector%E8%80%81%E6%9D%A8%E4%BB%A3%E7%A0%81.img/2020111021403355.png)
### 逆置拷贝函数
>反序复制范围，将first到last中的元素复制到result开始的范围，但顺序相反
>```cpp
>template <class _BI, class _OUTI>
>_OUTI reverse_copy(_BI first, _BI last, _OUTI result)
>{
>	while(first != last)
>	{
>		--last;
>		*result = *last;
>		++result;
>	}
>	return result;
>}
>```
>![在这里插入图片描述](img/C++%EF%BC%9Avector%E8%80%81%E6%9D%A8%E4%BB%A3%E7%A0%81.img/20201110221037160.png)
### 构建与销毁函数
>```cpp
>template <class _T>
>inline void construct(_T *p)
>{
>	new (p) _T();//在p位置构造一个_T类型对象
>}
>
>template <class _T1, class _T2)
>inline void construct(_T1* p, const _T2 & value)
>{
>	new(p) _T1(value);
>}
>
>template <class _Tp>
>inline void destory(_Tp * pointer)
>{
>	pointer->~_Tp();
>}
>
>template <class _FI>
>inline void destory(_FI first, _FI last)//销毁first到last范围的对象
>{
>	for(;first != last; ++first)
>	{
>		destory(&*first);
>	}
>}
>```

### 返回距离函数
>```cpp
>template <class _RAI>
>inline size_t distance(_RAI _F, _RAI _L)
>{
>	return _L-_F;
>}
>```

### vector基类
>向量基类有两个目的。首先，它的申请空间函数和释放空间函数分配存储（但不创建对象）。这使得异常安全更容易
>其次，基类封装了SGI样式分配器和标准一致性分配器之间的所有差异
>```cpp
>template <class _Tp> 
>class Vector_base 
>{
>public:
>	Vector_base(): _M_start(0), _M_finish(0), _M_end_of_storage(0) {}
>	Vector_base(size_t n): _M_start(0), _M_finish(0), _M_end_of_storage(0)  
>	{
>		_M_start = _M_allocate(n);//申请n个空间
>		_M_finish = _M_start;
>		_M_end_of_storage = _M_start + n;
>	}
>	~Vector_base() 
>	{ 
>		_M_deallocate(_M_start, _M_end_of_storage - _M_start);
>	}
>	
>protected:
>	_Tp* _M_start;
>	_Tp* _M_finish;
>	_Tp* _M_end_of_storage;
>	
>	_Tp* _M_allocate(size_t n)//申请空间
>	{ 
>		return (_Tp*)malloc(sizeof(_Tp)*n);
>	}
>	void _M_deallocate(_Tp* p, size_t n) // 释放空间，不需要n，因为malloc申请空间时首地址头部会记录空间大小的信息，free只要一个参数就可以了
>	{
>		free(p);
>	}
>};
>```
>_M_start，_M_finish，_M_end_of_storage示意图如下：
>![在这里插入图片描述](img/C++%EF%BC%9Avector%E8%80%81%E6%9D%A8%E4%BB%A3%E7%A0%81.img/20201110223827283.png)

### vector类

```cpp
template <class _Tp>
	class vector : protected Vector_base<_Tp>//保护继承于向量基类
	{
	private:
		typedef Vector_base<_Tp> _Base;
	public:
		typedef _Tp					value_type;//重命名元素类型
		typedef value_type*			pointer;//重命名指针类型
		typedef const value_type*	const_pointer;//重命名常指针
		typedef value_type*			iterator; //重命名迭代器
		typedef const value_type*	const_iterator;
		typedef value_type&			reference;//重命名引用
		typedef const value_type&	const_reference;
		typedef size_t				size_type;
		typedef int					difference_type;
	public:
		// 迭代器
		iterator begin() { return _M_start; }//返回指向起始的迭代器
		const_iterator begin() const { return _M_start; }

		iterator end() { return _M_finish; }//返回指向元素末尾的迭代器
		const_iterator end() const { return _M_finish; }

		//  元素个数
		size_type size() const
		{
			return size_type(end() - begin());//
		}
		// 可容纳的最大元素个数
		size_type max_size() const
		{
			return size_type(-1) / sizeof(_Tp);
		}
		// 容量
		size_type capacity() const
		{
			return size_type(_M_end_of_storage - begin());
		}
		// 判空
		bool empty() const
		{
			return begin() == end();
		}

		// 元素访问
		reference operator[](size_type n) { return *(begin() + n); }//arr[n]
		const_reference operator[](size_type n) const { return *(begin() + n); }

		//访问指定的元素
		reference at(const size_type pos)
		{
			if ((size_type)(_M_finish - _M_start) <= pos)//越界检查
			{
				_Xrange();//越界
			}
			return *(begin() + n);
		}
		//访问指定位置的元素
		const_reference at(const size_type pos) const
		{
			if ((size_type)(_M_finish - _M_start) <= pos)//越界检查
			{
				_Xrange();
			}
			return *(begin() + n);
		}
		//访问第一个元素
		reference front() { return *begin(); }
		const_reference front() const { return *begin(); }

		//访问最后一个元素
		reference back() { return *(end() - 1); }
		const_reference back() const { return *(end() - 1); }

		// 数据访问，返回指向内存数组中第一个元素的指针
		pointer data() { return _M_start; }
		const_pointer data() const { return _M_start; }
```

## 以下都是vector类内的

vector_base申请空间和释放空间，vector就不参与管理空间之类的，专注于对象，责任分离了

### vector构造函数

```cpp
vector():_Base(){}

vector(size_type n, const _Tp &value):_Base(n)
{
    _M_finish = uninitialized_fill_n(_M_start,n,value);//在第一个参数指向的空间里构造n个对象值为第三个参数
}

vector(size_type n):_Base(n)
{
    _M_finish = uninitialized_fill_n(_M_start,n, _Tp());//在第一个参数指向的空间里构造n个对象值为第三个参数
}

//拷贝构造函数
vector(cosnt vector<_Tp> &x):_Base(x.size())//基类构造函数参数是开辟空间的大小
{
    _M_finish = uninitialized_copy(x.begin(),x.end(),_M_start);//将第一个参数到第二个参数范围的元素的副本构造到从第三个参数开始的范围，并返回目标范围中最后一个元素的迭代器
}

//构造函数——范围拷贝
vector(const _Tp * _F, const _Tp *_L):Base(_F-_L)//将_F到_L范围的元素拷贝到该vector里
{
    _M_finish = uninitialized_copy(_F, _L, _M_start);//将第一个参数到第二个参数范围的元素的副本构造到从第三个参数开始的范围，并返回目标范围中最后一个元素的迭代器
}

//析构函数
~vector()
{
    destory(begin(), end());
}
```

//内置类型从语义上讲有构造函数，从实际上讲并没有构造函数，内置类型有个伪构造函数，int()缺省构造值为0

![image-20201117111632883](img/C++%EF%BC%9ATL_vector%E6%BA%90%E7%A0%81.img/image-20201117111632883.png)

### 重载=

veca=vecb可能有以下几种情况：

1.空=空

![image-20201117113319340](img/C++%EF%BC%9ATL_vector%E6%BA%90%E7%A0%81.img/image-20201117113319340.png)

2.空=不空

![image-20201117113333227](img/C++%EF%BC%9ATL_vector%E6%BA%90%E7%A0%81.img/image-20201117113333227.png)

3.不空=空

![image-20201117113346172](img/C++%EF%BC%9ATL_vector%E6%BA%90%E7%A0%81.img/image-20201117113346172.png)

4.元素少、容量少=元素多、容量多

![image-20201117113407399](img/C++%EF%BC%9ATL_vector%E6%BA%90%E7%A0%81.img/image-20201117113407399.png)

5.元素多、容量少=元素少、容量多

![image-20201117113445480](img/C++%EF%BC%9ATL_vector%E6%BA%90%E7%A0%81.img/image-20201117113445480.png)

6.元素少、容量多=元素多、容量多

![image-20201117113515089](img/C++%EF%BC%9ATL_vector%E6%BA%90%E7%A0%81.img/image-20201117113515089.png)

为实现以上多种情况，解决的主逻辑如下：

![image-20201117140714652](img/C++%EF%BC%9ATL_vector%E6%BA%90%E7%A0%81.img/image-20201117140714652.png)

```cpp
vector<_Tp> & operator=(const vector<_Tp> & x)
{
    if(this != &x)//避免自己给自己赋值
    {
        const size_type xlen = x.size();
        if(capacity() < xlen)  //左边空间容量小于右边元素个数
        {
            iterator tmp = M_alloc_and_copy(xlen, x.begin(), x.end());//申请xlen个长度的空间，把第二个参数到第三个参数范围之间的元素拷贝进去，返回首地址
            destory(_M_start, _M_finish);//析构左边空间的元素
            _M_deallocate(_M_start, _M_end_of_storage - _M_start);//释放左边空间
            _M_start = tmp;
            _M_end_of_storage = _M_start + xlen;
        }
        else if(size() >= xlen)//capacity()>=xlen 且左边的元素个数大于右边的元素个数
        {
            iterator i = copy(x.begin(), x.end(), begin());//把右边的拷到左边的空间
            destory(i,_M_finish);//析构掉左边多了的对象
        }
        else//capacity()>=xlen, 且左边的元素个数小于右边的元素个数
        {
            //先把右边的一部分直接拷贝给左边的对象，再把右边剩余的对象构建到左边接下来的空间
            copy(x.begin(),x.begin() + dize(), _M_start);//参数1与参数2之间的元素拷贝到参数3开始的位置
            uninitialized_copy(x.begin(), x.end(), _M_finish);//参数1到参数2之间的元素构建到参数3开始的位置
                
        }
        _M_finish = _M_start + xlen;
    }
    return *this;
    
}
```

以上代码也解决了空对空这样的特殊情况

### 申请空间并拷贝

```cpp
//申请n长度的空间，并把_F到_L范围的元素拷贝进去，返回首地址
iterator M_alloc_and_copy(size_type n, const_iterator _F, const_iterator _L)
{
    iterator result = _M_allocate(n);//申请空间
    uninitialized_copy(_F, _L, result);//拷贝数据
    return result;
}
```

### 辅助插入函数

aux为辅助之意，该函数实现在pos位置插入x，该函数辅助后期的push_back等函数

if第一个里面先construct因为空间与对象不能直接赋值，对象只能构建到空间里，对象与对象之间才能赋值

```cpp
void _M_insert_aux(iterator pos, const _Tp & x)
{
    if(_M_finish != _M_end_of_storage)//有剩余空间
    {
        construct(_M_finish, *(_M_finish)-1);//在_M_finish处构建对象*(_M_finish)-1, 即把最后一个对象先向下拷贝一个位置
        ++_M_finish;//更新_M_finish
        copy_backward(pos, _M_finish-2, _M_finish - 1);//将参数1到参数2中的元素从结尾开始复制到参数3终止的范围中，函数返回目标范围中第一个元素的迭代器
        *pos = x;
    }
    else//没有空白空间可供插入，要扩容
    {
        //扩容
        const size_type old_size = size();//旧的大小
        const size_type len = old_size < 4 ? old_size+1 : (old_size + old_size / 2);//扩容策略：1 2 3 4 6 6 9 9 9 13
        iterator new_start = M_alloccate(len);//重新开辟空间
        iterator new_finish = new_start;
        
        //分步拷贝，先拷贝pos之前的，再拷贝pos，再拷贝pos之后的
        new_finish = uninitialized_copy(_M_start, pos, new_start);//将参数1到参数2中元素的副本构造到从参数3开始的范围，并返回目标范围中最后一个元素的迭代器
        construct(new_finish, x);
        ++new_finish;//继续拷贝pos后面的
        new_finish = uninitialized_copy(pos, _M_finish, new_finish);
        
        //析构掉原来的对象
        destroy(begin(),end());//析构对象
        _M_deallocate(_M_start, _M_end_of_storage - _M_start);//释放空间，释放参数1开始的空间长度为参数2
        //更新迭代器
        _M_start = new_start;
        _M_finish = new_finish;
        _M_end_of_storage = _M_start + len;
        
    }
}
```

 `copy_backward(pos, _M_finish-2, _M_finish - 1);`将参数1到参数2中的元素从结尾开始复制到参数3终止的范围中，函数返回目标范围中第一个元素的迭代器

 <img src="img/C++%EF%BC%9ATL_vector%E6%BA%90%E7%A0%81.img/image-20201117142804763.png" alt="image-20201117142804763" style="zoom:50%;" />

扩容后拷贝数据示意图：先拷贝pos之前的，再拷贝pos，再拷贝pos之后的

 <img src="img/C++%EF%BC%9ATL_vector%E6%BA%90%E7%A0%81.img/image-20201117150926910.png" alt="image-20201117150926910" style="zoom:50%;" />

### 末尾插入

vector末尾插入x

```cpp
void push_back(cosnt _Tp & x)
{
    if(_M_finish != _M_end_of_storage)
    {
        construct(_M_finish,x);
        ++_M_finish;
    }
    else
    {
        _M_insert_aux(end(),x);//参数1位置插入x
    }
}
```

### 插入函数

pos位置插入x，返回指向插入的元素的迭代器

```cpp
iterator insert(iterator pos, const _Tp &x)
{
    size_type n = pos - begin();
    if(_M_finish != _M_end_of_storage && pos == end())//尾部插入
    {
    	construct(_M_finish,x);
        ++_M_finish;
    }
    else//非尾部插入
    {
        _M_insert_aux(pos,x);
	}
    return begin()+n;
}
```

一旦扩容，pos迭代器就会失效，所以规定调用insert函数后，迭代器会失效

对于vector来说，只要执行删除操作或插入操作，迭代器就会失效

### 删除尾部

对于vector来说，删除元素并不意味着释放空间，只是析构该对象

```cpp
void pop_back()
{
    --_M_finish;
    destory(_M_finish);//析构
}
```

### 擦除函数

删除pos位置的元素

```cpp
iterator erase(iterator pos)
{
    if(pos + 1 != end())//删除的不是末尾元素
    {
        copy(pos+1, _M_finish, pos);//参数1到参数2之间的元素拷贝到参数3开始的位置
	}
    --_M_finish;
    destory(_M_finish);
    return pos;
}
```

### 重新分配空间

重新分配大小为newsize，初始值为x

```cpp
void resize(size_type new_size, const _Tp &x)
{
    if(new_size < size())
    {
        erase(begin() +new_size(), end());//擦除参数1到参数2之间的元素，这个函数和底下的insert的重载版本该文档并没有实现，可查询源码
	}
    else//new_size大于旧size
    {
        insert(end(),new_size - size(), x);//在参数1到参数2之间的范围里插入x
	}
}
```

## 二维数组内存结构

 创建一个vector容器名字为vec，vec中元素的类型又是vector容器类型![image-20201118093747206](img/C++%EF%BC%9ATL_vector%E6%BA%90%E7%A0%81.img/image-20201118093747206.png)

![image-20201118095003932](img/C++%EF%BC%9ATL_vector%E6%BA%90%E7%A0%81.img/image-20201118095003932.png)

