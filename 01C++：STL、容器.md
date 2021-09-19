C++ STL standard template libaray 标准模板库，学习底层原理和增删改查的使用，容器的底层数据结构有什么样的优缺点，容器就有什么样的优缺点

# allocator

容器中，对象的构造析构，内存的开辟释放是通过容器的空间配置器allocator来实现的，allocator有四个函数：

- allocate：开辟内存
- deallocate：释放内存
- construct：构造对象
- destroy：析构对象

# 一、标准容器

C++11新加容器： array  forward_list(单向链表)  无序关联容器  tuple(元组)

## 1、顺序容器 

### vector

向量容器

底层数据结构：==动态开辟的数组==，每次以原来空间大小的2倍进行扩容的

```cpp
#include <vector>

//定义：
vector<int> vec;

//增加：
vec.push_back(20);  
//在容器末尾添加元素20，O(1)，可能导致容器扩容：会在新内存上拷贝构造原来内存上的对象，再把原来内存上的对象一一析构，然后free原来的内存
vec.insert(it, 20);  
//在it迭代器指向的位置前插入元素，返回指向插入的元素的迭代器，后面的元素会向后挪动，O(n)，也可能导致扩容

//删除：
vec.pop_back();  //末尾删除元素，O(1)
vec.erase(it);   //删除it迭代器指向的元素，后面的元素会向前移动，O(n)

//查询：
vec[3]; //有operator[]函数，可以通过下标实现随机访问，O(1)
通过iterator迭代器进行遍历
泛型算法find，for_each
foreach => 底层就是通过iterator实现

注意：
对容器进行连续插入或者删除操作(insert/erase)，一定要更新迭代器，否则第一次insert或者erase完成，迭代器就失效了

常用方法介绍：
size()  //返回元素个数
empty()  //判空
reserve(20);  //预留空间，只给容器底层开辟指定大小的空间，并不会添加新的元素，size()元素个数仍为0
resize(20);  //重置大小，容器扩容用的，不仅给容器底层开辟指定大小的空间，还会添加新的元素
swap;  //两个容器进行元素交换
```

| 函数    | 功能                           |
| ------- | ------------------------------ |
| reserve | 预分配空间，不初始化           |
| resize  | 重新调整空间大小，会初始化空间 |
| assign  | 清除掉以前的内容，放置新内容   |

代码示例：

```cpp
#include <vector>
#include <iostream>
using namespace std;

int main()
{
	vector<int> vec; // vector<string> vec; 0 1 2 4 8 16 32 64
	//vec.reserve(20); // 给vector容器预留空间
	vec.resize(20);

	cout << vec.empty() << endl;
	cout << vec.size() << endl; // int()

	for (int i = 0; i < 20; ++i)
	{
		vec.push_back(rand()%100 + 1);
	}

	cout << vec.empty() << endl;
	cout << vec.size() << endl;

	// vector的operator[]运算符重载函数
	int size = vec.size();
	for (int i = 0; i < size; ++i)
	{
		cout << vec[i] << " ";
	}
	cout << endl;

	// 把vec容器中所有的偶数全部删除
	auto it2 = vec.begin();
	while (it2 != vec.end())
	{
		if (*it2 % 2 == 0)
		{
			it2 = vec.erase(it2);  //第一次erase，it2就已经失效了，所以每次都要重新给it2赋值，erase的返回值是删除的元素的下一个元素，所以继续判断下一个元素，如果不能被2整除，++it2
		}
		else
		{
			++it2;
		}
	}

	// 通过迭代器遍历vector容器
	auto it1 = vec.begin();
	for (; it1 != vec.end(); ++it1)
	{
		cout << *it1 << " ";
	}
	cout << endl;

	// 给vector容器中所有的奇数前面都添加一个小于奇数1的偶数   44 45    56 57
	for (it1 = vec.begin(); it1 != vec.end(); ++it1)
	{
		if (*it1 % 2 != 0)
		{
			it1 = vec.insert(it1, *it1-1);//注意迭代器失效，每次都要更新迭代器
			++it1;//insert返回指向插入元素的迭代器，所以要加两次
		}
	}

	for (it1 = vec.begin(); it1 != vec.end(); ++it1)
	{
		cout << *it1 << " ";
	}
	cout << endl;
    
    return 0;
}
```

### deque

双端队列容器

底层数据结构：==动态开辟的二维数组==，第一维度以2个开始，以2倍的方式进行扩容，每次扩容后，原来第二维的数组从新的第一维数组的下标oldsize/2开始存放，上下都预留相同的空行，以方便支持dequeue的首尾元素的添加，如下列图

一般队列：头删尾插

双端队列：队头队尾都可插可删，初始first，last在容器中间

