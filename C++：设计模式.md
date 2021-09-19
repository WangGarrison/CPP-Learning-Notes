# 设计模式简介

设计模式是一套被反复使用的、多数人知晓的、经过分类编目、代码设计经验的总结。使用设计模式是为了重用代码、让代码更容易被他人理解、保证代码可靠性。 

设计模式融合了众多专家的经验，并以一种标准的形式供广大开发人员所用提供了一套通用的设计词汇和一种通用的语言，以方便开发人员之间进行沟通和交流，使得设计方案更加通俗易懂让人们可以更加简单方便地复用成功的设计和体系结构， 使得设计方案更加灵活，且易于修改， 可以提高软件系统的开发效率和软件质量，在一定程度上节约设计成本  

# GOF（四人帮，Gang of Four）

在 1994 年，由 Erich Gamma、Richard Helm、Ralph Johnson 和 John Vlissides 四人合著出版了一本名为 Design Patterns - Elements of Reusable Object-Oriented Software（中文译名：设计模式 - 可复用的面向对象软件元素） 的书，该书首次提到了软件开发中设计模式的概念。四位作者合称 GOF

该书归纳了 23 种在软件开发中使用频率较高的设计模式，旨在用模式来统一沟通面向对象方法在分析、设计和实现间的鸿沟  

设计模式一般有如下几个基本要素：模式名称、问题、目的、解决方案、效果、实例代码和相关设计模式，其中的关键元素包括以下四个方面：  **模式名称 (Pattern name) ， 问题 (Problem) ， 解决方案 (Solution) ， 效果 (Consequences) 。**  

# 七大原则

#### 开放封闭原则  OCP

Open Close Principle：==对扩展开放，对修改关闭==

#### 单一职责原则  SRP

Single Responsibility Principle：==一个类只做一件事==。就一个类而言，应该仅有一个引起它变化的原因  

#### 依赖倒转原则  DIP

Dependence Inversion Principle：这个原则是开闭原则的基础，指：针对接口编程，==依赖于抽象而不依赖于具体==。这样就降低了客户与实现模块的耦合  

高层模块不应该依赖于低层模块。两个都应该依赖抽象。抽象不应该依赖细节。细节应依赖于抽象。  

#### 迪米特原则，最小知识原则  LOP

Law of Demeter：一个实体应当尽量少地与其他实体之间发生相互作用，使得系统功能模块相对独立。高内聚，==低耦合==

#### 接口隔离原则  ISP

Interface Segregation Principle：使用多个==隔离的接口==，比使用单个接口要好。它还有另外一个意思是：降低类之间的耦合度。

接口隔离原则的核心定义，是不出现臃肿的接口，但是“小”是有限度的，首先就是不能违反单一职责原则。  

#### 合成/聚合复用原则  CRP（非六大）

Composite Reuse Principle：尽量使用合成/聚合的方式，而==不是使用继承==。

#### 里氏代换原则  LSP

Liskov Substitution Principle：里氏代换原则是面向对象设计的基本原则之一。 里氏代换原则中说，==任何基类可以出现的地方，子类一定可以出现==

LSP 是继承复用的基石，只有当派生类可以替换掉基类，且软件单位的功能不受到影响时，基类才能真正被复用，而派生类也能够在基类的基础上增加新的行为。

里氏代换原则是对开闭原则的补充。实现开闭原则的关键步骤就是抽象化，而基类与子类的继承关系就是抽象化的具体实现，所以里氏代换原则是对实现抽象化的具体步骤的规范。

# 设计模式分类

根据目的（模式是用来做什么的）可分为创建型(Creational)，结构型(Structural)和行为型(Behavioral)三类：  

- <font color='red'>创建型</font>  模式主要用于创建对象
- <font color='red'>结构型</font>  模式主要用于处理类或对象的组合  
- <font color='red'>行为型</font>  模式主要用于描述类或对象如何交互和怎样分配职责  

根据范围，即模式主要是处理类之间的关系还是处理对象之间的关系，可分为类模式和对象模式两种：

- 类模式处理类和子类之间的关系，这些关系通过继承建立，在编译时刻就被确定下来，是一种静态关系
- 对象模式处理对象间的关系，这些关系在运行时变化，更具动态性。  

# 创建型模式

这些设计模式提供了一种在创建对象的同时隐藏创建逻辑的方式，而不是使用 new 运算符直接实例化对象。这使得程序在判断针对某个给定实例需要创建哪些对象时更加灵活。

## 简单工厂模式

**定义**

简单工厂模式提供了专门的工厂类用于创建对象，将对象的创建和对象的使用分离开，它作为一种最简单的工厂模式在软件开发中得到了较为广泛的应用  

