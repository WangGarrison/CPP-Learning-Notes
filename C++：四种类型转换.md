# C语言中的类型转换

```c
(int)a;
```

C语言的类型强转方式如下：(Type) value/expression，就是在变量或者表达式前面加小括号和类型表示类型强转。  

```c
int main()
{
	int a = 10;
	float ft = 12.25f;
	int *ip = &a;
	float *fp = &ft;
	a = ft; // 隐式转换
	a = (int)ft; // 显示转换
	ip = (int *) fp; // 显示转换
	return 0;
}
```

C语言的类型转换意义不明确。 如 :（都是显示转换，但意义不一样）

```c
a= (int)ft;
ip = (int *)fp; 
```

C类型装换不安全：

```cpp
int main()
{
    int *p = nullptr;
    double *b = (double*)p; //可以转换，但不安全，p本来是是整形指针变量，解引用只能访问4字节，但是强转成doouble*，解引用就可以访问8字节，非法访问内存
}
```

# C++的四种类型强转  

C++提供了自己特有的四种类型强转，分别如下：  

## const_cast

```cpp
const_cast<new_type> (expression);  
```

==去除指针或引用的const属性==，new_type为去除const后的类型

变量本身的const属性是不能去除的，要想修改变量的值，一般是去除指针（或引用）的const属性，再进行间接修改。所以**强制转换类型必须是指针*或引用 &**  ，示例如下：

 <img src="img/C++%EF%BC%9A%E5%9B%9B%E7%A7%8D%E7%B1%BB%E5%9E%8B%E8%BD%AC%E6%8D%A2.img/image-20210111154308947.png" alt="image-20210111154308947" style="zoom:50%;" />

通过const_cast运算符，也只能将const type*转换为type*，将const type&转换为type&。

也就是说目标类型和源类型除了const属性不同，其他地方完全相同。

<img align='left' src="img/C++%EF%BC%9A%E5%9B%9B%E7%A7%8D%E7%B1%BB%E5%9E%8B%E8%BD%AC%E6%8D%A2.img/image-20210419193752287.png" alt="image-20210419193752287" style="zoom: 45%;" />

## static_cast(用的最多)

```cpp
static_cast<new_type> (expression); 
```

**static_cast**能==提供编译器认为安全的互相转换==，该运算符把expression转换为new_type类型，但没有运行时类型检查来保证转换的安全性，对于类只能在有联系的指针类型间进行转换  

主要用法如下：

- 用于基本类型间的转换  

  ```cpp
  char a = 'a';
  int b = static_cast<char>(a);//正确，将char型数据转换成int型数据
  
  double *c = new double;
  void *d = static_cast<void*>(c);//正确，将double指针转换成void指针
  
  int e = 10;
  const int f = static_cast<const int>(e);//正确，将int型数据转换成const int型数据
  
  const int g = 20;
  int *h = static_cast<int*>(&g);//编译错误，static_cast不能转换掉g的const属性
  ```

- 不能用于基本类型强转为指针  

    <img align='left' src="img/C++%EF%BC%9A%E5%9B%9B%E7%A7%8D%E7%B1%BB%E5%9E%8B%E8%BD%AC%E6%8D%A2.img/image-20210111160036610.png" alt="image-20210111160036610" style="zoom:50%;" />

- 不能用于编译器认为不安全的类型转换
  
  ![image-20210225230437170](img/C++%EF%BC%9A%E5%9B%9B%E7%A7%8D%E7%B1%BB%E5%9E%8B%E8%BD%AC%E6%8D%A2.img/image-20210225230437170.png)

- **用于类层次结构中基类（父类）和派生类（子类）之间指针或引用的转换**。 
  进行**上行**转换（把派生类的指针或引用转换成基类表示）是**安全**的； 
  进行**下行**转换（把基类指针或引用转换成派生类表示）时，由于没有动态类型检查，所以是**不安全**的。   

  ```cpp
  class Base
  {};
  
  class Derived : public Base
  {};
  
  int main()
  {
  	Base* pB = new Base();
  	if (Derived* pD = static_cast<Derived*>(pB))
  	{
  	}//下行转换是不安全的(坚决抵制这种方法)
  
  	Derived* pD = new Derived();
  	if (Base* pB = static_cast<Base*>(pD))
  	{
  	}//上行转换是安全的
  }
  ```

- 用于其它类型（基本类型和类类型）向类类型之间的转换；static_cast<类类型>(其它类型)，依赖构造函数的类型转换功能

- 把空指针转换成目标类型的空指针，把任何类型的表达式转换成void类型

- static_cast不能转换掉expression的const

## reinterpret_cast

```cpp
reinterpret_cast<new_type>(expression);
```

reinterpret_cast可以把任何指针转换成其它类型的指针，new_type必须是一个指针、引用、算术类型、指向函
数的指针或指向一个类成员的指针。任何指针都可以转换成其它类型的指针，不安全的转换

==类似C的强转==，不安全

```cpp
class Base {};
class Test {};
int main()
{
	int a = 10;
	float* fp = reinterpret_cast<float*>(&a);
	Base b;
	Test* tp = reinterpret_cast<Test*>(&b); // 不安全
	return 0;
}
```