<img src="img/C++%EF%BC%9ASTL%E3%80%81%E5%AE%B9%E5%99%A8.img/image-20210224124659023.png" alt="image-20210224124659023" style="zoom:50%;" />

<img src="img/C++%EF%BC%9ASTL%E3%80%81%E5%AE%B9%E5%99%A8.img/image-20210224100306928.png" alt="image-20210224100306928" style="zoom: 50%;" />

```cpp
#include <deque>

//定义:
deque<int> deq;

//增加: 
deq.push_back(20);  //从尾部添加元素，O(1)，可能引起扩容
deq.push_front(20);  //从首部添加元素，O(1) 注意:vector没有push_front方法，可以这样实现首部添加vec.insert(vec.begin());
deq.insert(it, 20);  //it迭代器指向的位置添加元素，O(n)，后面的元素依次向后挪动

//删除:
deq.pop_back();  //从末尾删除元素，O(1)
deq.pop_front();  //从首部删除元素，O(1)
deq.erase(it);  //从it指向的位置删除元素，O(n)

//查询搜索:
使用iterator，注意连续insert/erase要考虑迭代器失效问题
```

### list

链表容器

底层数据结构：==双向的循环链表==，链表结点：pre，data，next

增加删除查询操作和deque一样

 <img src="img/C++%EF%BC%9ASTL%E3%80%81%E5%AE%B9%E5%99%A8.img/image-20210224103940535.png" alt="image-20210224103940535" style="zoom:50%;" />

```cpp
#include <list>

//定义:
list<int> li;

//增加: 
li.push_back(20);  //从尾部添加元素，O(1)
li.push_front(20);  //从首部添加元素，O(1) 
li.insert(it, 20);  //it迭代器指向的位置添加元素，O(1)，链表中进行insert的时候，经常先要进行一个query查询操作，对于链表来说，查询操作效率比较慢

//删除:
li.pop_back();  //从末尾删除元素，O(1)
li.pop_front();  //从首部删除元素，O(1)
li.erase(it);  //从it指向的位置删除元素，O(1)

//查询搜索:
使用iterator，注意连续insert/erase要考虑迭代器失效问题
```

dequeue和list比vector多出来的增加删除接口：

push_front和pop_front，因为对于vector来说这两个方法效率太低，O(n)，所以干脆就不给提供了

### 向量、双端队列、链表对比

容器的纵向考察：容器掌握的深度：使用、底层原理

容器的横向考察：各个相似容器之间的对比

**vector和deque之间的区别？**

vector特点：动态数组，内存是连续的，2倍的方式进行扩容，`vector<int> vec`，定义一个向量数组，该vec容量是0，有插入操作会扩容到1，0-1-2-4-8... ，可以reserve(20)

deque特点：双端队列，动态开辟的二维数组空间，第二维是固定长度的数组空间，扩容的时候是把第一维的数组进行二倍扩容

答：

- 底层数据结构的区别：vector动态数组，deque动态开辟的二维数组
- 前中后插入删除元素的时间复杂度：中间插入都是O(n)，末尾插入都是O(1)，deque前插O(1)，vector没有push_front方法，可以vec.insert(vec.begin());是O(n)
- 对于内存的使用效率：vector低，vector需要的内存空间必须是连续的，而deque可以分块进行数据存储，只需要分段连续，不需要一大片都连续，所以deque对于内存的利用效率更高些
- 在中间进行insert或者erase，vector和deque它们的效率谁能好一点？谁能差一点？虽然时间复杂度都是O(n)，但是vector由于它整片都是连续的，所以挪动起来更方便，效率相比deque更好

**deque底层内存是否是连续的？**

答：不是，每一个第二维内部是连续的，但是二维与二维之间并不是连续的，因为第二维是动态扩容开辟，分段连续

 <img src="img/C++%EF%BC%9ASTL%E3%80%81%E5%AE%B9%E5%99%A8.img/image-20210224105532931.png" alt="image-20210224105532931" style="zoom:50%;" />

**vector和list之间的区别？**

- 底层数据结构：vector动态数组，list双向循环链表
- 数组增加删除是O(n)，涉及元素的移动，链表增加删除结点是O(1)
- 数组查询O(n)，随机访问arr[i]，O(1)，而链表查询也是O(n)，但不支持随机访问

## 2、容器适配器

标准容器 - 容器适配器

怎么理解适配器？

- 容器适配器底层没有自己的数据结构，它是另外一个容器的封装，它的方法全部由底层依赖的容器进行实现的，如下Stack代码，它的底层就是deque，相当于将deque适配成一个Stack
- 没有实现自己的迭代器

