```cpp
class Object
{
public:
	Object() { cout << "construct object" << this << endl; }
};
int main()
{
	Object * op = new Object[10];
	delete op;

	return 0;
}
```

上述代码可以正确运行

换成`delete [] op`仍旧可以正确运行

 <img src="img/C++%EF%BC%9Adelete%E4%B8%8E%E6%9E%90%E6%9E%84%E5%87%BD%E6%95%B0.img/image-20210110164532947.png" alt="image-20210110164532947" style="zoom:50%;" />

因为它没有析构函数，虽然之前讲的编译器会给任何类默认的析构函数，但是这是在语义上来讲的，实际上如果程序员不写析构函数，系统不会产生析构函数，它会产生语义析构函数，但不会发生真实的调动，那什么时候会产生真实的析构函数呢？如下例子

```cpp
class Base
{
public:
	~Base() {}
};
class Object :public Base
{
public:
	Object() { cout << "construct object" << this << endl; }
};
int main()
{
	Object * op = new Object[10];
	delete op;
	return 0;
}
```

Object类继承了Base类，而Base类给出了析构函数，这时候Object就会产生真实的析构函数，`delete op`就会崩溃，简言之：由于父对象有析构函数，所以在代码生成阶段，子对象也必然有一个析构函数，所以`delete op`崩溃了

 <img src="img/C++%EF%BC%9Adelete%E4%B8%8E%E6%9E%90%E6%9E%84%E5%87%BD%E6%95%B0.img/image-20210110165342397.png" alt="image-20210110165342397" style="zoom:50%;" />

没有显示给出析构函数的类会不会产生真实的析构函数呢？

在以下两种情况下会：父对象有显示的析构函数或成员对象有显示的析构函数，那么本类就会有真实的析构函数，那么delete时会就要求去调动析构函数

注意：由于析构函数会重置虚表指针，所以在某些编译器下，有虚函数的类也会有真实的析构函数（vs没有）

有真实析构函数的类就要严格遵循：开辟多个对象的空间，就要用delete[]去释放多个对象空间的规则，为什么呢？

---

```cpp
class Object
{
public:
	Object()
	{
		cout << "construct object" << this << endl;
	}
};
int main()
{
	Object * op = new Object[10];
	delete op;       //ok
	//delete[] op;   //ok
	//free(op);      //ok
}
```

因为Object类没有真实的析构函数（系统生成的是一个语义上的析构函数），所以上面主函数里三种释放方式都是可以的

如果Object有真实的析构函数，如下，那么`delete op;`与`free(op);`就错误了，因为这时候op上面有4个字节说明它开了10个对象的空间，就要用`delete []op;`

```cpp
class Object
{
public:
	Object()
	{
		cout << "construct object" << this << endl;
	}
    ~Object() {}
};
int main()
{
	Object * op = new Object[10];
	//delete op;   //error
	delete[] op;   //ok
	//free(op);    //error
}
```

---

对于如下只创建一个对象的示例：

```cpp
class Object
{
public:
	Object(int x)
	{
		cout << "construct object" << this << endl;
	}
	~Object() {}
};
int main()
{
	Object * op = new Object(23);
	delete op;        //ok，先调动析构函数再释放空间
	//delete[] op;   //error
	//free(op);       //ok，直接释放空间
}
```

这个时候，用`delete[] op;`会出错，因为Object有真实的析构函数，所以`delete[]op;`时，编译器会认为op上面有四个字节代表对象的个数，但其实op上面并没有，因为op是一个对象

---

类有真实析构函数时，定义该类一组对象时，首地址上面会多产生四个字节代表这组对象的个数，所以要用delete[]

类没有真实的析构函数时，定义该类一组对象时，首地址上面并不会多产生四字节指示个数，因为没有析构所以就没必要记录这组对象的个数，又因为没有这4个字节，所以用delete或delete[]或free释放都可以