定义一个工厂类，==它可以根据参数的不同返回不同类的实例==，被创建的实例通常都具有共同的父类。因为在简单工厂模式中用于创建实例的方法是静态(static)方法，因此简单工厂模式又被称为静态工厂方法(Static Factory Method)模式，它属于类创建型模式  

当你需要什么，只需要传入一个正确的参数，就可以获取你所需要的对象，而无须知道其创建细节。简单工厂模式结构比较简单，其核心是工厂类的设计  

简单工厂模式属于创建型模式，它提供了一种创建对象的最佳方式。

**意图**

定义一个创建对象的接口，让其子类自己决定实例化哪一个工厂类，工厂模式使其创建过程延迟到子类进行。

**具体做法**

首先将需要创建的各种不同对象的相关代码封装到不同的类中，这些类称为具体产品类，而将它们公共的代码进
行抽象和提取后封装在一个抽象产品类中，每一个具体产品类都是抽象产品类的子类；然后提供一个工厂类用于创建各种产品，在工厂类中提供一个创建产品的工厂方法，该方法可以根据所传入的参数不同创建不同的具体产品对象；客户端只需调用工厂类的工厂方法并传入相应的参数即可得到一个产品对象  

**UML图**

 <img src="img/C++%EF%BC%9A%E8%AE%BE%E8%AE%A1%E6%A8%A1%E5%BC%8F.img/image-20210103121746241.png" alt="image-20210103121746241" style="zoom:50%;" />

**Factory**（工厂角色）：工厂角色即工厂类，它是简单工厂模式的核心，负责实现创建所有产品实例的内
部逻辑；工厂类可以被外界直接调用，创建所需的产品对象；  在工厂类中提供了静态的工厂方法 factoryMethod()，它的返回类型为抽象产品类型 Product。  

**Product**（抽象产品角色）：它是工厂类所创建的所有对象的父类，封装了各种产品对象的公有方法，它的引入将提高系统的灵活性，使得在工厂类中只需定义一个通用的工厂方法，因为所有创建的具体产品对象都是其子类对象  

**ConcreteProduct**（具体产品角色）：它是简单工厂模式的创建目标，所有被创建的对象都充当这个角色的某个具体类的实例。每一个具体产品角色都继承了抽象产品角色，需要实现在抽象产品中声明的抽象方法。  

在简单工厂模式中，客户端通过工厂类来创建一个产品类的实例，而无须直接使用 new 关键字来创建对象，它是工厂模式家族中最简单的一员。  

**优点**

- 工厂类包含必要的判断逻辑，可以决定在什么时候创建哪一个产品类的实例，客户端可以免除直接创建
  产品对象的职责，而仅仅“消费”产品，简单工厂模式实现了对象创建和使用的分离。  
- 客户端无须知道所创建的具体产品类的类名，只需要知道具体产品类所对应的参数即可，对于一些复杂
  的类名，通过简单工厂模式可以在一定程度减少使用者的记忆量  
- 通过引入配置文件，可以在不修改任何客户端代码的情况下更换和增加新的具体产品类，在一定程度上
  提高了系统的灵活性  

- 屏蔽产品的具体实现，调用者只关心产品的接口。

**缺点**

- 由于工厂类集中了所有产品的创建逻辑，职责过重，一旦不能正常工作，整个系统都要受到影响  
- 使用简单工厂模式势必会增加系统中类的个数（引入了新的工厂类），增加了系统的复杂度和理解难
  度  
- 一旦添加新产品就不得不修改工厂逻辑，在产品类型较多时，有可能造成工厂逻辑过于复杂，不利于系统的扩展和维护  
- 简单工厂模式由于使用了静态工厂方法，造成工厂角色无法形成基于继承的等级结构。  

**实例**

工厂类负责创建的对象比较少，由于创建的对象较少，不会造成工厂方法中的业务逻辑太过复杂时可以使用简单工厂模式。  

- 您需要一辆汽车，可以直接从工厂里面提货，而不用去管这辆汽车是怎么做出来的，以及这个汽车里面的具体实现。 

- 设计一个连接服务器的框架，需要三个协议，"POP3"、"IMAP"、"HTTP"，可以把这三个作为产品类，共同实现一个接口。

**注意事项**

作为一种创建类模式，在任何需要生成复杂对象的地方，都可以使用工厂方法模式。有一点需要注意的地方就是复杂对象适合使用工厂模式，而简单对象，特别是只需要通过 new 就可以完成创建的对象，无需使用工厂模式。如果使用工厂模式，就需要引入一个工厂类，会增加系统的复杂度。

**代码示例**

<img src="img/C++%EF%BC%9A%E8%AE%BE%E8%AE%A1%E6%A8%A1%E5%BC%8F.img/image-20210103113251450.png" alt="image-20210103113251450" style="zoom:67%;" />