```cpp
template<typename T, typename Container=deque<T>>  
class Stack
{
public:
    void push(const T & val) { con.push_back(val); }//底层的deque的末尾作为栈顶进行入栈出栈
    void pop() { con.pop_back(); }
    T top() const { return con.back(); }  
private:
    Container con;
};
```

stack适配器源码：

 <img src="img/C++%EF%BC%9ASTL%E3%80%81%E5%AE%B9%E5%99%A8.img/image-20210224121447705.png" alt="image-20210224121447705" style="zoom:50%;" />

###  stack queue priority_queue

| 容器适配器     | 基本概念                                               | 基本操作                                                     | 底层容器 |
| -------------- | ------------------------------------------------------ | ------------------------------------------------------------ | -------- |
| stack          | 栈：先进后出                                           | push入栈， pop出栈， top查看栈顶元素， empty判空，size返回元素个数 | deque    |
| queue          | 队列：先进先出，后进后出，头出尾入                     | push队尾入， pop队头出， front查看队头元素 ，back查看队尾元素， empty判空， size返回队列中元素个数 | deque    |
| priority_queue | 优先级队列：优先级大的先出队，底层数据结构默认是大根堆 | push入队， pop出队， top查看队顶元素， empty判空， size返回元素个数 | vector   |

**stack与queue底层容器：deque，为什么不依赖vector呢？**

- vector的初始内存使用效率太低了，没有deque好：vector 0-1-2-4-8，deque 4096/sizeof(T)
- 对于queue来说需要支持尾部插入，头部删除，时间复杂度最好都为O(1)，而vector尾插是O(1)，头删却是O(n)，如果queue依赖vector，其出队效率很低
- vector需要大片连续内存，deque只需要分段内存，当存储大量数据时，显然deque对于内存的利用率更好

**priority_queue底层容器：vector，为什么依赖vector，不依赖deque？**

- 优先级队列底层默认把数据组成一个大根堆结构，而大根堆的构建就需要在一个内存连续的数组上（堆中结点和它左右孩子的关系是通过下标计算的），vector动态数组底层是绝对连续的，而deque是分段连续的，所以用vector

    <img src="img/C++%EF%BC%9ASTL%E3%80%81%E5%AE%B9%E5%99%A8.img/image-20210224125903912.png" alt="image-20210224125903912" style="zoom:50%;" />

**三种容器适配器代码示例：**

```cpp
//stack-------------------------------------------------------------------------------
#include <iostream>
#include <stack>
using namespace std;

int main()
{
	stack<int> s1;
	for (int i = 0; i < 20; ++i)
	{
		s1.push(rand() % 100 + 1);
	}

	cout << s1.size() << endl;

	while (!s1.empty())
	{
		cout << s1.top() << " "; //查看栈顶元素的值
		s1.pop();  //栈顶出栈
	}
}

//queue-------------------------------------------------------------------------------
#include <iostream>
#include <queue>
using namespace std;

int main()
{
    queue<int> que;
    for(int i = 0; i < 20; ++i)
    {
        que.push(rand()%100 + 1);  //尾入
    }
    
    cout<<que.size()<<endl;
    
    while (!que.empty())
	{
		cout << que.front() << " ";
		que.pop();   //头出
	}
    return 0;
}

//priority_queue----------------------------------------------------------------------
#include <queue>
#include <iostream>
using namespace std;

int main()
{
	priority_queue<int> pque;

	for (int i = 0; i < 20; ++i)
	{
		pque.push(rand() % 100 + 1);
	}

	cout << pque.size() << endl;

	while (!pque.empty())
	{
		cout << pque.top() << " ";
		pque.pop();
	}
	return 0;
}
```

## 3、关联容器 

主要分为两类：

- set：集合，存的是关键字key
- map：映射表存的是 [key,value]键值对

常用增删查方法：

- 增加：insert(val);
- 遍历：iterator自己搜索或调用find成员方法，`unordered_set<int>::iterator it = set1.find(15);
  	cout << *it;`
- 删除：erase(key)  erase(it)

### 3.1、无序关联容器 => 链式哈希表

无序关联容器底层是链式哈希表，里边元素是无序的，增删查O(1)

无序关联容器在有序关联容器前面加了个unordered_，这些容器操作都是一模一样的，无非就是底层数据结构不同，90%的情况都是在使用无序关联容器，因为大部分应用场景只在乎增删查的时间复杂度要快，如果对元素的顺序有要求则只能选择基于红黑树的有序关联容器

**四种无序关联容器**

- unordered_set 单重集合：不允许key重复
- unordered_multiset 多重集合：允许key重复
- unordered_map 单重映射表
- unordered_multimap 多重映射表

**头文件**

```cpp
//无序关联容器头文件
#include <unordered_set>  //unordered_set  unordered_multiset
#include <unordered_map>  //unordered_map  unordered_multimap 
using namespace std;
```