## dynamic_cast

```cpp
Base* ba = new Base(2);
Derived* de = dynamic_cast<Derived*>(ba);
```

dynamic_cast转换符只能用于含有虚函数的类，dynamic_cast运算符的主要用途：==将基类的指针或引用安全地转换成派生类的指针或引用==

当我们将dynamic_cast用于某种类型的指针或引用时，只有该类型至少含有虚函数时(最简单是基类析构函数为虚函数)，才能进行这种转换。否则，编译器会报错

dynamic_cast转换操作符在执行类型转换时首先将检查能否成功转换，如果能成功转换则转换之如果转换失败，T是指针则反回一个nullptr，如果是转换的是T引用，则抛出一个bad_cast异常，所以在使用dynamic_cast转换之间应使用if语句对其转换成功与否进行测试  

 <img src="img/C++%EF%BC%9A%E5%9B%9B%E7%A7%8D%E7%B1%BB%E5%9E%8B%E8%BD%AC%E6%8D%A2.img/image-20210111170955155.png" alt="image-20210111170955155" style="zoom:50%;" />

```cpp
/*
C++语言级别提供的四种类型转换方式
int a = (int)b;
const_cast : 去掉（指针或者引用）常量属性的一个类型转换
static_cast :  提供编译器认为安全的类型转换（没有任何联系的类型之间的转换就被否定了）
reinterpret_cast : 类似于C风格的强制类型转换
dynamic_cast : 主要用在继承结构中，可以支持RTTI类型识别的上下转换
*/

class Base
{
public:
	virtual void func() = 0;
};
class Derive1 : public Base
{
public:
	void func() { cout << "call Derive1::func" << endl; }
};
class Derive2 : public Base
{
public:
	void func() { cout << "call Derive2::func" << endl; }
	// Derive2实现新功能的API接口函数
	void derive02func()
	{
		cout << "call Derive2::derive02func" << endl;
	}
};
/*
typeid(*p).name() == "Derive"
*/
void showFunc(Base *p)
{
	// dynamic_cast会检查p指针是否指向的是一个Derive2类型的对象？
	// p->vfptr->vftable RTTI信息 如果是，dynamic_cast转换类型成功，
	// 返回Derive2对象的地址，给pd2；否则返回nullptr
	// static_cast编译时期的类型转换  dynamic_cast运行时期的类型转换 支持RTTI信息识别
	Derive2 *pd2 = dynamic_cast<Derive2*>(p); //static_cast的话永远能转换成功，因为static_cast编译时期类型转换
	if (pd2 != nullptr)
	{
		pd2->derive02func();
	}
	else
	{
		p->func(); // 动态绑定  *p的类型 Derive2  derive02func
	}
}
int main()
{
	Derive1 d1;
	Derive2 d2;
	showFunc(&d1);
	showFunc(&d2);

	//static_cast 基类类型 《=》 派生类类型  能不能用static_cast?当然可以！
	//int *p = nullptr;
	//double* b = reinterpret_cast<double*>(p);

	//const int a = 10;
	//int *p1 = (int*)&a;
	//int *p2 = const_cast<int*>(&a);
	// const_cast<这里面必须是指针或者引用类型 int* int&>
	//int b = const_cast<int>(a);

	return 0;
}
```

# RTTI  

RTTI(Run Time Type Identification)即通过运行时类型识别，程序能够使用基类的指针或引用来检查这些指针或引用所指的对象的实际派生类型  

## RTTI机制的产生  

为什么会出现RTTI这一机制，这和C++语言本身有关系。和很多其他语言一样，C++是一种静态类型语言。其数据类型是在编译期就确定的，不能在运行时更改。然而由于面向对象程序设计中多态性的要求，C++中的指针或引用(Reference)本身的类型，可能与它实际代表(指向或引用)的类型并不一致。

有时我们需要将一个多态指针转换为其实际指向对象的类型，就需要知道运行时的类型信息，这就产生了运行时类型识别的要求。和Java相比，C++要想获得运行时类型信息，只能通过RTTI机制，并且C++最终生成的代码是直接与机器相关的  

RTTI提供了两个非常有用的操作符：typeid和dynamic_cast

- typeid：typeid操作符，返回指针和引用所指的实际类型。
- dynamic_cast：将基类类型的指针或引用安全地转换为派生类类型的指针或引用

## typeid

typeid操作符，返回指针和引用所指的实际类型；  

头文件：`#include <typeinfo>`

```cpp
template<class T>
void fun(T a)
{
	cout << typeid(a).name() << endl;  //typeif(a).name()返回变量a的类型（字符串方式）
}

int main()
{
	int a = 10;
	fun(a);
	fun(&a);
}
```

 <img src="img/C++%EF%BC%9A%E5%9B%9B%E7%A7%8D%E7%B1%BB%E5%9E%8B%E8%BD%AC%E6%8D%A2.img/image-20210111171046440.png" alt="image-20210111171046440" style="zoom:50%;" />