```cpp
#include <iostream>
#include <stdexcept>
#include <cstddef>
#include <string>
#include<map>
using namespace std;

class BadShapeCreation : public logic_error
{
public:
	BadShapeCreation(string type): logic_error("Cannot create type " + type) {}
};
class Shape
{
public:
	virtual void draw() = 0;
	virtual void erase() = 0;
	virtual ~Shape() {}
	static Shape* factory(const string& type);
};
class Circle : public Shape
{
	Circle() {} // Private constructor
	friend class Shape;
public:
	void draw()
	{
		cout << "Circle::draw" << endl;
	}
	void erase()
	{
		cout << "Circle::erase" << endl;
	}
	~Circle()
	{
		cout << "Circle::~Circle" << endl;
	}
};

class Square : public Shape
{
	Square() {}
	friend class Shape;
public:
	void draw()
	{
		cout << "Square::draw"<< endl;
	}
	void erase()
	{
		cout << "Square::erase" << endl;
	}
	~Square()
	{
		cout << "Square::~Square" << endl;
	}
};

Shape* Shape::factory(const string& type) throw(BadShapeCreation)
{
	if (type == "Circle") return new Circle;
	if (type == "Square") return new Square;
	throw BadShapeCreation(type);
}

class ShapeFactory
{
	std::map<string, Shape*> shapes;
public:
	ShapeFactory(string str[], int n)
	{
		for (int i = 0; i < n; ++i)
		{
			shapes.insert(std::map<string, Shape*>::value_type(str[i], Shape::factory(str[i])));
		}
	}
	~ShapeFactory()
	{
		for (auto& x : shapes)
		{
		}
	}
	Shape* GetShape(const string str)
	{
		return shapes[str];
	}
};

int main()
{
	string sname;
	string str[] = { "Circle","Square" };
	int n = sizeof(str) / sizeof(str[0]);
	ShapeFactory sfact(str, n);
	Shape * sp = nullptr;
	cin >> sname;
	sp = sfact.GetShape(sname);
	sp->draw();
	return 0;
}
```

## 工厂方法模式

**简单工厂问题**

在简单工厂模式中只提供一个工厂类，该工厂类处于对产品类进行实例化的中心位置，它需要知道每一个产品对象的创建细节，并决定何时实例化哪一个产品类。简单工厂模式最大的缺点是当有新产品要加入到系统中时，必须修改工厂类，需要在其中加入必要的业务逻辑，这违背了“开闭原则”。此外，在简单工厂模式中，所有的产品都由同一个工厂创建，工厂类职责较重，业务逻辑较为复杂，具体产品与工厂类之间的耦合度高，严重影响了系统的灵活性和扩展性，而工厂方法模式则可以很好地解决这一问题  

在工厂方法模式中，我们不再提供一个统一的工厂类来创建所有的产品对象，而是==针对不同的产品提供不同的工厂==，系统提供一个与产品等级结构对应的工厂等级结构  

**定义**

工厂方法模式(Factory Method Pattern)：定义一个用于创建对象的接口，让子类决定将哪一个类实例化。

工厂方法模式让一个类的实例化延迟到其子类。工厂方法模式又简称为工厂模式(Factory Pattern)，又可称作虚
拟构造器模式(Virtual Constructor Pattern)或多态工厂模式(Polymorphic Factory Pattern)  

工厂方法模式提供一个抽象工厂接口来声明抽象工厂方法，而由其子类来具体实现工厂方法，创建具体的产品对象。  

工厂方法模式是简单工厂模式的延伸，它继承了简单工厂模式的优点，同时还弥补了简单工厂模式的不足。工厂方法模式是使用频率最高的设计模式之一，是很多开源框架和 API 类库的核心模式  

在工厂方法模式中，核心的工厂类不再负责所有产品的创建，而是将具体创建工作交给子类去做。这个核心类仅仅负责给出具体工厂必须实现的接口，而不负责哪一个产品类被实例化这种细节，这使得工厂方法模式可以允许系统在不修改工厂角色的情况下引进新产品。  

**UML图**

![image-20210103121051223](img/C++%EF%BC%9A%E8%AE%BE%E8%AE%A1%E6%A8%A1%E5%BC%8F.img/image-20210103121051223.png)

Product：抽象产品，是具体产品类型的基类，用于描述具体产品的公共接口

Concrete Product：具体产品，抽象产品类型的派生类（子类） ,工厂类创建的目标类（具体工厂和具体产品之间一一对应），描述生产的具体产品  

Factory：抽象工厂，具体工厂的父类，声明了工厂方法(Factory Method)，用于返回一个产品。抽象工厂是工厂方法模式的核心，所有创建对象的工厂类都必须实现该接口  