**unordered_set代码示例**

```cpp
int main()
{
    unordered_set<int> set1;  //不会存储key值重复的元素
    for(int i = 0; i < 50; ++i)
    {
        set1.insert(rand() % 20 + 1);//不像vector/deque/list的insert(it, val) 
    }
    
    cout<<set1.size()<<endl;  //返回容器元素个数
    cout<<set1.count(15)<<endl;  //返回key为15的元素的个数，单重集合，key唯一，输出0或1
}
```

![image-20210224135805672](img/C++%EF%BC%9ASTL%E3%80%81%E5%AE%B9%E5%99%A8.img/image-20210224135805672.png)

存了50个元素却只有18个元素，说明添加的时候有很多是重复的，单重集合是不存储重复的key的

**unordered_map代码示例**

```cpp
/*
map中存的是[key,value]键值对
struct
{
	first;    //key，struct默认访问权限公有
	seconde;  //value
};
*/
int main()
{
	unordered_map<int, string> map1;
    
    //增加
	map1.insert(make_pair(1000, "张三"));  //将键值对打包成pair对象插入到map表中
    map1.insert({1010, "李四"});
    map1.insert({1020, "王五"});
    
    //范文
    cout<<map1.size()<<endl;
    cout<<map1[1000]<<endl;  
    cout<<map1[1010]<<endl;  
    map1[2000] = "wang";  //就相当于 map1.insert({2000,"wang"});
    cout<<map1[2000]<<endl;
    
    //查询
    auto it1 = map1.find(1000);  //find返回的是打包的pair对象
    if(it1 != map1.end())
    {
        cout<<"key: "<<it1->first<<endl;
        cout<<"value: "<<it1->second<<endl;
    }
    
    //删除
    map1.erase(1020);
}
```

![image-20210224144825736](img/C++%EF%BC%9ASTL%E3%80%81%E5%AE%B9%E5%99%A8.img/image-20210224144825736.png)

**应用场景**

- 处理海量数据查重时候经常用到哈希表

```cpp
//海量数据查重
#include <iostream>
#include <unordered_map>
using namespace std;

int main()
{
	const int ARR_LEN = 1000;
	int arr[ARR_LEN] = { 0 };
	for (int i = 0; i < ARR_LEN; ++i)
	{
		arr[i] = rand() % 200 + 1;
	}

	//上面的1000个整数中，统计那些数字重复了，并且统计数字重复的次数
	unordered_map<int, int> map1;  //<数字,次数>
	for (int k : arr)
	{
        /*
		auto it = map1.find(k);
		if (it == map1.end())  //该数字还没存入map表
		{
			map1.insert({ k, 1 });
		}
		else  //之前出现过
		{
			it->second++;
		}*/
        map1[k]++;  //k不存在，[]副作用插入一个新的[k,0]，++到[k,1]
	}

    //打印方式1：
	for (const pair<int, int> &p : map1)  //foreach遍历必须用常引用，只是读操作
	{
		if (p.second > 1)
		{
			cout << "key: " << p.first << "重复次数：" << p.second << endl;
		}
	}
    
    //打印方式2：
    auto it = map1.begin();
    for(; it != map1.end(); ++it)
    {
        if(it->second > 1)
        {
            cout << "key: " << it->first << "重复次数：" << it->second << endl;
        }
    }
}
```

- 海量数据去重用单重set

```cpp
//海量数据去重
//海量数据查重
#include <iostream>
#include <unordered_set>
using namespace std;

int main()
{
	const int ARR_LEN = 1000;
	int arr[ARR_LEN] = { 0 };
	for (int i = 0; i < ARR_LEN; ++i)
	{
		arr[i] = rand() % 200 + 1;
	}

	//上面的1000个整数中，把数字进行去重，打印出只出现一次的数字
	unordered_set<int> uset;
	for (int v : arr)
	{
		uset.insert(v);
	}

	for (int v : uset)
	{
		cout << v << " ";
	}
	cout << endl;
}
```

### 3.2、有序关联容器 => 红黑树 

有序关联容器底层是红黑树，元素是经过排序的，增删查O(log2n)  2是底数(树的层数，树的高度)

有序关联容器的操作与无序关联容器一模一样

用迭代器去遍历有序集合容器时，其实就是对底层红黑树中序遍历的结果

**四种有序关联容器**

- set 单重集合
- multiset 多重集合
- map 单重映射表
- multimap 多重映射表

**头文件**

```cpp
//有序关联容器头文件
#include <set>  //set  multiset
#include <map>  //map  multimap
using namespace std;
```

**有序set示例**

