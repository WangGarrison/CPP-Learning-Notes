# 作业：观察者模式

Sunny软件公司欲开发一款实时在线股票软件，该软件需要提供如下功能：

当股票购买者所购买的某只股票价格变化幅度达到5%时，系统将自动发送通知（包括新价格）给购买该股票的所有股民。试使用观察者模式设计并实现改系统。

```cpp
#include <iostream>
#include <list>
using namespace std;

/*
设计模式：观察者模式
Sunny软件公司欲开发一款实时在线股票软件，该软件需要提供如下功能：当股票购买者所购买的某只股票价格变化幅度达到5%时，系统将自动发送通知（包括新价格）给购买该股票的所有股民。试使用观察者模式设计并实现改系统。
*/
class Observer
{
public:
	virtual void response() = 0;
	virtual ~Observer() { cout << "析构Observer" << endl; }
};

class PurchaseA :public Observer  //观察者A
{
public:
	virtual void response()
	{
		cout << "A收到股票价格的变化" << endl;
	}
	~PurchaseA()
	{
		cout << "析构A" << endl;
	}
};

class PurchaseB :public Observer  //观察者B
{
public:
	virtual void response()
	{
		cout << "B收到股票价格的变化" << endl;
	}
	~PurchaseB()
	{
		cout << "析构B" << endl;
	}
};

class Subject  //观察的目标
{
private:
	list<shared_ptr<Observer>> obs_list;
	double price;
public:
	~Subject()
	{
		cout << "析构Subject" << endl;
	}
	void attach(shared_ptr<Observer> obs)//注册方法
	{
		obs_list.push_back(obs);
	}
	void dettach(shared_ptr<Observer> obs)//注销方法
	{
		obs_list.remove(obs);
	}
	void price_changed(int newprice)//价格变动，自动通知所有的观察者
	{
		if (price != 0 && ((newprice - price) / price >= 0.05))
		{
			cout << "价格变化幅度达到%5，" << "新价格：" << newprice << endl;
			for (auto it : obs_list)
			{
				it->response();
			}
		}
		else
		{
			cout << "价格变化幅度在%5之内，" << "新价格" << newprice << endl;
		}
		price = newprice;
	}
};

int main()
{
	shared_ptr<Observer> a(new PurchaseA);
	shared_ptr<Observer> b(new PurchaseB);

	shared_ptr<Subject> sub(new Subject());
	sub->attach(a);
	sub->attach(b);
	sub->price_changed(1);

	sub->price_changed(5);
}
```

# 观察者模式如何释放资源：使用智能指针，参考工厂模式

//底下代码虽然改用了智能指针，但是没能自动执行析构函数

```cpp
#include "pch.h"
#include <iostream>
#include <list>
#include <string>
using namespace std;

class Subject;  

class Observer
{
protected:
	string name;
	weak_ptr<Subject> sub;
public:
	Observer(string na, weak_ptr<Subject> s)
	{
		this->name = na;
		this->sub = s;
	}
	virtual ~Observer() {}
	virtual void update() = 0;
};

class StockObserver : public Observer
{
public:
	StockObserver(string name, weak_ptr<Subject> sub) :Observer(name, sub) {}
	void update();
	~StockObserver()
	{
		cout << "StockObserver destruct!" << endl;
	}
};

class NBAObserver : public Observer
{
public:
	NBAObserver(string name, weak_ptr<Subject> sub) :Observer(name, sub) {}
	void update();
	~NBAObserver()
	{
		cout << "NBAObserver destruct" << endl;
	}
};

class Subject
{
protected:
	list<shared_ptr<Observer> > obs_list;
public:
	string event;
	virtual void attach(shared_ptr<Observer>) = 0;
	virtual void detach(shared_ptr<Observer>) = 0;
	virtual void notify() = 0;
	virtual ~Subject() {}
};

class Secretary :public Subject
{
public:
	virtual void attach(shared_ptr<Observer> obs)
	{
		obs_list.push_back(obs);
	}
	virtual void detach(shared_ptr<Observer> obs)
	{
		obs_list.remove(obs);
	}
	void notify()
	{
		for (auto &x : obs_list)
		{
			x->update();
		}
	}
	~Secretary()
	{
		cout << "Secretary destruct" << endl;
	}
};

void StockObserver::update()
{
	cout << name << " " << " 收到消息" << sub.lock()->event << endl;
	if (sub.lock()->event == "Boss来了")
	{
		cout << "我马上关闭股票，开始工作！" << endl;
	}
}

void NBAObserver::update()
{
	cout << name << " " << " 收到消息" << sub.lock()->event << endl;
	if (sub.lock()->event == "Boss来了")
	{
		cout << "我马上关闭NBA，开始工作！" << endl;
	}
}

int main()
{
	shared_ptr<Subject> dwq(new Secretary());
	shared_ptr<Observer> xs(new NBAObserver("小帅", dwq));
	shared_ptr<Observer> lm(new StockObserver("李明", dwq));
	shared_ptr<Observer> wf(new NBAObserver("王芳", dwq));
	dwq->attach(xs);
	dwq->attach(lm);
	dwq->attach(wf);

	dwq->event = "去吃饭了！";
	dwq->notify();

	dwq->event = "Boss来了";
	dwq->notify();
	return 0;
}
```

