interface类型：所有函数都是纯虚函数，这样的抽象类型称之为接口类型

#### 类内的静态成员

```cpp
class Test
{
private:
	int value;
	static int count;//计数器，静态成员，由该类所有对象共享一份，要在类外给予初始化，也不能在构造函数参数列表进行初始化，因为它只有一份，放在初始列表，每构建一次对象就构建一次计数器，所以不可以
public:
	Test(int x) :value(x) { cout << "Create Test:" << ++count << endl; }
	~Test() {}
	void fun() { cout << " "<<count << endl; }
};
int Test::count = 0;  //类内静态成员，要在类外给予初始化
int main()
{
	Test t1(10);
	Test t2(20);
	cout << sizeof(t1) << endl;  //静态成员放在.data区，不算在对象里面，所以只有value占4字节
	return 0;
}
```

 <img src="img/C++%EF%BC%9A%E8%AE%BE%E8%AE%A1%E6%A8%A1%E5%BC%8F.img/image-20201230171824045.png" alt="image-20201230171824045" style="zoom:67%;" />

#### 类内静态方法

类内的静态方法没有this指针，所以不能访问类的常规数据成员，只能访问类的静态数据成员，而类的普通方法，则两者（常规数据和静态数据）都可以访问

由于类内静态方法没有this指针，所以不能加const，只有有this指针才能加const变为常方法，所以类外的函数（全局函数）也不能尾巴上加const变常方法

#### 友元

**友元函数**

```cpp
class Object
{
private:
	int value;
public:
	Object(){}
	~Object(){}
	friend void fun(Object&)  //友元函数，可以访问该类的私有、保护、公有
};

void fun(Object &obj)
{
	cout << obj.value;
}
```

友元函数尾巴不能加const，因为友元函数仍然是类外函数，它没有this指针

友元的特点：

- 友元是单向的，不具有自反性

- 友元不具有继承性，如下图，基类的友元函数，只可以访问基类的私有，与派生类中隐藏基类的私有，但不可以访问派生类多加的私有

   <img src="img/C++%EF%BC%9A%E8%AE%BE%E8%AE%A1%E6%A8%A1%E5%BC%8F.img/image-20201230213128475.png" alt="image-20201230213128475" style="zoom:67%;" />

- 友元不具有传递性，如下图，B是A的友元，C是B的友元，但C不是A的友元

   <img src="img/C++%EF%BC%9A%E8%AE%BE%E8%AE%A1%E6%A8%A1%E5%BC%8F.img/image-20201230213742846.png" alt="image-20201230213742846" style="zoom:50%;" />

**一个类的成员函数作为另一个类的友元函数**

即在类内把另一个类的成员函数声明为友元，则该函数可以访问本类的私有、保护、公有

**友元类**

一个类可以作为另一个类的友元。如A类是B类的友元类，则A类的所有成员函数都是B类的友元函数

```cpp
class A
{
}
class B
{
    friend A;//将A类说明成B类的友元类，则通过A类的成语函数可以访问B类的私有、保护、公有
}
```

#### map容器的使用

map：关联容器，一对一映射，基于关键字快速查找，不允许有重复值

```cpp
#include <map>
#include <string>
#include <iostream>
using namespace std;
int main()
{
	std::map<string, int> simap; //存放名字和年龄的容器
	simap.insert(std::pair<string, int>("wang", 20));
	simap.insert(std::pair<string, int>("jia", 21));
	simap.insert(std::pair<string, int>("xin", 20));

	cout << simap["wang"];
    cout << simap["jia"];
    cout << simap["xin"];
}
```

![image-20201230222834623](img/C++%EF%BC%9A%E8%AE%BE%E8%AE%A1%E6%A8%A1%E5%BC%8F.img/image-20201230222834623.png)



![image-20201230224652937](img/C++%EF%BC%9A%E8%AE%BE%E8%AE%A1%E6%A8%A1%E5%BC%8F.img/image-20201230224652937.png)



#### 如何实现：只允许在栈区定义对象，不允许在堆区定义对象

答：删掉new操作运算符的重载

<img src="img/C++%EF%BC%9A%E9%9D%99%E6%80%81%E3%80%81%E5%8F%8B%E5%85%83%E3%80%81map.img/image-20201231150657031.png" alt="image-20201231150657031" style="zoom:67%;" />



#### 如何实现：一个类不允许被继承？

答：加final关键字或者构造函数设置为私有

#### 如何实现，父类不允许直接在主函数中被构造，派生类才可以直接创建？

答：父类的构造函数设置为protected

#### 异常

抛出异常，后面的代码就没有必要接着向下执行了，所以一旦抛出异常，就终止当前函数的执行，回收栈帧，释放资源，从异常点回到调用点，异常会一层一层往回抛，直到某个调用点可以处理异常，如果抛到主函数都不能处理异常，就抛给操作系统，操作系统杀死进程

![image-20201231155026851](img/C++%EF%BC%9A%E9%9D%99%E6%80%81%E3%80%81%E5%8F%8B%E5%85%83%E3%80%81map.img/image-20201231155026851.png)

由于抛出了一个类型为 **const char\*** 的异常，因此，当捕获该异常时，必须在 catch 块中使用 const char*。