```cpp
int main()
{
	set<int> set1;  //有序单重集合
	for (int i = 0; i < 20; ++i)
	{
		set1.insert(rand() % 20 + 1);
	}
	
	for (int v : set1)
	{
		cout << v << " ";
	}
	cout << endl;
}
```

![image-20210224162928121](img/C++%EF%BC%9ASTL%E3%80%81%E5%AE%B9%E5%99%A8.img/image-20210224162928121.png)

向有序容器中添加自定义类型时注意：要提供默认的<运算符的重载函数

```cpp
class Student
{
public:
	Student(int id, string name) :_id(id), _name(name) {}
	bool operator<(const Student &stu) const //提供默认的<运算符的重载函数
	{
		return _id < stu._id;
	}
private:
	int _id;
	string _name;
	friend ostream& operator<<(ostream &out, const Student & stu);
};

ostream& operator<<(ostream &out, const Student & stu)
{
	cout << "id: " << stu._id << " name: " << stu._name << endl;
	return out;
}

int main()
{
	set<Student> set1;
	set1.insert(Student(1020, "wang"));
	set1.insert(Student(1010, "yang"));
	set1.insert(Student(1030, "zhang"));

	for (auto it = set1.begin(); it != set1.end(); ++it)
	{
		cout << *it << endl;
	}
}
```

![image-20210224164444989](img/C++%EF%BC%9ASTL%E3%80%81%E5%AE%B9%E5%99%A8.img/image-20210224164444989.png)

**有序map示例**

map表中要求value有默认的构造函数，因为[]去引用元素如果不存在会构造一个对象，因此要有默认构造函数

```cpp
class Student
{
public:
	Student(int id=0, string name="") :_id(id), _name(name) {}  //构造函数要有默认值
private:
	int _id;
	string _name;
	friend ostream& operator<<(ostream &out, const Student & stu);
};

ostream& operator<<(ostream &out, const Student & stu)
{
	cout << "id: " << stu._id << " name: " << stu._name << endl;
	return out;
}

int main()
{
	map<int, Student> stuMap;
	stuMap.insert({ 3, Student(1030, "wang") });
	stuMap.insert({ 1, Student(1010, "yang") });
	stuMap.insert({ 2, Student(1020, "zhang") });

	cout << stuMap[4] << endl;

	//打印方式1：
	for (auto it : stuMap)  //stuMap中元素是什么类型，it就是什么类型
	{
		cout <<"key: "<<it.first<<"  value: "<< it.second << endl;
	}

	cout << "-------------------------------------------------------------" << endl;

	//打印方式2：
	auto it = stuMap.begin();
	for (; it != stuMap.end(); it++)
	{
		cout <<"key: "<<it->first<<"  value: "<< it->second << endl;
	}
	
}
```

![image-20210224171328200](img/C++%EF%BC%9ASTL%E3%80%81%E5%AE%B9%E5%99%A8.img/image-20210224171328200.png)

# 二、近容器

数组，string，bitset(位容器)，课程开头提了下，后面没细讲近容器

# 三、迭代器

vec.begin();  //返回首元素迭代器

vec.end();  //end返回容器末尾元素的后继位置

注意迭代器失效问题：对容器进行连续插入或者删除操作(insert/erase)，一定要更新迭代器，否则第一次insert或者erase完成，迭代器就失效了

```cpp
1) 正向迭代器：
容器类名::iterator  迭代器名;

vector<int>::iterator it;
auto it = vec.begin();


2) 常量正向迭代器
容器类名::const_iterator  迭代器名;


3) 反向迭代器
容器类名::reverse_iterator  迭代器名;


4) 常量反向迭代器
容器类名::const_reverse_iterator  迭代器名;
```

**代码示例：**

```cpp
int main()
{
	vector<int> vec;
	for (int i = 0; i < 20; ++i)
	{
		vec.push_back(rand() % 100);
	}
    
    //正向迭代器
	vector<int>::iterator it1 = vec.begin();
	auto it2 = vec.begin();
    for(; it2 != vec.end(); ++it2)
    {
        cout<<*it2<<" ";
    }
    cout<<endl;
    
    //反向迭代器
    //rbegin返回的是最后一个元素的反向迭代器
    //rend返回的是首元素前驱位置的迭代器
    vector<int>::reverse_iterator rit = vec.rbegin();  
    for(; rit != vec.rend(); ++rit)
    {
        cout<<*rit<<" ";
    }
    cout<<endl;
}
```

```cpp
vector<int>::const_iterator it1 = vec.begin();
```

**vec.begin();返回的是普通迭代器，为什么能赋值给const_iterator?**

答：因为const_iterator是iterator的父类，子类可以赋值给父类

```cpp
class const_iterator
{
public:
    const T& operator*() { return *_ptr; }
};
class iterator:public const_iterator
{};
```