Concrete Factory ：具体工厂，抽象工厂的子类；被外界调用;  描述具体工厂；实现 FactoryMethod（创建产品的实例） 工厂方法，并可由客户端调用，返回一个具体产品类的实例  

与简单工厂模式相比，工厂方法模式最重要的区别是引入了抽象工厂角色，抽象工厂可以是接口，也可以是抽象类或者具体类。  

**优点**

- 在工厂方法模式中，工厂方法用来创建客户所需要的产品，同时还向客户隐藏了哪种具体产品类将被实例化这一细节，用户只需要关心所需产品对应的工厂，无须关心创建细节，甚至无须知道具体产品类的类名  
- 基于工厂角色和产品角色的多态性设计是工厂方法模式的关键。它能够让工厂可以自主确定创建何种产品对象，而如何创建这个对象的细节则完全封装在具体工厂内部。工厂方法模式之所以又被称为多态工厂模式，就正是因为所有的具体工厂类都具有同一抽象父类。  
- 系统中加入新产品时，无须修改抽象工厂和抽象产品提供的接口，无须修改客户端，也无须修改其他的具体工厂和具体产品，而只要添加一个具体工厂和具体产品就可以了，这样，系统的可扩展性也就变得非常好，完全符合“开闭原则“

**缺点**

- 在添加新产品时，需要编写新的具体产品类，而且还要提供与之对应的具体工厂类，系统中类的个数将成对增加，在一定程度上增加了系统的复杂度，有更多的类需要编译和运行，会给系统带来一些额外的开销。  
- 由于考虑到系统的可扩展性，需要引入抽象层，在客户端代码中均使用抽象层进行定义，增加了系统的抽象性和理解难度，且在实现时可能需要用到 DOM、反射等技术，增加了系统的实现难度  

**适用场景**

- 客户端不知道它所需要的对象的类。在工厂方法模式中，客户端不需要知道具体产品类的类名，只需要知道所对应的工厂即可，具体的产品对象由具体工厂类创建，可将具体工厂类的类名存储在配置文件或数据库中。
- 抽象工厂类通过其子类来指定创建哪个对象。在工厂方法模式中，对于抽象工厂类只需要提供一个创建产品的接口，而由其子类来确定具体要创建的对象，利用面向对象的多态性和里氏代换原则，在程序运行时，子类对象将覆盖父类对象，从而使得系统更容易扩展    

**示例**

![image-20210103132908151](img/C++%EF%BC%9A%E8%AE%BE%E8%AE%A1%E6%A8%A1%E5%BC%8F.img/image-20210103132908151.png)

```cpp
#include <iostream>
using namespace std;

class Car//车类
{
public:
	virtual void Show(void) = 0;
	virtual ~Car() {}
};

class BydCar : public Car //比亚迪汽车
{
public:
	BydCar() { cout << "Byd::Byd()" << endl; }
	~BydCar() { }
	virtual void Show(void)
	{
		cout << "BYD auto" << endl;
	}
};

class GeelyCar : public Car // 吉利汽车
{
public:
	GeelyCar() {cout << "Geely::Geely()" << endl; }
	~GeelyCar() {}
	virtual void Show(void)
	{
		cout << "GEELY auto" << endl;
	}
};

class Factory//车厂
{
public:
	virtual Car* createCar(void) = 0;
};

class BydFactory : public Factory
{
public:
	virtual Car* createCar(void) { return (new BydCar());}
};

class GeelyFactory : public Factory
{
public:
	virtual Car* createCar(void) { return (new GeelyCar()); }
};

int main()
{
	Factory* factory = new BydFactory();
	Car* bydCar = factory->createCar();
	factory = new GeelyFactory();
	Car* GeelyCar = factory->createCar();

	delete factory;
	delete bydCar;
	delete GeelyCar;
	return 0;
}
```

## 单例模式

**动机**

对于一个软件系统的某些类而言，我们无须创建多个实例。如 Windows 系统中的任务管理器，垃圾回收器。在实际开发中，我们也经常遇到类似的情况，为了节约系统资源，有时需要==确保系统中某个类只有唯一一个实例==，当这个唯一实例创建成功之后，我们无法再创建一个同类型的其他对象，所有的操作都只能基于这个唯一实例。为了确保对象的唯一性，我们可以通过单例模式来实现，这就是单例模式的动机所在  

**定义**

单例模式（Singleton Pattern）是最简单的设计模式之一。这种类型的设计模式属于创建型模式，它提供了一种创建对象的最佳方式。

这种模式涉及到一个单一的类，该类负责创建自己的对象，同时确保只有单个对象被创建。这个类提供了一种访问其唯一的对象的方式，可以直接访问，不需要实例化该类的对象。

单例模式的三个特点

