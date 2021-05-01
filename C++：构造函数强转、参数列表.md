## 对象之间的关系

包含：当对象A是对象B的属性时，称对象B包含对象A，如下

```cpp
class CLA
{};
class CLB
{
    int val;
    CLA A;
}
int main()
{
    CLB B; //B对象包含了A对象， 构建时先构建B的成员，再构建B，析构顺序相反
}
```

继承：当对象A是对象B的特例时，称对象A继承对象B

关联：当对象A的引用是对象B的属性时，称对象A和对象B之间是关联关系，所谓对象的引用是指对象的名称，地址，句柄等可以获得和操纵该对象的途径

## 类的成员构造顺序

```cpp
class Object
{
private:
    int value;
    int num;
public:
    Object(int x = 0)
        :num(x), value(num)
    {}
    ~Object() 
    {}
    void Print() const
    {
        cout<<"value: "<<value<<endl;
        cout<<"num: "<<num<<endl;
	}
};
int main()
{
    Object obj(10);//隐式调用构造函数，等价于Object obj = Object(10)
    obj.Print();
    return 0;
}
```

打印结果：

![image-20201121115257710](img/C++%EF%BC%9A%E7%BB%A7%E6%89%BF%E7%AC%AC%E4%B8%80%E8%AF%BE.img/image-20201121115257710.png)

==成员构建的顺序与类中设计的顺序有关，和参数列表的顺序无关，所以先构建value，再构建num==

## 构造函数完成类型转换

构造函数功能：

- 创建对象（不创建空间）
- 初始化对象
- 类型转换

内置类型可以赋值给对象，构造函数完成了类型转换

```cpp
int main()
{      
    Object obj(30);
    int x = 100;
    
    obj = x;//正确，等价于: obj = (Object)x; 编译器会调用Object单参的构造函数去创建一个临时对象，把创建好的对象再赋值给obj，赋值完成以后，临时对象生存期结束，调动析构函数析构临时对象；构造函数可以当类型转换的前提条件是：构造函数只有一个参数
}
```

函数前加`explicit`关键字可以禁止隐式的类型转换，构造函数前加它可以消除构造函数隐式类型转换的功能

```cpp
explicit Object(int x = 0):value(x), num(x)//消除构造函数的隐式的类型转换功能
{
    cout<<"Creat Object: "<<this<<endl;
}
```

加了explicit关键字，隐式转换不可以，显示的仍旧可以

```cpp
obj = x;         //隐式转换: 不可以
obj = (Object)x; //显示转换: 可以
```

对于两个参数的构造函数：

```cpp
explicit Object(int x, int y):value(x), num(y)
{
    cout<<"Creat Object: "<<this<<endl;
}

int main()
{
    Object obj(10,10);
    int x = 100, y = 200;
    
    obj = (Object)x,y;//不可以，这是个逗号表达式，先执行obj=(Object)x,再执行y,由于构造函数有两个参数所以没有类型转换功能，逗号前的表达式出错，如果构造函数里y有默认值，则变为单参构造函数，就可以
    
	//若是这样: obj = (Object)(x,y);则先算(x,y)逗号表达式的值(逗号表达式把最右边的值作为表达式的值)为y，再把y的值作为构造函数强转的参数, 即变为obj=(Object)y;
    
    obj = Object(x,y);//可以，这是构造函数构建无名对象
}
```

总结：

- `(Object)x`是强转，`Object(x)`是调动构造函数构建无名对象

- 单参构造函数具有类型转换功能，多参构造函数其他参数若有默认值则仍具有类型转换功能

- 逗号表达式把最右边的值作为表达式的值

- 逗号运算符优先级最弱，有=和，先算=

   <img src="img/C++%EF%BC%9A%E7%BB%A7%E6%89%BF%E7%AC%AC%E4%B8%80%E8%AF%BE.img/image-20201121124000270.png" alt="image-20201121124000270" style="zoom:50%;" />

## 重载强转运算符

```cpp
//类内定义，成员函数有this指针
operator int() const//重载强转
{
    return value; //或者return num;
}
int main()
{
    Object obj(10,10);
    int x = 100;
    
    x = (int)obj;//x=obj.operator int() 编译器改写: x=operator int(&obj);
}
```

构造函数不写返回值，但其实返回的是构建的对象

重载强转，不写返回值，其实返回的是强转的类型

```cpp
obj = x; //要设计单参构造函数

x = (int)obj;//要设计重载强转
```

自定了构造函数带参数，那么默认的就没了吗？
答:只要我们定义了一个构造函数，系统就不会自动生成缺省的构造函数

>缺省构造函数，又称默认构造函数，是C++以及其他的一些面向对象的程序设计语言中，对象的不需要参数即可调用的构造函数

有重载强转函数，就可以完成类对象的比较> < ==,比较时会把对象强转为int或double（这些比较运算符是数量类型的专用运算符）

## 构造函数参数列表

```cpp
class Test
{
    int ix;
    Object obj;
public:
    Test(int x = 10):ix(x)//,obj(x+10)
    {
        ix = x;			//对于内置类型来说，这里赋值和参数列表赋值没区别
        obj = (x+10);	//进入Test构造函数之前构建了一次obj对象，这里强转又会构建一次Object类型的无名对象，而在参数列表里写则只构建一次就给obj赋好初始值了
        cout<<"Create Test: "<<this<<endl;
	}
};
int main()
{
    Test test;
}
```

==test对象包含了obj对象，调动Test构造函数构建test对象时先会构建test的成员ix和obj，然后进入Test构造函数函数体==，所以在参数列表里给成员的构造函数初始值效率比在函数体里写要高

对于类的成员，尽可能拿列表方式给予初始化，例如如下的拷贝构造函数，用参数列表给予值比较好

```cpp
Test(const Test & it):ix(it.ix),obj(it.obj)
{
    //ix = it.ix;
    //obj = it.obj;//用参数列表方式效率高
}
```

注意：只有构造函数和拷贝构造函数可以用列表方案，因为列表方案是来构建对象的，例如如下是不允许的

```cpp
Test & operator=(const Test & it):obj(it.obj)//不允许，因为obj对象已经是存在的，这里列表方式会再构建一次就会出错
{
    //obj = it.obj;//只能这样子
}
```

## 作业题

```cpp
class Object
{
	int value;
    int num;
public:
    Object(int x = 0,int y = 0):value(x),num(y)
    {
        cout<<"Create Object: "<<this<<endl;
	}
    Object(const Object & obj):value(obj.value),num(obj.num)
    {
        cout<<"Copy Create Object: "<<this<<endl;
	}
    Object & operator=(const Object & obj)
    {
        if(this != &obj)
        {
            value = obj.value;
            num = obj.num;
		}
        cout<<this<<" = "<<&obj<<endl;
        return *this;
    }
    ~Object()
    {
        cout<<"Destroy Object: "<<this<<endl;
    }
};
```

```cpp
class Test1
{
private:
    int ix;
    Object &obj;//问题1: 这个如何构建？
};
class Test2
{
private:
    int ix;
    Object *pobj;//问题2: 这个如何构建？
};
class Test3
{
    int ix;
    Test1 *next;//问题3：这个如何构建
    Test1 &test;//问题4：这个如何构建
};
```