# 四、函数对象（类似C的函数指针）

函数对象 =》 C语言里面的函数指针

 <img src="img/C++%EF%BC%9ASTL%E3%80%81%E5%AE%B9%E5%99%A8.img/image-20210225102847530.png" alt="image-20210225102847530" style="zoom: 40%;" />

**使用C的函数指针实现比大小效率低的问题**

```cpp
template<typename T>
bool mygreater(T a, T b)
{
	return a > b;
}

template<typename T>
bool myless(T a, T b)
{
	return a < b;
}

template<typename T, typename Compare>
bool compare(T a, T b, Compare comp)  //Compare推导出来是函数指针
{
    //通过函数指针调用函数，是没有办法内联的，所以会有函数调用开销，效率很低
	return comp(a, b);
}

int main()
{
	cout << compare(1, 2, mygreater<int>) << endl;  //0
	cout << compare(1, 2, myless<int>) << endl;     //1
}
```

> 通过函数指针调用函数，为什么没有办法内联？
>
> 答：编译阶段comp函数指针并不知道它具体要调用mygreater还是myless，所以没有办法内联，只有运行的时候才会跑到这个函数指针具体指向的地址上执行指令

通过函数指针调用函数，是没有办法内联的，所以会有函数调用开销，效率很低。

**C++使用函数对象比大小**

```cpp
template<typename T>
class mygreater
{
public:
    bool operator()(T a, T b)  //两个参数：二元函数对象
    {
        return a>b;
    }
};
template<typename T>
class myless
{
public:
    bool operator()(T a, T b)
    {
        return a<b;
    }
};
template<typename T, typename Compare>
bool compare(T a, T b, Compare comp)  //Compare推导出来是函数对象
{
	return comp(a, b);  //operator()(a,b);
}

int main()
{
    //现在传入的是mygreater与myless的无名对象，Compare推导出来的就是具体对象的operator()(a,b)，即编译过程中comp知道具体调用的是哪个对象的哪个函数，所以可以内联，省下函数调用开销，调高效率
	cout << compare(1, 2, mygreater<int>()) << endl;  //0
	cout << compare(1, 2, myless<int>()) << endl;     //1
}
```

![image-20210227225554868](img/1C++%EF%BC%9ASTL%E3%80%81%E5%AE%B9%E5%99%A8.img/image-20210227225554868.png)

**使用函数对象的好处：**

- 通过函数对象调用operator()，可以通过内联省略函数的调用开销，比通过函数指针调用函数（不能内联）效率高。（使用函数对象虽然我们没有显示加上inline关键字，但编译器优化时可能会优化成内联）
- 因为函数对象是用类生成的，所以类中可以添加相关的成员变量，用来记录函数对象使用时的更多信息

**函数对象使用示例1：将优先级队列从大根堆修改为小根堆**

priority_queque原型：第三个参数默认是less，修改为greater就可以实现小根堆了

 <img src="img/C++%EF%BC%9ASTL%E3%80%81%E5%AE%B9%E5%99%A8.img/image-20210225113327864.png" alt="image-20210225113327864" style="zoom:50%;" />

```cpp
#include <queue>
#include <iostream>
#include <vector>
using namespace std;

int main()
{
	//输出优先级队列，默认从大到小，底层vector，大根堆
	priority_queue<int> que1;
	for (int i = 0; i < 10; ++i)
	{
		que1.push(rand() % 100);
	}
	cout << "默认输出顺序：（大根堆）" << endl;
	while (!que1.empty())
	{
		cout << que1.top() << " ";
		que1.pop();
	}
	cout << endl << endl;

	//通过修改优先级队列的函数对象，使得其组织方式从大根堆变成小根堆，从小到大输出
	using MinHeap = priority_queue<int, vector<int>, greater<int>>;//默认第三个参数是less,修改为greater就变成小根堆了
	MinHeap que2;  //底层vector，大根堆
	for (int i = 0; i < 10; ++i)
	{
		que2.push(rand() % 100);
	}
	cout << "修改为小根堆之后的输出结果：" << endl;
	while (!que2.empty())
	{
		cout << que2.top() << " ";
		que2.pop();
	}
	cout << endl << endl;
}
```

![image-20210225112541229](img/C++%EF%BC%9ASTL%E3%80%81%E5%AE%B9%E5%99%A8.img/image-20210225112541229.png)

 **函数对象使用示例2：将set从小到大输出改变为从大到小输出，set底层是红黑树**

set原型：第二个参数是函数对象，默认是less，修改为greater就可以从大到小输出了

 <img src="img/C++%EF%BC%9ASTL%E3%80%81%E5%AE%B9%E5%99%A8.img/image-20210225113428589.png" alt="image-20210225113428589" style="zoom:50%;" />