- 单例类只能有一个实例。
- 单例类必须自己创建自己的唯一实例。
- 单例类必须给所有其他对象提供这一实例。

  <img src="img/C++%EF%BC%9A%E8%AE%BE%E8%AE%A1%E6%A8%A1%E5%BC%8F.img/image-20210103143758331.png" alt="image-20210103143758331" style="zoom:67%;" />

> 类里面不可以直接定义本类对象，这样实例化对象时会引起无限递归，但是可以定义成static成员，因为静态成员不属于某个具体的对象，而是隶属于整个类，构建对象的时候，对象里并不包含static成员，static成员存储在.data区
>
> ```cpp
> class Object
> {
>    	int value;
>    	static Object x;
> public:
>    	Object() {}
> };
> int main()
> {
>    	Object obj;
> }
> ```

**饿汉单例模式**

优点：程序加载时就进行实例化，之后的操作效率会很高。
缺点： 由于程序加载时就进行实例化，如果后续不对此类进行任何操作，就会导致内存的浪费  

```cpp
class Singleton
{
public:
	static Singleton* GetInstance()
	{
		return &instance;
	}
	~Singleton() {}
private:
	Singleton() {}
	Singleton(const Singleton& src) = delete;
	Singleton& operator=(const Singleton&) = delete;
private:
	static Singleton instance;
};
Singleton Singleton::instance; //饿汉式单例模式 一定是线程安全的
```

上边代码是线程安全的 ，因为进入主函数之前，single::instance是全局变量，已经创建了，不会在进入主函数之后若干个线程同时创建，所以实例化对象的时候并不会引起竞争关系，这个方案就是饿汉方式

```cpp
class Singleton
{
public:
	static Singleton &GetInstance(int x)
	{
		static Singleton instance(x); // 线程不一定安全
		return instance;
	}
	~Singleton() {}
private:
	Singleton(int x) {}
	Singleton(const Singleton& src) = delete;
	Singleton& operator=(const Singleton&) = delete;
};
```

上边代码线程不安全，静态成员需要根据传入的参数x来确定是否修改一些值，当多个线程同时执行时就有可能产生不确定性

**懒汉单例模式**

优点： 在第一次调用的时候才进行实例化。
缺点： 当多个线程同时进入到 if(instance == nullptr) {...} 时，会创建多个对象。  

```cpp
class Singleton
{
private:
	static Singleton* instance;
	Singleton() {}
	Singleton(const Singleton&) = delete;
	Singleton& operator=(const Singleton&) = delete;
public:
	~Singleton() {}
	static Singleton* GetInstance()
	{
		if (instance == nullptr)
		{
			instance = new Singleton();
		}
		return instance;
	}
};
Singleton* Singleton::instance = nullptr;
```

加锁后

```cpp
std::mutex mtx;
class Singleton
{
private:
	static Singleton* instance;
	Singleton() {}
	Singleton(const Singleton&) = delete;
	Singleton& operator=(const Singleton&) = delete;
public:
	~Singleton() {}
	static Singleton* GetInstance()
	{
		if (instance == nullptr)
		{
			std::lock_guard<std::mutex> lock(mtx);
			instance = new Singleton();
		}
		return instance;
	}
};
Singleton* Singleton::instance = nullptr;
```

**模板加继承** 

```cpp
template<class T>
class Singleton
{
public:
	Singleton(const Singleton&) = delete;
	Singleton& operator=(const Singleton&) = delete ;
protected:
	Singleton() {}
	virtual ~Singleton() {}
public:
	static T& instance()
	{
		static T theInstance;
		return theInstance;
	}
};

class MyClass : public Singleton<MyClass>
{
	int x; 
protected:
	friend class Singleton<MyClass>;
	MyClass() { x = 0; }
public:
	void setValue(int n) { x = n; }
	int getValue() const { return x; }
};

int main()
{
	MyClass& m = MyClass::instance();
	cout << m.getValue() << endl;
	m.setValue(1);
	cout << m.getValue() << endl;
	return 0;
} 
```

 **饿汉式单例类与懒汉式单例类比较**  

饿汉式单例类在类被加载时就将自己实例化，它的优点在于无须考虑多线程访问问题，可以确保实例的唯一性从调用速度和反应时间角度来讲，由于单例对象一开始就得以创建，因此要优于懒汉式单例。但是无论系统在运行时是否需要使用该单例对象，由于在类加载时该对象就需要创建，因此从资源利用效率角度来讲，饿汉式单例不及懒汉式单例，而且在系统加载时由于需要创建饿汉式单例对象，加载时间可能会比较长  