```cpp
#include <iostream>
#include <set>
using namespace std;

int main()
{
	set<int> set1;
	for (int i = 0; i < 10; ++i)
	{
		set1.insert(rand() % 100);
	}
	cout << "set默认输出：" << endl;
	for (int v : set1)
	{
		cout << v << " ";
	}
	cout << endl << endl;

	set<int, greater<int>> set2;
	for (int i = 0; i < 10; ++i)
	{
		set2.insert(rand() % 100);
	}
	cout << "set将less改成greater后的输出：" << endl;
	for (int v : set2)
	{
		cout << v<<" ";
	}
	cout << endl << endl;
}
```

![image-20210225114109700](img/C++%EF%BC%9ASTL%E3%80%81%E5%AE%B9%E5%99%A8.img/image-20210225114109700.png)



# 五、泛型算法和绑定器

### 泛型算法

泛型算法是STL库里面定义的一些算法,这些算法可以用一个接口操作各种数据类型,因此称为泛型算法

template + 迭代器 + 函数对象

> 泛型算法详细介绍在C++ Primer 3rd末尾附录

**泛型算法头文件**

```cpp
#include <algorithm>  //包含了C++ STL里面的泛型算法
```

泛型算法特点：

- 泛型算法的参数接收的都是迭代器，为什么？因为泛型算法是给所有容器都能使用的，要通用
- 参数还可以接收函数对象，功能可以更改，例如sort可递增排序，也可改变函数对象实现递减排序

**sort**

`sort(first, last);`

默认对[first, last]之间的元素从小到大排序，采用的是快排算法

`sort(first, last, greater<int>());`

从大到小排序

**binary_search**

`binary_search(vec.begin(), vec.end(), val)`

有序容器中进行二分查找，在由[first,last]标记的有序序列中查找 value 如果找到 则返回 true 否则 返回 false  

**find**

`find(first, last, val);`

find()利用底层元素类型的等于操作符 对[first,last)范围内的元素与 value 进行比较 当发现匹配时 结束搜索过程 且 find()返回指向该元素的一个迭代器， 如果没有发现匹配 则返回容器的end();即尾元素的后继

**find_if**

`find_if( vec.begin(), vec.end(),[](int val)->bool { return val<48; });  `

第三个参数需要一个一元的函数对象

依次检查[first,last)范围内的元素 并把 谓词pred (第三个参数)应用在这些元素上面 如果 pred 计算结果为 true 则搜索过程结束 find_if()返回指向该元素的 InputIterator 如果没有找到匹配 则返回 last  (容器尾元素的后继)

**for_each**

`for_each(vec.begin(), vec.end(), [](int val)->void{...});`

用迭代器遍历容器中的每一个元素，并且对之执行第三个参数的函数对象

### **绑定器**

头文件：`#include <functional>`

bind1st：把二元函数对象的operator()(a, b)的第一个形参绑定起来

bind2st：把二元函数对象的operator()(a, b)的第二个形参绑定起来

绑定器 + 二元函数对象 =》一元函数对象

**泛型算法、绑定器代码示例**

```cpp
#include <iostream>
#include <vector>
#include <algorithm> // 包含了C++ STL里面的泛型算法
#include <functional> // 包含了函数对象和绑定器
using namespace std;

/*
五、泛型算法 = template + 迭代器 + 函数对象
特点一：泛型算法的参数接收的都是迭代器
特点二：泛型算法的参数还可以接收函数对象（C函数指针）
sort,find,find_if,binary_search,for_each

绑定器 + 二元函数对象 =》 一元函数对象
bind1st：把二元函数对象的operator()(a, b)的第一个形参绑定起来
bind2nd：把二元函数对象的operator()(a, b)的第二个形参绑定起来
*/
int main()
{
	int arr[] = { 12,4,78,9,21,43,56,52,42,31};
	vector<int> vec(arr, arr+sizeof(arr)/sizeof(arr[0]));

	for (int v : vec)
	{
		cout << v << " ";
	}
	cout << endl;

	// 默认小到大的排序
	sort(vec.begin(), vec.end());

	for (int v : vec)
	{
		cout << v << " ";
	}
	cout << endl;

	// 有序容器中进行二分查找
	if (binary_search(vec.begin(), vec.end(), 21)) 
	{
		cout << "binary_search 存在" << endl;
	}

	auto it1 = find(vec.begin(), vec.end(), 21);
	if (it1 != vec.end())
	{
		cout << "find 21存在" << endl;
	}

	// 传入函数对象greater，改变容器元素排序时的比较方式
	sort(vec.begin(), vec.end(), greater<int>());
	for (int v : vec)
	{
		cout << v << " ";
	}
	cout << endl;

	// 78 56 52 43 42 31 21 12 9 4
	// 把48按序插入到vector容器当中  找第一个小于48的数字
	// find_if需要的是一个一元函数对象
	// greater a > b    less a < b(48)
	auto it2 = find_if(vec.begin(), vec.end(),
		//bind1st(greater<int>(), 48)); //greater第一个参数绑定48，即48 > val?
		//bind2nd(less<int>(), 48));//less第二个参数绑定48，即val < 48?
		[](int val)->bool {return val < 48; });
	vec.insert(it2, 48);
	for (int v : vec)
	{
		cout << v << " ";
	}
	cout << endl;

	// for_each可以遍历容器的所有元素，可以自行添加合适的函数对象对容器
	// 的元素进行过滤
	for_each(vec.begin(), vec.end(), 
		[](int val)->void
	{
		if (val % 2 == 0)
		{
			cout << val << " ";
		}
	});
	cout << endl;

	return 0;
}
```

# stl下标[]引用元素的副作用

如果元素不存在,下标引用不会抛出异常,而是创建一个此下标的元素,整数默认值是0.

所以用下标返回时可以先find一下看有没有

![image-20210224144007307](img/C++%EF%BC%9ASTL%E3%80%81%E5%AE%B9%E5%99%A8.img/image-20210224144007307.png)

```cpp
map1[2000] = "wang";
//就相当于
map1.insert({2000,"wang"});
```

# multimap使用示例

equal_range会返回一个封装了两个迭代器的 pair 对象，这两个迭代器所确定范围内的元素的键相同，

通过equal_range就可以知道该键所有的value了

```cpp
#include <iostream>
#include <map>
#include <unordered_map>
using namespace std;

int main()
{
	unordered_multimap<int, int> umap;
	umap.insert({ 1,2 });
	umap.insert({ 1,3 });
	umap.insert({ 1,4 });

	//输出方式1：
	for (auto it = umap.begin(); it != umap.end(); ++it)
	{
		cout << it->first << " " << it->second << endl;//输出12,13,14		
	}

	//输出方式2：
	//equal_range会返回一个封装了两个迭代器的 pair 对象，这两个迭代器所确定范围内的元素的键相同
	auto startend = umap.equal_range(1);
	for (auto it = startend.first; it != startend.second; ++it)
	{
		cout << it->first << " " << it->second << endl;  //输出12,13,14		
	}

	multimap<int, int> mp;
	mp.insert({ 5,6 });
	mp.insert({ 7,8 });

	auto sted = mp.equal_range(5);
	for (auto it = sted.first; it != sted.second; ++it)
	{
		cout << it->first << " " << it->second << endl;  //输出56
	}
}
```

# vector为空，size()-1的坑

vector的size()函数返回的是一个无符号整数，当size() == 0，再减1,会导致溢出，从而使数据变大
如代码:

```cpp
int main()
{
	vector<int> arr;
	cout<<arr.size()<<endl;		// 输出  0
	cout<<arr.size() - 1<<endl;	// 输出  4294967295
}
```

**原因：**

vec.size()，size返回值是unsigned int，当vec为空，返回无符号整型0,如果用这个0去减1，相当于32位0减了1， 因为是无符号，所以不会变为负数，而是32位1,32位1的十进制就是4294967295

<img align='left' src="img/01C++%EF%BC%9ASTL%E3%80%81%E5%AE%B9%E5%99%A8.img/image-20210829143014548.png" alt="image-20210829143014548" style="zoom:50%;" />

**该坑常见场景及解决方法：**

```cpp
int main()
{
	vector<int> arr;
	cout << arr.size() << endl;		// 输出  0
	cout << arr.size() - 1 << endl;	// 输出  429496729

	// arr为空，但是表达式居然成立
	for (int i = 0; i < arr.size() - 1; ++i)  
	{
		cout << "i < arr.size()-1成立" << endl;
		arr[i] = i;  // 直接越界访问出错
	}

	// 解决办法：arr.size()返回的是一个无符号整数，直接-1会成一个很大的数，所以先用int接收，再减1
	int size = arr.size();
	for (int i = 0; i < size - 1; ++i)
	{
		cout << "i < size-1成立" << endl;
		arr[i] = i;  // 循坏条件不成立，执行不到这一步
	}
}
```

# 区分greater<int>与less<int>

```cpp
sort (vec.begin(), vec.end());  // 从小到大
sort(vec.begin(), vec.end(), less<int>());  // 从小到大
sort(vec.begin(), vec.end(), greater<int>());  // 从大到小

priority_queue<int>  que0;// 大顶堆
priority_queue<int, vector<int>, less<int>> que1;  // 大顶堆
priority_queue<int,vector<int>,greater<int>> que2;  // 小顶堆
```