懒汉式单例类在第一次使用时创建，无须一直占用系统资源，实现了延迟加载，但是必须处理好多个线程同时访问的问题，特别是当单例类作为资源控制器，在实例化时必然涉及资源初始化，而资源初始化很有可能耗费大量时间，这意味着出现多线程同时首次引用此类的机率变得较大，需要通过双重检查锁定等机制进行控制，这将导致系统性能受到一定影响  

**单例模式优点**

- 在内存里只有一个实例，减少了内存的开销，尤其是频繁的创建和销毁实例（比如管理学院首页页面缓存）。
- 避免对资源的多重占用（比如写文件操作）。

**缺点**

- 由于单例模式中没有抽象层，因此单例类的扩展有很大的困难  
- 单例类的职责过重，在一定程度上违背了“单一职责原则”。因为单例类既充当了工厂角色，提供了工厂方法，同时又充当了产品角色，包含一些业务方法，将产品的创建和产品的本身的功能融合到一起  
- 现在很多面向对象语言(如 Java、 C#)的运行环境都提供了自动垃圾回收的技术，因此，如果实例化的共享对象长时间不被利用，系统会认为它是垃圾，会自动销毁并回收资源，下次利用时又将重新实例化，这将导致共享的单例对象状态的丢失  

**适用场景**

- 系统只需要一个实例对象，如系统要求提供一个唯一的序列号生成器或资源管理器，或者需要考虑资源
  消耗太大而只允许创建一个对象  
- 客户调用类的单个实例只允许使用一个公共访问点，除了该公共访问点，不能通过其他途径访问该实例 
- 典型应用场景 ， 日志文件，网站的计数器， 应用配置。控制资源的情况下，方便资源之间的互相通信，如线程池， 数据库连接池等   

# 结构型模式

这些设计模式关注类和对象的组合。

## 代理模式

**模式动机**

通过引入一个新的对象来实现对真实对象的操作或者将新的对象作为真实对象的一个替身，这种实现机制既为代理模式，通过引入代理对象来间接访问一个对象，这就是代理模式的模式动机  

在某些情况下，一个客户不想或者不能直接引用一个对象，此时可以通过一个称之为“代理”的第三者来实现间接引用， 代理对象在客户端对象和目标对象之间起到中介的作用，它去掉客户不能看到的内容和服务或者增添客户需要的额外的新服务。  

**定义**

为其他对象提供一种代理以控制对这个对象的访问。

在直接访问对象时带来的问题，比如说：要访问的对象在远程的机器上。在面向对象系统中，有些对象由于某些原因（比如对象创建开销很大，或者某些操作需要安全控制，或者需要进程外的访问），直接访问会给使用者或者系统结构带来很多麻烦，我们可以在访问此对象时加上一个对此对象的访问层。

代理模式的结构比较简单，其核心是代理类，为了让客户端能够一致性地对待真实对象和代理对象，在代理模式中引入了抽象层，代理模式结构如图所示：  

<img src="img/C++%EF%BC%9A%E8%AE%BE%E8%AE%A1%E6%A8%A1%E5%BC%8F.img/image-20210104160330305.png" alt="image-20210104160330305" style="zoom:67%;" />

```cpp
#include<iostream>
using namespace std;

struct Subject
{
	virtual void request() = 0;
};
class RealSubject : public Subject
{
public:
	virtual void request()
	{
		cout << "Real Subject::request" << endl;
	}
};
class Proxy : public Subject
{
private:
	RealSubject* realSubject;
public:
	Proxy() :realSubject(new RealSubject()) {}
	void preRequest() { cout << "pre" << endl; }
	void postRequest() { cout << "post" << endl; }
	void request()
	{
		preRequest();
		realSubject->request();
		postRequest();
	}
};
int main()
{
	Subject* sub = new Proxy();
	sub->request();
	return 0;
}
```

**包含的角色**

- Subject（抽象主题角色）：它声明了真实主题和代理主题的共同接口，这样一来在任何使用真实主题的地方都可以使用代理主题，客户端通常需要针对抽象主题角色进行编程  
- Proxy（代理主题角色）：它包含了对真实主题的引用，从而可以在任何时候操作真实主题对象；在代理主题角色中提供一个与真实主题角色相同的接口，以便在任何时候都可以替代真实主题；代理主题角色还可以控制对真实主题的使用，负责在需要的时候创建和删除真实主题对象，并对真实主题对象的使用加以约束。通常在
  代理主题角色中，客户端在调用所引用的真实主题操作之前或之后还需要执行其他操作，而不仅仅是单纯调用真实主题对象中的操作
- RealSubject（真实主题角色）：它定义了代理角色所代表的真实对象，在真实主题角色中实现了真实的业务操作，客户端可以通过代理主题角色间接调用真实主题角色中定义的操作。    

**常用的代理模式**

- 远程代理(Remote Proxy)：为一个位于不同的地址空间的对象提供一个本地的代理对象,这个不同的地址空间可以是在同一台主机中，也可是在另一台主机中，远程代理又称为大使(Ambassador)。  
- 虚拟代理(Virtual Proxy)：如果需要创建一个资源消耗较大的对象,先创建一个消耗相对较小的对象来表示,真实对象只在需要时才会被真正创建。  
- 保护代理(Protect Proxy)：控制对一个对象的访问,可以给不同的用户提供不同级别的使用设计模式之代理模式权限  
- 缓冲代理(Cache Proxy)：为某一个目标操作的结果提供临时的存储空间，以便多个客户端可以共享这些结果
- 智能引用代理(Smart Reference Proxy)：当一个对象被引用时,提供一些额外的操作，例如将对象被调用的次数记录下来等。    

在这些常用的代理模式中，有些代理类的设计非常复杂，例如远程代理类，它封装了底层网络通信和对远程对象的调用，其实现较为复杂。  

**模式优点**

- 能够协调调用者和被调用者，在一定程度上降低了系统的耦合度  
- 客户端可以针对抽象主题角色进行编程，增加和更换代理类无须修改源代码，符合开闭原则，系统具有较好的灵活性和可扩展性。 此外，不同类型的代理模式也具有独特的优点，例如：  
  - 远程代理为位于两个不同地址空间对象的访问提供了一种实现机制，可以将一些消耗资源较多的对象和操作移至性能更好的计算机上，提高系统的整体运行效率  
  - 虚拟代理通过一个消耗资源较少的对象来代表一个消耗资源较多的对象，可以在一定程度上节省系统的运行开销  
  - 缓冲代理为某一个操作的结果提供临时的缓存存储空间，以便在后续使用中能够共享这些结果，优化系统性能，缩短执行时间  
  - 保护代理可以控制对一个对象的访问权限，为不同用户提供不同级别的使用权限  

**模式缺点**

- 由于在客户端和真实主题之间增加了代理对象，因此有些类型的代理模式可能会造成请求的处理速度变慢，例如保护代理  
- 实现代理模式需要额外的工作，而且有些代理模式的实现过程较为复杂，例如远程代理。  

**适用场景**

- 远程代理：当客户端对象需要访问远程主机中的对象时  
- 虚拟代理：当需要用一个消耗资源较少的对象来代表一个消耗资源较多的对象，从而降低系统开销、 缩短运行时间时可以使用虚拟代理，例如一个对象需要很长时间才能完成加载时  
- 缓冲代理：当需要为某一个被频繁访问的操作结果提供一个临时存储空间，以供多个客户端共享访问这些结果时可以使用缓冲代理。通过使用缓冲代理，系统无须在客户端每一次访问时都重新执行操作，只需直接从临时缓冲区获取操作结果即可
- 保护代理：当需要控制对一个对象的访问，为不同用户提供不同级别的访问权限时可以使用保护代理。  
- 智能引用代理：当需要为一个对象的访问（引用）提供一些额外的操作时可以使用智能引用代理。    

**虚拟代理实例**

![image-20210104161741083](img/C++%EF%BC%9A%E8%AE%BE%E8%AE%A1%E6%A8%A1%E5%BC%8F.img/image-20210104161741083.png)

![image-20210104161804077](img/C++%EF%BC%9A%E8%AE%BE%E8%AE%A1%E6%A8%A1%E5%BC%8F.img/image-20210104161804077.png)

**远程代理实例**

![image-20210104161835440](img/C++%EF%BC%9A%E8%AE%BE%E8%AE%A1%E6%A8%A1%E5%BC%8F.img/image-20210104161835440.png)

![image-20210104161845370](img/C++%EF%BC%9A%E8%AE%BE%E8%AE%A1%E6%A8%A1%E5%BC%8F.img/image-20210104161845370.png)

![image-20210104161900131](img/C++%EF%BC%9A%E8%AE%BE%E8%AE%A1%E6%A8%A1%E5%BC%8F.img/image-20210104161900131.png)

![image-20210104161910313](img/C++%EF%BC%9A%E8%AE%BE%E8%AE%A1%E6%A8%A1%E5%BC%8F.img/image-20210104161910313.png)

# 行为型模式

这些设计模式特别关注对象之间的通信。主要用于描述类或对象如何交互和怎样分配职责  

关注系统中对象之间的相互交互，研究运行时对象之间的相互通信和协作，明确对象职责  

## 观察者模式

**模式动机**

建立一套低耦合的消息触发机制。  

建立一种对象与对象之间的依赖关系，一个对象发生改变时将自动通知其他对象，其他对象将相应做出反应。在
此，发生改变的对象称为观察目标，而被通知的对象称为观察者，一个观察目标可以对应多个观察者，而且这些观察者之间没有相互联系，可以根据需要增加和删除观察者，使得系统更易于扩展，这就是观察者模式的模式动机。  

**定义**

观察者模式(Observer Pattern)：定义对象之间的一种一对多依赖关系，使得每当一个对象状态发生改变时，其相关依赖对象皆得到通知并被自动更新。观察者模式的别名包括发布-订阅（Publish/Subscribe）模式、模型-视图
（Model/View）模式、源-监听器（Source/Listener）模式或从属者（Dependents）模式。观察者模式是一种对象行为型模式。

观察者模式结构中通常包括观察目标和观察者两个继承层次结构，其结构如图所示：

猫叫老鼠跑，狗也跟着叫，使用观察者模式描述该过程

<img src="img/C++%EF%BC%9A%E8%AE%BE%E8%AE%A1%E6%A8%A1%E5%BC%8F.img/image-20210104154104218.png" alt="image-20210104154104218" style="zoom:67%;" />    

  ![image-20210104154727359](img/C++%EF%BC%9A%E8%AE%BE%E8%AE%A1%E6%A8%A1%E5%BC%8F.img/image-20210104154727359.png)

**包含的角色**

- Subject（目标）：目标又称为主题，它是指被观察的对象。在目标中定义了一个观察者集合，一个观察目标可以接受任意数量的观察者来观察，它提供一系列方法来增加和删除观察者对象，同时它定义了通知方法
  notify()。目标类可以是接口，也可以是抽象类或具体类  

- ConcreteSubject（具体目标）：具体目标是目标类的子类，通常它包含有经常发生改变的数据，当它的状态发生改变时，向它的各个观察者发出通知；同时它还实现了在目标类中定义的抽象业务逻辑方法（如果有的
  话）。如果无须扩展目标类，则具体目标类可以省略  
- Observer（观察者）：观察者将对观察目标的改变做出反应，观察者一般定义为接口，该接口声明了更新数据的方法 update()，因此又称为抽象观察者。  
- ConcreteObserver（具体观察者）：在具体观察者中维护一个指向具体目标对象的引用，它存储具体观察者的有关状态，这些状态需要和具体目标的状态保持一致；它实现了在抽象观察者 Observer 中定义的update()方法。通常在实现时，可以调用具体目标类的 attach()方法将自己添加到目标类的集合中或通过detach()方法将自己从目标类的集合中删除  

**优点**

- 观察者模式可以实现表示层和数据逻辑层的分离，定义了稳定的消息更新传递机制，并抽象了更新接口，使得可以有各种各样不同的表示层充当具体观察者角色。  
- 观察者模式在观察目标和观察者之间建立一个抽象的耦合。观察目标只需要维持一个抽象观察者的集合，无须了解其具体观察者。由于观察目标和观察者没有紧密地耦合在一起，因此它们可以属于不同的抽象化层次。
- 观察者模式支持广播通信，观察目标会向所有已注册的观察者对象发送通知，简化了一对多系统设计的难度
- 观察者模式满足“开闭原则” 的要求，增加新的具体观察者无须修改原有系统代码，在具体观察者与观察目标之间不存在关联关系的情况下，增加新的观察目标也很方便      

**缺点**

- 如果一个观察目标对象有很多直接和间接观察者，将所有的观察者都通知到会花费很多时间。  
- 如果在观察者和观察目标之间存在循环依赖，观察目标会触发它们之间进行循环调用，可能导致系统崩溃 
- 观察者模式没有相应的机制让观察者知道所观察的目标对象是怎么发生变化的，而仅仅只是知道观察目标发生了变化   

**适用场景**

- 一个抽象模型有两个方面，其中一个方面依赖于另一个方面，将这两个方面封装在独立的对象中使它们可以各自独立地改变和复用  
- 一个对象的改变将导致一个或多个其他对象也发生改变，而并不知道具体有多少对象将发生改变，也不知道这些对象是谁。  
- 需要在系统中创建一个触发链， A 对象的行为将影响 B 对象， B 对象的行为将影响 C 对象……，可以使用观察者模式创建一种链式触发机制。  

**MVC架构**

在当前流行的 MVC(Model-View-Controller)架构中也应用了观察者模式， MVC 是一种架构模式， 它包含三个角
色： 模型(Model)， 视图(View)和控制器(Controller)。其中模型可对应于观察者模式中的观察目标，而视图对应于观察者，控制器可充当两者之间的中介者。当模型层的数据发生改变时，视图层将自动改变其显示内容。如图所
示： MVC 结构示意图  

<img src="img/C++%EF%BC%9A%E8%AE%BE%E8%AE%A1%E6%A8%A1%E5%BC%8F.img/image-20210104155852677.png" alt="image-20210104155852677" style="zoom:67%;" />